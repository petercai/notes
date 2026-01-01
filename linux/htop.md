Below is a **comprehensive, practical list of `htop` TUI keyboard commands** commonly available on Linux.
(Some keys may vary slightly by distro/version or your custom `~/.config/htop/htoprc`, but this covers **almost all standard bindings**.)

---

## 🔑 Global / Navigation

| Key             | Action                   |
| --------------- | ------------------------ |
| `↑` `↓`         | Move selection up / down |
| `←` `→`         | Scroll horizontally      |
| `PgUp` / `PgDn` | Page up / down           |
| `Home` / `End`  | Jump to top / bottom     |
| `Tab`           | Switch between panels    |
| `Shift + Tab`   | Reverse panel switch     |
| `Esc`           | Cancel / go back         |
| `h`             | Show help                |
| `q`             | Quit `htop`              |
| `F10`           | Quit                     |

---

## 📊 Process List / Sorting

| Key  | Action                  |
| ---- | ----------------------- |
| `F6` | Choose sort column      |
| `>`  | Sort by next column     |
| `<`  | Sort by previous column |
| `I`  | Invert sort order       |
| `P`  | Sort by CPU usage       |
| `M`  | Sort by memory usage    |
| `T`  | Sort by time            |
| `N`  | Sort by PID             |

---

## 🔍 Searching & Filtering

| Key  | Action              |
| ---- | ------------------- |
| `F3` | Search process      |
| `F4` | Filter processes    |
| `/`  | Incremental search  |
| `\`  | Clear search filter |

---

## ⚙️ Process Control

| Key        | Action                     |
| ---------- | -------------------------- |
| `F9`       | Kill process (signal menu) |
| `k`        | Kill process               |
| `Space`    | Tag / untag process        |
| `U`        | Untag all                  |
| `Ctrl + A` | Tag all                    |
| `Ctrl + K` | Kill tagged processes      |
| `Ctrl + P` | Pin process                |

---

## 🧵 Tree / Threads

| Key         | Action                  |
| ----------- | ----------------------- |
| `F5`        | Tree view               |
| `t`         | Toggle tree view        |
| `H`         | Hide user threads       |
| `K`         | Hide kernel threads     |
| `Shift + H` | Toggle threads globally |

---

## 🧠 CPU / Memory Views

| Key         | Action                    |
| ----------- | ------------------------- |
| `1`         | Toggle per-CPU view       |
| `2`         | Toggle per-NUMA node view |
| `Shift + M` | Toggle memory meters      |
| `Shift + P` | Toggle CPU meters         |
| `Shift + T` | Toggle task time          |

---

## 👤 User / Process Scope

| Key         | Action                   |
| ----------- | ------------------------ |
| `u`         | Filter by user           |
| `Shift + U` | Show all users           |
| `F7`        | Decrease priority (nice) |
| `F8`        | Increase priority        |

---

## 🧩 Columns & Layout

| Key                    | Action                |
| ---------------------- | --------------------- |
| `F2`                   | Setup / configuration |
| `F2 → Columns`         | Add/remove columns    |
| `F2 → Meters`          | Configure top meters  |
| `F2 → Colors`          | Change color scheme   |
| `F2 → Display options` | Toggle UI options     |
| `F2 → Key bindings`    | Custom key mapping    |

---

## 📈 Information Panels

| Key   | Action |
| ----- | ------ |
| `F1`  | Help   |
| `F2`  | Setup  |
| `F3`  | Search |
| `F4`  | Filter |
| `F5`  | Tree   |
| `F6`  | Sort   |
| `F7`  | Nice − |
| `F8`  | Nice + |
| `F9`  | Kill   |
| `F10` | Quit   |

---

## 🧪 Advanced / Less-Known

| Key        | Action                |
| ---------- | --------------------- |
| `Ctrl + L` | Redraw screen         |
| `Ctrl + R` | Refresh               |
| `Ctrl + C` | Cancel / exit dialogs |
| `Ctrl + S` | Pause updates         |
| `Ctrl + Q` | Resume updates        |
| `=`        | Expand tree node      |
| `-`        | Collapse tree node    |
| `+`        | Expand subtree        |

---

## 📁 Config File Location

```bash
~/.config/htop/htoprc
```

You can inspect or version-control this file to **fully understand or customize** all bindings.

---

## 🧠 Pro Tips

* `htop` is **fully configurable** → `F2` is your power menu
* Combine **tree view + sorting** for debugging fork bombs
* Tag multiple processes → kill once (safer & faster)
* Use **filter (`F4`)** instead of search for live narrowing

---

If you want, I can also:

* 🔧 Dump **your exact keymap** from `htoprc`
* 📋 Provide a **printable cheat sheet**
* ⚔️ Compare `htop` vs `btop` vs `atop`
* 🧠 Explain which columns matter most for debugging leaks

Just tell me 👍
