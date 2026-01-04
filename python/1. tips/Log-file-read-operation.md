

There are a few practical ways to **print or log every file read** happening in a Python app, depending on how deep you want to go:

- ✅ **Python-level** (monkey-patch `open()` and wrap file objects): logs all `.read()`, `.readline()`, `.readlines()`, iteration, etc., across your code and most libraries.
- ✅ **Python audit hooks** (`sys.addaudithook`): logs **file opens** (not reads), very low overhead.
- ✅ **OS-level tracing** (`strace`, `dtruss`, Process Monitor): logs **every kernel read** regardless of language (even if libraries use `os.read` or memory-mapped files).

Below are ready-to-use options with code and commands.

---

## Option 1 — Python-level: log every `.read*()` call (drop-in)

This monkey-patches `builtins.open` (and `pathlib.Path.open`) to wrap returned file objects with a logger that intercepts reads. It works across your app and most third-party libraries (e.g., `pandas.read_csv()` internally calls `open`).

> ⚠️ Notes:
> 
> - This adds overhead (every read is instrumented).
> - Won’t catch memory-mapped reads (`mmap`) or code that calls `os.read` on raw FDs directly (rare for pure Python apps).
> - Safe by default (logs sizes, not contents). You can enable content previews if you want.

```Python
# file_read_logging.py
import builtins
import io
import os
import pathlib
import threading
import logging
import traceback
from functools import wraps

__all__ = ["enable_file_read_logging", "disable_file_read_logging"]

# Globals to restore later
_ORIGINAL_OPEN = builtins.open
_ORIGINAL_PATH_OPEN = pathlib.Path.open

# A guard to prevent recursion when our logger itself writes files, etc.
_thread_guard = threading.local()
_thread_guard.active = False

def _safe_log(logger, level, msg):
    """Avoid re-entrancy if logger writes to files (which could cause infinite loops)."""
    if getattr(_thread_guard, "active", False):
        return
    try:
        _thread_guard.active = True
        logger.log(level, msg)
    finally:
        _thread_guard.active = False

class _LoggedFile:
    """
    A thin wrapper around a file object that logs read operations.
    Works for both text and binary files.
    """
    def __init__(self, f, logger, include_stack=False, preview_chars=0):
        self._f = f
        self._logger = logger
        self._include_stack = include_stack
        self._preview_chars = preview_chars  # 0 = disabled
        # expose common attributes for compatibility
        self.name = getattr(f, "name", None)
        self.mode = getattr(f, "mode", None)

    def _fmt_stack(self):
        if not self._include_stack:
            return ""
        # Exclude the last few frames coming from this module
        stack = traceback.format_stack(limit=12)
        # Drop last 2 frames (this method + caller in wrapper)
        return "\nCall stack:\n" + "".join(stack[:-2])

    def _preview(self, data):
        if self._preview_chars &lt;= 0:
            return ""
        try:
            if isinstance(data, (bytes, bytearray)):
                sample = data[: self._preview_chars]
                # Show as hex for bytes to avoid binary noise
                return f" | preview(bytes hex)={sample.hex()}"
            else:
                s = str(data)
                return f" | preview='{s[: self._preview_chars]}'"
        except Exception:
            return ""

    # --- Read methods we intercept ---

    def read(self, *args, **kwargs):
        data = self._f.read(*args, **kwargs)
        n = len(data) if hasattr(data, "__len__") else data if isinstance(data, int) else None
        _safe_log(self._logger, logging.INFO,
                  f"[READ] file={self.name!r} mode={self.mode!r} size={n}{self._preview(data)}{self._fmt_stack()}")
        return data

    def readline(self, *args, **kwargs):
        line = self._f.readline(*args, **kwargs)
        n = len(line) if hasattr(line, "__len__") else None
        _safe_log(self._logger, logging.INFO,
                  f"[READLINE] file={self.name!r} mode={self.mode!r} size={n}{self._preview(line)}{self._fmt_stack()}")
        return line

    def readlines(self, *args, **kwargs):
        lines = self._f.readlines(*args, **kwargs)
        total = sum(len(x) for x in lines) if lines else 0
        _safe_log(self._logger, logging.INFO,
                  f"[READLINES] file={self.name!r} mode={self.mode!r} total_bytes={total} count={len(lines)}{self._fmt_stack()}")
        return lines

    # Buffered binary APIs
    def read1(self, *args, **kwargs):
        data = self._f.read1(*args, **kwargs)
        n = len(data) if hasattr(data, "__len__") else None
        _safe_log(self._logger, logging.INFO,
                  f"[READ1] file={self.name!r} size={n}{self._preview(data)}{self._fmt_stack()}")
        return data

    def readinto(self, b):
        n = self._f.readinto(b)
        _safe_log(self._logger, logging.INFO,
                  f"[READINTO] file={self.name!r} size={n}{self._fmt_stack()}")
        return n

    def readinto1(self, b):
        # Some buffered readers support this
        n = self._f.readinto1(b)
        _safe_log(self._logger, logging.INFO,
                  f"[READINTO1] file={self.name!r} size={n}{self._fmt_stack()}")
        return n

    # Iteration over lines
    def __iter__(self):
        return self

    def __next__(self):
        line = next(self._f)
        n = len(line) if hasattr(line, "__len__") else None
        _safe_log(self._logger, logging.INFO,
                  f"[ITER] file={self.name!r} size={n}{self._preview(line)}{self._fmt_stack()}")
        return line

    # Delegate everything else to the underlying file
    def __getattr__(self, name):
        return getattr(self._f, name)

    def close(self):
        _safe_log(self._logger, logging.DEBUG, f"[CLOSE] file={self.name!r}")
        return self._f.close()

def enable_file_read_logging(
    logger: logging.Logger | None = None,
    include_stack: bool = False,
    preview_chars: int = 0,
):
    """
    Enable logging of all file .read*() operations by monkey-patching builtins.open and Path.open.

    Args:
        logger: a logging.Logger to emit records; defaults to a sensible stdout logger.
        include_stack: if True, include a capture of the Python stack for each read.
        preview_chars: if &gt;0, log a preview (first N chars/bytes) of read content; be careful with PII.
    """
    global _ORIGINAL_OPEN, _ORIGINAL_PATH_OPEN

    if logger is None:
        logger = logging.getLogger("file_reads")
        if not logger.handlers:
            h = logging.StreamHandler()  # stdout/stderr
            h.setFormatter(logging.Formatter("%(asctime)s %(levelname)s %(message)s"))
            logger.addHandler(h)
        logger.setLevel(logging.INFO)

    @wraps(_ORIGINAL_OPEN)
    def patched_open(*args, **kwargs):
        f = _ORIGINAL_OPEN(*args, **kwargs)
        # Only wrap things that look like file objects (TextIOBase/BufferedIOBase/FileIO)
        if isinstance(f, (io.TextIOBase, io.BufferedIOBase, io.RawIOBase)):
            _safe_log(logger, logging.DEBUG, f"[OPEN] file={getattr(f, 'name', None)!r} mode={getattr(f, 'mode', None)!r}")
            return _LoggedFile(f, logger, include_stack=include_stack, preview_chars=preview_chars)
        return f

    @wraps(_ORIGINAL_PATH_OPEN)
    def patched_path_open(path_obj, *args, **kwargs):
        f = _ORIGINAL_PATH_OPEN(path_obj, *args, **kwargs)
        if isinstance(f, (io.TextIOBase, io.BufferedIOBase, io.RawIOBase)):
            _safe_log(logger, logging.DEBUG, f"[OPEN] file={getattr(f, 'name', None)!r} mode={getattr(f, 'mode', None)!r}")
            return _LoggedFile(f, logger, include_stack=include_stack, preview_chars=preview_chars)
        return f

    builtins.open = patched_open
    pathlib.Path.open = patched_path_open

def disable_file_read_logging():
    """Restore original open functions."""
    global _ORIGINAL_OPEN, _ORIGINAL_PATH_OPEN
    builtins.open = _ORIGINAL_OPEN
    pathlib.Path.open = _ORIGINAL_PATH_OPEN
```

