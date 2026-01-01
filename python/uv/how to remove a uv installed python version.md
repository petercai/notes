<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# how to remove a uv installed python version

To remove a Python version installed by **uv**, you can try the following steps. However, as of now, there seems to be an issue with uninstalling specific Python versions using the `uv` command, as reported in GitHub issues[^1]. Here's a general approach you can take:

1. **List Installed Versions**: First, list all the installed Python versions managed by `uv`:

```bash
uv python list --only-installed
```

2. **Attempt Uninstallation**: Try to uninstall the specific Python version using the `uv` command. Note that this might not work due to the reported issues:

```bash
uv python uninstall &lt;version&gt;
```

For example:

```bash
uv python uninstall 3.12
```

3. **Manual Removal**: If the above command does not work, you might need to manually remove the Python version. You can find the installation directory using:

```bash
uv python dir
```

Then, manually delete the specific version from this directory. However, be cautious with manual removals to avoid breaking other configurations.
4. **Clean Up**: After manual removal, you might want to clean up any leftover data:

```bash
uv cache clean
```

5. **Check Documentation**: Keep an eye on the official `uv` documentation for updates on uninstallation procedures, as this functionality is still evolving[^4].

If you encounter issues, consider reaching out to the `uv` community or checking GitHub for updates on this feature.

<div>⁂</div>

[^1]: https://github.com/astral-sh/uv/issues/8751

[^2]: https://docs.astral.sh/uv/guides/install-python/

[^3]: https://docs.astral.sh/uv/concepts/python-versions/

[^4]: https://stackoverflow.com/questions/78076401/uninstall-uv-python-package-installer

[^5]: https://www.datacamp.com/tutorial/python-uv

[^6]: https://www.jennapederson.com/blog/managing-python-versions-and-packages-with-uv/

[^7]: https://www.reddit.com/r/PythonLearning/comments/1iddage/please_help_me_uninstall_all_the_python_versions/

[^8]: https://github.com/astral-sh/uv/issues/8135

