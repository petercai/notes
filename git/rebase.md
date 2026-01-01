If you want **“just pull it, unless there’s an actual conflict”**, and you **don’t care about unstaged or local changes**. This is a very common “force-but-safe” workflow.

### One-time global config

```bash
git config --global pull.rebase true
git config --global rebase.autoStash true
```

### Resulting behavior

```text
pull
├─ stash local changes
├─ rebase onto remote
├─ apply stash back
└─ stop ONLY if conflict exists
```

This is the **cleanest, least surprising** solution.

---

## ✅ Alternative: merge instead of rebase (also fine)

If you prefer merge commits instead of rebase:

```bash
git config --global pull.ff only
git config --global merge.autoStash true
```

Behavior:

* Fast-forward only
* Auto-stash local changes
* Fail only when conflict is unavoidable

---

## 🧠 Why `git pull` fails by default

By default Git:

* Refuses to overwrite unstaged changes
* Tries merge without stashing
* Stops early even when conflict *would not* happen

That’s why `autoStash` exists.

---

## ⭐ Recommended final setup (copy & paste)

This is what most senior devs use:

```bash
git config --global pull.rebase true
git config --global rebase.autoStash true
git config --global rebase.updateRefs true
```