**Usage:**

```Python
from file_read_logging import enable_file_read_logging, disable_file_read_logging

# Turn it on (stack off, no content preview, safest)
enable_file_read_logging(include_stack=False, preview_chars=0)

# Your app code...
with open("example.txt", "r") as f:
    data = f.read()

# Turn it off when done (optional if process exits)
disable_file_read_logging()
```

**Tips**

- If you need to see _who_ is reading, set `include_stack=True`.
- If you need a small peek into content (debugging parsing issues), set `preview_chars=80` (be mindful of secrets/PII).
- If you log to a file, consider directing the logger to `StreamHandler` (stdout/stderr) to avoid recursive file operations.

---

## Option 2 — Python’s Audit Hook: log **opens** (not reads)

If you only need to know **what files are opened** (and by whom) with minimal performance impact, use `sys.addaudithook` (Python 3.8+).

```Python
# audit_open_logging.py
import sys
import logging

def enable_open_audit_logging():
    logger = logging.getLogger("file_open_audit")
    if not logger.handlers:
        h = logging.StreamHandler()
        h.setFormatter(logging.Formatter("%(asctime)s %(levelname)s %(message)s"))
        logger.addHandler(h)
    logger.setLevel(logging.INFO)

    def audit_hook(event, args):
        # 'open' is emitted for builtins.open and some os-level opens
        if event in ("open", "openat"):  # 'openat' may appear on some platforms
            try:
                path = args[0]  # (path, mode, flags)
            except Exception:
                path = "&lt;unknown&gt;"
            logger.info(f"[AUDIT OPEN] event={event} path={path!r} args={args}")

    sys.addaudithook(audit_hook)
```

