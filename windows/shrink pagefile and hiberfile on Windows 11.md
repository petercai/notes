Here are the **safe and correct ways** to shrink **pagefile.sys** and **hiberfil.sys** on Windows 11.

---

# ✅ **1. Reduce pagefile.sys size**

**pagefile.sys = virtual memory**.  
Windows sets it automatically, but you can manually reduce it.

### **Steps**

1. **Win + R →** type `sysdm.cpl` → Enter
    
2. Go to **Advanced** tab
    
3. Under **Performance**, click **Settings**
    
4. Go to **Advanced** tab
    
5. Under **Virtual memory**, click **Change…**
    
6. Uncheck **Automatically manage paging file size for all drives**
    
7. Choose your **C:** drive
    
8. Select **Custom size**
    
9. Set values (recommended below), then **Set → OK → Reboot**
    

### **Safe pagefile sizes**

|RAM Size|Recommended Pagefile|
|---|---|
|8 GB|2–4 GB|
|16 GB|2–8 GB|
|32 GB|2–8 GB|
|64 GB+|2–4 GB (usually enough)|

If your system has **16GB RAM**, setting pagefile to **2048–4096 MB** is usually safe unless you run very heavy apps.

---

# ✅ **2. Shrink or remove hiberfil.sys**

**hiberfil.sys** is used by:

- **Hibernate**
    
- **Fast Startup** (even if you _don’t_ use full hibernation)
    

Removing or reducing _hiberfil.sys_ requires **Command Prompt (Admin)**.

---

## **Option A — Reduce hiberfil.sys (keep Fast Startup)**

This shrinks it from ~40% of RAM → ~20% of RAM.

1. **Run CMD as Administrator**
    
2. Run:
    

```
powercfg /hibernate /type reduced
```

This usually shrinks the file by **40–60%**.

---

## **Option B — Turn off Hibernate (delete hiberfil.sys)**

This deletes the file entirely.

1. **Run CMD as Administrator**
    
2. Run:
    

```
powercfg -h off
```

This removes **hiberfil.sys** immediately.

Effects:

- ❌ Hibernate disabled
    
- ❌ Fast Startup disabled
    
- ✔️ frees 6–15 GB depending on RAM
    

---

# 🔎 Why are your files so big?

- **pagefile.sys = 11 GB** → Windows is auto-managing and probably set it near your RAM size
    
- **hiberfil.sys = 6.2 GB** → Typical for **Reduced mode** if you have ~16GB RAM  
    (Full mode would be ~10–12GB)
    

---

# 💡 Recommended Setup (most users)

If you want to save space but keep stability:

### **Pagefile**

```
Initial size: 2048 MB
Maximum size: 4096 MB
```

### **Hibernate**

```
powercfg /hibernate /type reduced
```

This keeps Fast Startup working and usually reduces hiberfil.sys to ~3–5GB.

---

If you want, tell me:

- How much **RAM** you have
    
- How much **free disk space** you want to reclaim
    

I can recommend **exact safe sizes** for your system.