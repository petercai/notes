# 1 Xet Storage

When you see **“Xet Storage is enabled for this repo”**, it usually means the repository is using **[XetHub/Xet](https://xethub.com/)** (by XetData) as a **storage backend** in addition to Git.

Here’s the breakdown:

### 1. What Xet is

- Xet is like a Git extension for **large files, datasets, and machine learning model weights**.
    
- Similar to **Git LFS**, but optimized for very large files and high-throughput workflows.
    
- Instead of bloating your Git repo with multi-GB binaries, it stores them in **Xet Storage** and only keeps lightweight pointers inside Git.
    

### 2. What happens when “Xet Storage is enabled”

- Your repo can now handle large assets (datasets, model checkpoints, media, etc.) seamlessly.
    
- Files tracked by Xet are stored in Xet’s backend (content-addressable, deduplicated storage).
    
- Clones, pulls, and pushes will fetch these big files from Xet rather than standard Git.
    
- This makes your Git repo faster, smaller, and easier to collaborate on.
    

### 3. Typical use cases

- **Machine learning projects** → datasets, trained models.
    
- **Audio/video repos** → media files that would normally be too big for GitHub.
    
- **Scientific projects** → large CSVs, simulations, etc.
    

### 4. Developer experience

- You still interact with the repo like normal (`git clone`, `git pull`, etc.).
    
- Behind the scenes, Xet fetches the big files from Xet Storage.
    
- No need to worry about Git history ballooning because of large binaries.
    

👉 In short:  
**“Xet Storage is enabled for this repo” means the repository is configured to use Xet’s large-file storage backend, so it can manage and version very large files efficiently without bloating Git itself.**

Do you want me to also compare it directly with **Git LFS**, so you know when Xet is better?