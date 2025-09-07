# Prefer to install binary

In macOS Homebrew, you can force `brew install` or `brew upgrade` to prefer installing a binary (bottle) instead of building from source using the `--formula` flag and avoiding options that trigger source builds. Here are the key methods:

### 1. **Use `--formula` (for new installs)**

```sh
brew install --formula <package>
```

This ensures Homebrew treats it as a formula and installs the precompiled binary when available.

### 2. **Use `--bottle` (for existing installs or upgrades)**

```sh
brew upgrade --bottle <package>
```

This instructs Homebrew to prefer bottles.

### 3. **Avoid `--HEAD` and `--build-from-source`**

- Do **not** use `--HEAD`, which forces Homebrew to build from the latest source.
- Do **not** use `--build-from-source`, which explicitly tells Homebrew to compile.

### 4. **Ensure Bottles Are Available**

- Check if a bottle is available before installation:
    
    ```sh
    brew info <package>
    ```
    
    If a bottle is available, the output will show something like:
    
    ```
    <package>: stable <version> (bottled)
    ```
    

### 5. **Set `HOMEBREW_NO_BOTTLE_SOURCE_FALLBACK=1`**

If you want to strictly avoid source builds, set this environment variable:

```sh
export HOMEBREW_NO_BOTTLE_SOURCE_FALLBACK=1
```

This prevents Homebrew from falling back to building from source if a bottle is unavailable.

### 6. **Check Why Homebrew Builds from Source**

If Homebrew still builds from source, check:

```sh
brew install <package> --debug
```

This may reveal missing dependencies or formulas marked as "no bottle."



# no deps

The `HOMEBREW_NO_INSTALLED_DEPENDENTS_CHECK` environment variable is used in Homebrew to skip checking for dependents (other packages that depend on the package you're working with) during certain operations, such as uninstalling or upgrading a package. This can be useful if you want to avoid the overhead of checking for dependents or if you're certain that no other packages depend on the one you're modifying.

### How to Use `HOMEBREW_NO_INSTALLED_DEPENDENTS_CHECK`

1. **Set the Environment Variable Temporarily**
   You can set the environment variable for a single command by prefixing the `brew` command with it:

   ```bash
   HOMEBREW_NO_INSTALLED_DEPENDENTS_CHECK=1 brew uninstall <package-name>
   ```

   Replace `<package-name>` with the name of the package you want to uninstall.

   Example:
   ```bash
   HOMEBREW_NO_INSTALLED_DEPENDENTS_CHECK=1 brew uninstall python
   ```

2. **Set the Environment Variable for the Current Shell Session**
   If you want to skip dependent checks for multiple commands in the same shell session, you can export the variable:

   ```bash
   export HOMEBREW_NO_INSTALLED_DEPENDENTS_CHECK=1
   brew uninstall <package-name>
   brew upgrade <another-package-name>
   ```

   This will apply to all subsequent `brew` commands in that terminal session.

3. **Set the Environment Variable Permanently**
   If you want to make this behavior persistent across all terminal sessions, you can add the export line to your shell's configuration file (e.g., `.bashrc`, `.zshrc`, etc.):

   ```bash
   echo 'export HOMEBREW_NO_INSTALLED_DEPENDENTS_CHECK=1' >> ~/.zshrc
   ```

   Then, reload your shell configuration:

   ```bash
   source ~/.zshrc
   ```

   Now, Homebrew will skip dependent checks for all future commands.

### When to Use `HOMEBREW_NO_INSTALLED_DEPENDENTS_CHECK`
- **Uninstalling Packages**: If you're sure no other packages depend on the one you're removing, you can skip the dependent check to speed up the process.
- **Upgrading Packages**: If you want to avoid the overhead of checking for dependents during an upgrade.
- **Custom Workflows**: If you're managing dependencies manually and don't want Homebrew to enforce dependency checks.

### Caveats
- Skipping dependent checks can lead to broken packages if other installed software depends on the package you're modifying.
- Use this option with caution, especially if you're not familiar with the dependency tree of your installed packages.

### Example Use Cases
4. **Uninstalling a Package Without Checking Dependents**:
   ```bash
   HOMEBREW_NO_INSTALLED_DEPENDENTS_CHECK=1 brew uninstall node
   ```

5. **Upgrading a Package Without Checking Dependents**:
   ```bash
   HOMEBREW_NO_INSTALLED_DEPENDENTS_CHECK=1 brew upgrade openssl
   ```

6. **Global Configuration**:
   Add the following to your shell configuration file to always skip dependent checks:
   ```bash
   echo 'export HOMEBREW_NO_INSTALLED_DEPENDENTS_CHECK=1' >> ~/.zshrc
   source ~/.zshrc
   ```

By using `HOMEBREW_NO_INSTALLED_DEPENDENTS_CHECK`, you can streamline Homebrew operations, but be mindful of potential dependency issues.