

You can change Git commit times (both **author date** and **commit date**) using several methods — depending on whether you want to modify **just the last commit**, **a few recent commits**, or **an entire history**.

Let’s go through them step-by-step 👇

---

## 🕓 1. Change the **last commit’s** time

### Option A — Edit with `--amend`

```bash
# change both author and commit date to current time
git commit --amend --no-edit --date="$(date)"
```

### Option B — Set a specific timestamp

```bash
GIT_COMMITTER_DATE="2025-10-17 14:30:00" \
git commit --amend --no-edit --date "2025-10-17 14:30:00"
```

💡 This updates both:

- `AuthorDate` → the time the author originally wrote the code
    
- `CommitDate` → the time the commit was created
    

---

## 🕓 2. Change **multiple past commits** (interactive rebase)

If you want to rewrite several commit timestamps:

```bash
git rebase -i HEAD~5
```

This opens an editor like:

```
pick a1b2c3d Commit 1
pick e4f5g6h Commit 2
pick i7j8k9l Commit 3
...
```

Change the commits you want to edit to:

```
edit a1b2c3d Commit 1
```

Then for each commit:

```bash
git commit --amend --no-edit --date="2024-09-01 09:00:00"
git rebase --continue
```

Repeat for each one.

---

## 🕓 3. Change **all commits in a branch** automatically

You can run a **filter-branch** or (better) **git filter-repo** script.

### Using `git filter-repo` (recommended)

```bash
pip install git-filter-repo
git filter-repo --env-filter '
export GIT_AUTHOR_DATE="2025-10-17T10:00:00"
export GIT_COMMITTER_DATE="2025-10-17T10:00:00"
'
```

This rewrites all commits with the specified time (or you can use logic to vary by commit).

---

## ⚠️ Important Notes

- These commands **rewrite history** — use only on **private branches** or branches not yet pushed/shared.
    
- If already pushed, you must **force push**:
    
    ```bash
    git push --force
    ```
    
- Always **make a backup branch** before doing history rewrites.
    

---

## ✅ Quick Reference Summary

| Goal                                  | Command                                                                                              |
| ------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Change last commit time to now        | `git commit --amend --no-edit --date="$(date)"`                                                      |
| Change last commit to a specific time | `GIT_COMMITTER_DATE="YYYY-MM-DD HH:MM:SS" git commit --amend --no-edit --date "YYYY-MM-DD HH:MM:SS"` |
| Change several commits                | `git rebase -i HEAD~N` + amend each                                                                  |
| Change all commits in repo            | `git filter-repo --env-filter 'export GIT_AUTHOR_DATE=...; export GIT_COMMITTER_DATE=...'`           |

---

Would you like me to show how to **randomize** commit times (e.g., simulate natural work history)?