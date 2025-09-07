# install from github

To install a Python package directly from a GitHub repository using `pip`, you can use the following syntax:

```bash
pip install git+https://github.com/<username>/<repository>.git
```

Replace `<username>` with the GitHub username and `<repository>` with the name of the repository.

### Examples

1. **Install from the Main Branch**:
   ```bash
   pip install git+https://github.com/<username>/<repository>.git
   ```

   Example:
   ```bash
   pip install git+https://github.com/psf/requests.git
   ```

2. **Install from a Specific Branch**:
   If you want to install from a specific branch, append `@<branch-name>` to the URL:

   ```bash
   pip install git+https://github.com/<username>/<repository>.git@<branch-name>
   ```

   Example:
   ```bash
   pip install git+https://github.com/psf/requests.git@develop
   ```

3. **Install from a Specific Tag or Commit**:
   If you want to install from a specific tag or commit, append `@<tag-or-commit-hash>` to the URL:

   ```bash
   pip install git+https://github.com/<username>/<repository>.git@<tag-or-commit-hash>
   ```

   Example (using a tag):
   ```bash
   pip install git+https://github.com/psf/requests.git@v2.25.1
   ```

   Example (using a commit hash):
   ```bash
   pip install git+https://github.com/psf/requests.git@58a2c1a
   ```

4. **Install from a Private Repository**:
   If the repository is private, you can use the following format and ensure you have the necessary permissions:

   ```bash
   pip install git+https://<username>:<token>@github.com/<username>/<repository>.git
   ```

   Replace `<username>` with your GitHub username, `<token>` with a GitHub personal access token, and `<repository>` with the repository name.

   Example:
   ```bash
   pip install git+https://username:ghp_your_token_here@github.com/username/repository.git
   ```

### Notes
- Ensure you have `git` installed on your system, as `pip` uses `git` to clone the repository.
- If the repository contains a `setup.py` or `pyproject.toml` file, `pip` will automatically install the package and its dependencies.
- If you encounter issues, make sure the repository is publicly accessible (or that you have the correct credentials for private repos).

This method is useful for testing the latest changes in a repository or using a specific version of a package that hasn't been released to PyPI.