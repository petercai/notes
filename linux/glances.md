Here’s a **practical, near-complete cheat sheet of Glances TUI keyboard commands on Linux**.
(Glances is modular and version-dependent, but this covers **the standard/default bindings** you’ll encounter in current releases.)

---

# 🖥️ Glances TUI Keyboard Commands (Linux)

## 🚪 General / Global

| Key        | Action                               |
| ---------- | ------------------------------------ |
| `h`        | Show help (this list inside Glances) |
| `q`        | Quit Glances                         |
| `Esc`      | Quit current popup                   |
| `Ctrl + C` | Force quit                           |
| `r`        | Reset min/max stats                  |
| `d`        | Show/hide disk I/O stats             |
| `t`        | Show/hide network I/O                |
| `l`        | Show/hide log messages               |
| `w`        | Toggle wide mode                     |
| `Ctrl + L` | Redraw screen                        |

---

## 🧭 Navigation

| Key             | Action                                |
| --------------- | ------------------------------------- |
| `↑` / `↓`       | Move selection                        |
| `PgUp` / `PgDn` | Page up / down                        |
| `Home` / `End`  | Jump to top / bottom                  |
| `Tab`           | Switch process view (CPU ↔ MEM ↔ I/O) |

---

## 🔍 Search & Filter

| Key | Action           |
| --- | ---------------- |
| `/` | Search process   |
| `\` | Clear search     |
| `f` | Filter processes |
| `F` | Clear filter     |

---

## 📊 Process Sorting

| Key | Sort By      |
| --- | ------------ |
| `a` | Auto sort    |
| `c` | CPU          |
| `m` | Memory       |
| `i` | I/O          |
| `p` | Process name |
| `t` | Time         |
| `P` | PID          |
| `s` | Swap         |
| `u` | User         |

---

## ⚙️ Process Control

| Key | Action                      |
| --- | --------------------------- |
| `k` | Kill selected process       |
| `K` | Kill all filtered processes |
| `n` | Nice − (decrease priority)  |
| `N` | Nice + (increase priority)  |

---

## 🌲 Process View Modes

| Key | Action                      |
| --- | --------------------------- |
| `T` | Tree view                   |
| `C` | Cumulative CPU              |
| `I` | Show/hide idle processes    |
| `z` | Highlight running processes |
| `Z` | Highlight zombie processes  |

---

## 🧠 CPU / Memory / System Modules

| Key | Toggle        |
| --- | ------------- |
| `1` | Per-CPU stats |
| `2` | CPU frequency |
| `3` | Load average  |
| `4` | Memory        |
| `5` | Swap          |
| `6` | Sensors       |
| `7` | GPU           |
| `8` | Network       |
| `9` | Disk I/O      |

---

## 📦 Containers / Virtualization (if enabled)

| Key | Action            |
| --- | ----------------- |
| `o` | Docker containers |
| `v` | Virtual machines  |

---

## 🌐 Network

| Key | Action             |
| --- | ------------------ |
| `b` | Bytes/s ↔ Bits/s   |
| `e` | Network interfaces |
| `E` | Network errors     |

---

## 📁 Filesystem / Disk

| Key | Action      |
| --- | ----------- |
| `D` | Disk usage  |
| `R` | RAID status |

---

## 📈 Views & Layout

| Key | Action                              |
| --- | ----------------------------------- |
| `V` | Toggle vertical / horizontal layout |
| `W` | Toggle wide process view            |
| `x` | Enable/disable extended stats       |

---

## 🧪 Advanced / Misc

| Key | Action                   |
| --- | ------------------------ |
| `s` | Open Glances server mode |
| `S` | Client/server stats      |
| `A` | Show alerts              |
| `y` | Enable/disable alerting  |
| `g` | Enable graph view        |
| `G` | Fullscreen graphs        |

---

## ⚠️ Alerts & Colors

| Key | Action             |
| --- | ------------------ |
| `!` | Show alert history |
| `H` | Toggle colors      |
| `B` | Toggle bold mode   |

---

## 🧾 Configuration & Notes

* Config file:

  ```bash
  ~/.config/glances/glances.conf
  ```
* Modules shown & hotkeys **depend on enabled plugins**
* Remote mode & Docker/K8s add extra keys dynamically

---

## 🧠 Power-User Tips

* Use **filter (`f`) + kill (`K`)** for batch cleanup
* Combine **tree (`T`) + cumulative CPU (`C`)** to find fork bombs
* Glances excels at **system-wide telemetry**, not micro-process control
* For per-process deep dives → use `htop`; for ops dashboards → Glances

---

If you want, I can:

* 📄 Generate a **one-page printable Glances cheat sheet**
* ⚔️ Compare **Glances vs htop vs btop vs atop**
* 🧩 List **Docker / K8s-specific Glances commands**
* 🔧 Explain **glances.conf tuning for servers**

Just say the word.
