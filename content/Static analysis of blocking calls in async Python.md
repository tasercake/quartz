---
---

#python #asyncio #static-analysis

**This page explores solutions to [[AsyncIO's blocking call problem]]**

We'd like to statically analyze a codebase to identify calls to blocking functions inside an async context. The shape of the solution is likely a plugin for [[mypy]] or a standalone CLI tool.

## Requirements

The tool would ideally satisfy the following requirements:

- Be able to manually annotate known blocking functions with something like a monadic `IOBlocking` type (e.g. DB call returns `IOBlocking[Dict[str, int]]`)
- For a start, we could manually annotate all known blocking calls, then propagate those blocking annotations to any functions that call them.
- Automagically know which stdlib functions (and common 3rd party lib functions) are blocking
- Provide escape hatches for when we _know_ a function is non-blocking (or don't care that it does)

Some stuff for me to figure out before designing a solution:

- Could a standalone tool do this? What does it take to recursively scan a large codebase?
- How does mypy work under the hood?
- Is it possible to build a plugin that does this? Or must it be part of mypy core?
- How would blocking analysis work with inheritance?

[This GitHub discussion](https://github.com/python/mypy/issues/12649) goes over most of these issues and suggests a few ways to enable [[mypy]] to help with this problem.

### An imperfect solution: Manual runtime blocking analysis

Django has a [`async_unsafe` decorator](https://github.com/django/django/blob/38ad710aba885ad26944ff5708ce1a02a446d2d3/django/utils/asyncio.py#L5) to manually wrap functions that are known to be unsafe to run directly in async contexts. This appears to provide the benefit of not having to keep track of nested sync/async contexts because it uses [`asyncio.get_running_loop()`](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio.get_running_loop) which immediately tells us whether we're running in a context where it's unsafe to run blocking functions.
The implementation is roughly as follows:

```python
from asyncio import get_running_loop
from functools import wraps

from typing import Callable, ParamSpec, TypeVar

P = ParamSpec("P")
R = TypeVar("R")


def async_unsafe(f: Callable[P, R]) -> Callable[P, R]:
    @wraps(f)
    def wrapped(*args: P.args, **kwargs: P.kwargs) -> R:
        # Check if the current thread has a running event loop
        try:
            get_running_loop()
        except RuntimeError:
            pass
        else:
            raise Exception(f"Unsafe to run {f.__name__} in an async context.")
        return f(*args, **kwargs)

    return wrapped
```