This is very lightweight and **doesn’t require monkey-patching**, but it won’t capture per-call `.read()` events.

---

## Option 3 — OS-level tracing (captures _everything_, even outside Python)

If you need **ground truth** of every kernel-level file read (including `os.read`, `mmap` page-ins, C extensions, etc.), run the app under an OS tracer.

### Linux

```Shell
# Run and trace opens &amp; reads (current process)
strace -f -e trace=openat,read -s 256 -o io.log python your_app.py

# Attach to an already-running PID
strace -f -p &lt;PID&gt; -e trace=openat,read -s 256 -o io.log
```

### macOS

```Shell
# dtruss (requires sudo). Trace opens &amp; reads:
sudo dtruss -f -n Python 2&gt; io.log

# Or, high-level filesystem activity
sudo fs_usage -w -f filesys | grep -i python
```

### Windows

- Use **Sysinternals Process Monitor (ProcMon)**:
    1. Filter by `Process Name is python.exe` (or your exe).
    2. Show operations: `CreateFile`, `ReadFile`, etc.
    3. Save captured events to CSV for analysis.

**Pros:** Full fidelity across all libraries.\ **Cons:** Heavier output, platform-specific tools, not embedded in Python.

---

## Choosing the right approach

- **Want quick in-app visibility of who reads what?** → **Option 1** (monkey-patch).
- **Only need to know which files are opened?** → **Option 2** (audit hook).
- **Need absolute ground truth (including C extensions, `os.read`, mmap)?** → **Option 3** (OS tracing).

---

## Caveats & best practices

- **Performance:** Logging every `.read()` can be very chatty for large or tight loops. Consider sampling (log every N reads) or logging only totals.
- **Security/PII:** Avoid logging contents unless necessary. Use `preview_chars` sparingly and never in production with sensitive data.
- **Threads/async:** The wrapper is thread-safe for logging, but the underlying I/O characteristics don’t change.
- **Limitations (Option 1/2):**
    - Code using raw FDs with `os.read` may bypass the wrapper (Option 1).
    - Audit hooks (Option 2) capture opens but not per-read calls.
    - Memory-mapped I/O won’t trigger reads in Python layer.

---

### Quick question

Which environment are you running this in (Linux/macOS/Windows), and do you want Python-level logging or OS-level tracing? If you share a snippet of how files are read in your app (e.g., `open`, `pathlib`, `pandas`, `gzip`, etc.), I can tailor the hook to cover those paths precisely.