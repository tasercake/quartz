---
---

[[mypy]]'s config file '`mypy.ini` supports an [exclude](https://mypy.readthedocs.io/en/stable/config_file.html#confval-exclude) option, which is a [[regular expression|regex]] that determines which files to exclude from analysis. This provides a low-friction path to introducing static analysis to an existing (untyped) Python codebase.

However as the codebase evolves & more type safety is built in, there's currently no easy way to determine which (if any) files are _unnecessarily_ excluded from analysis (i.e. files that could be included in analysis without triggering any errors or warnings). I wonder if we could add a flag to mypy to warn of files that can safely be included in analysis?

A point of reference here is the [`warn_unused_ignores`](https://mypy.readthedocs.io/en/stable/config_file.html#confval-warn_unused_ignores) option, which flags up `# type: ignore` comments in source code that can safely be removed.

Some potential approaches:

- Run analysis over all files (as if the `exclude` option didn't exist), then test the name of every file

A few questions to answer first:

- Does the `exclude` option double as a performance optimization?
-
