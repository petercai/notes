<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# how check dependencies version update info in a uv managed project

To check and update dependency versions in a project managed by **uv**, you can follow these steps and use the uv CLI commands designed for dependency management:

## Checking Dependency Versions and Updates in uv

- The dependencies and their version constraints are declared in the `pyproject.toml` file under the `[project]` section, using dependency specifiers compliant with PEP 508 (e.g., `requests>=2.31.0,<3.0.0`)[^1].
- The exact resolved versions of dependencies (including transitive dependencies) are recorded in the `uv.lock` file, which uv manages automatically to ensure reproducible environments[^4][^7].
- To see if updates are available, uv itself does not have a built-in command exactly like `poetry update` or `npm outdated` yet. However, you can upgrade dependencies individually or all at once with some commands described below[^2][^5].


## How to Update Dependencies in uv

- To upgrade a specific package to the latest compatible version according to your version constraints, run:

```
uv lock --upgrade-package <package-name>
```

This updates the lockfile and your environment for that package without changing other dependencies[^3].
- To add or change a dependency version, use:

```
uv add <package-name>[extras]==version
```

For example, to upgrade `requests` to version 2.31.0 explicitly:

```
uv add 'requests==2.31.0'
```

This updates `pyproject.toml` and the lockfile accordingly[^1][^3][^7].
- For adding multiple dependencies from a `requirements.txt` file (useful for migration), use:

```
uv add -r requirements.txt
```

- Currently, there is a feature request for a command like `uv upgrade` to automatically bump all dependency versions in `pyproject.toml` to the latest available, similar to Poetry's update command, but this is not yet implemented[^5].


## Summary of Recommended Workflow

1. Check your `pyproject.toml` for declared version constraints.
2. To update a single dependency, run:

```
uv lock --upgrade-package <dependency>
```

3. To explicitly set or change a dependency version, use:

```
uv add <dependency>==<version>
```

4. After updating dependencies, test your project to ensure compatibility.
5. Commit changes to `pyproject.toml` and `uv.lock` for reproducibility.

This workflow ensures your dependencies are updated in a controlled manner, with uv managing the lockfile and environment consistency.

This approach reflects the current uv capabilities as of mid-2025; more automated bulk upgrade features may be introduced in the future[^2][^3][^5].

<div style="text-align: center">⁂</div>

[^1]: https://docs.astral.sh/uv/concepts/projects/dependencies/

[^2]: https://github.com/astral-sh/uv/issues/6692

[^3]: https://docs.astral.sh/uv/guides/projects/

[^4]: https://realpython.com/python-uv/

[^5]: https://github.com/astral-sh/uv/issues/6794

[^6]: https://dev.to/joshhortt/how-to-keep-project-dependencies-up-to-date-1lm7

[^7]: https://www.datacamp.com/tutorial/python-uv

[^8]: https://docs.astral.sh/uv/reference/cli/

[^9]: https://python.plainenglish.io/managing-python-dependencies-with-uv-e6d2c52fd488?gi=c80f35263eea

[^10]: https://fossies.org/linux/uv/docs/concepts/projects/sync.md

[^11]: https://dev.to/thomas_bury_b1a50c1156cbf/mastering-python-project-management-with-uv-part-2-deep-dives-and-advanced-use-4mlb

[^12]: https://stackoverflow.com/questions/78902565/how-do-i-install-python-dev-dependencies-using-uv

[^13]: https://www.reddit.com/r/learnpython/comments/1kkw9ou/installing_dependencies_from_a_project_using_uv/

[^14]: https://www.youtube.com/watch?v=w0C1MwRAmHw

[^15]: https://fossies.org/linux/uv/docs/concepts/dependencies.md

[^16]: https://docs.astral.sh/uv/dependencies/

[^17]: https://fossies.org/linux/uv/docs/concepts/projects.md

