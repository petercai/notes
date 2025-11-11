Windows 11 has quietly hidden “Credential Manager” deeper in Settings, but it’s still there.  
Here are **all the ways to open it** depending on what’s most convenient for you:

---

### 🧭 Option 1 — Search directly

1. Press **⊞ Win key**.
    
2. Type **Credential Manager**.
    
3. Click the result **“Credential Manager (Control Panel app)”**.
    

Even though Windows 11 pushes you toward the new Settings app, this still opens the classic Control Panel tool.

---

### ⚙️ Option 2 — Run command

Press **Win + R**, type:

```
control /name Microsoft.CredentialManager
```

and hit **Enter**.  
→ This jumps straight to **Control Panel › Credential Manager**, even in Windows 11.

---

### 💾 Option 3 — Navigate manually (classic Control Panel)

If you prefer menus:

1. Open **Control Panel** (search “Control Panel”).
    
2. Set **View by → Large icons** or **Small icons**.
    
3. Click **Credential Manager** → choose **Windows Credentials** tab.
    
4. Look for entries such as  
    `git:https://github.com` or `git:https://<org>@github.com`.
    

You can **edit** or **remove** them to reset which GitHub account Git / LazyGit uses.

---

### 🧰 Option 4 — Use PowerShell (no GUI)

You can query or clear credentials without opening any window:

```powershell
# List stored credentials (requires Git Credential Manager Core)
git credential-manager list
# Clear GitHub credentials
git credential-manager clear https://github.com
```

---

### ✅ Tip

After you remove or clear the GitHub entry, the next `git push` from LazyGit will prompt a fresh login where you can pick a new account.

---

Would you like me to show how to make Git / LazyGit remember **two different GitHub accounts** (personal + work) without constantly switching in Credential Manager?