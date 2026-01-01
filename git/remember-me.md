On Linux, the **right way** to remember Git server username/password is to use a **credential helper**. Git has this built in — you just need to enable the one you want.

Below are the **practical options**, from most common → most secure.

---

## ✅ Option 1 (Recommended): Cache credentials in memory (timeout-based)

Good for servers, SSH boxes, CI-like workflows.

```bash
git config --global credential.helper cache
```

By default:

* Keeps username/password in memory
* Expires after **15 minutes**

### Customize timeout (e.g. 8 hours)

```bash
git config --global credential.helper "cache --timeout=28800"
```

✔ No disk storage
✔ Very simple
✖ Lost after reboot

---

## ✅ Option 2: Store credentials **plain text** (persistent)

If this is a **trusted machine** (VM, private server):

```bash
git config --global credential.helper store
```

After next successful login, Git writes to:

```text
~/.git-credentials
```

Example content:

```text
https://username:password@git.example.com
```

✔ Survives reboot
✔ Zero extra dependencies
❌ Password stored **unencrypted**

> 🔥 Use only if you understand the risk.


---

## 🧠 What I recommend (TL;DR)

| Environment         | Recommendation            |
| ------------------- | ------------------------- |
| Personal laptop     | keyring (libsecret)       |
| Private server / VM | `credential.helper store` |
| Temporary shell     | `credential.helper cache` |
| Long-term / pro use | **SSH keys**              |

---
