# 1
To install **Fontconfig using Homebrew** when the source server is blocked, you can manually place the `fontconfig-<version>.tar.gz` file into Homebrew’s cache directory so it can use it during installation. Here's how to do it:

---

### ✅ Step-by-Step Instructions

#### **1. Locate Homebrew’s Cache Directory**

Run this command to find the cache location:

```Shell
brew --cache
```

This will return something like:

```Shell
/Users/&lt;your-username&gt;/Library/Caches/Homebrew
```

---

#### **2. Find the Expected Filename**

Homebrew expects the tarball to have a specific name. You can find it by running:

```Shell
brew fetch fontconfig --build-from-source
```

This will fail due to the firewall, but it will show the expected filename and path.

Alternatively, check the expected URL with:

```Shell
brew info fontconfig
```

---

#### **3. Copy the File to the Cache**

Assuming you have `fontconfig-2.14.2.tar.gz`, copy it to the cache directory:

```Shell
cp /path/to/fontconfig-2.14.2.tar.gz $(brew --cache)/fontconfig--2.14.2.tar.gz
```

Make sure the filename matches exactly what Homebrew expects (you may need to rename it).

---

#### **4. Install Fontconfig from Source**

Now you can install it using:

```Shell
brew install fontconfig --build-from-source
```

Homebrew will use the cached tarball instead of downloading it.

---

### 🧠 Tips

- You can also use `brew edit fontconfig` to inspect or modify the formula if needed.
- If you’re behind a proxy, setting `http_proxy` and `https_proxy` environment variables might help Homebrew fetch files directly.

Would you like help verifying the exact version Homebrew expects or checking the formula file?