---
---

#asyncio #python #performance
In [[AsyncIO's blocking call problem]] I describe a very easy way for developers working with [[asyncio]] to accidentally cripple their app's performance.

asyncio's [debug mode](https://docs.python.org/3/library/asyncio-dev.html#debug-mode) enables a few additional checks at runtime, some of which help identify performance issues:

- Callbacks taking longer than 100 milliseconds are logged
- The execution time of the I/O selector is logged if it takes too long to perform an I/O operation

To see how the asyncio debug logging works, I've written a simple script demonstrating a couple of common footguns when using asyncio:

```python
import asyncio
import time

async def foo():
    # An async function that calls a blocking function
    time.sleep(1)

def bar():
    # A straight up blocking function
    time.sleep(1)

async def baz():
    # An async function that correctly wraps a blocking call in an executor
    loop = asyncio.get_running_loop()
    await loop.run_in_executor(None, time.sleep, 1)

async def main():
    await foo()
    bar()
    await baz()

if __name__ == "__main__":
    asyncio.run(main(), debug=True)
```

Calls to functions like `foo` and `bar` can cripple the performance of asyncio apps – they call the blocking function `time.sleep` without offloading it to a thread. In this toy example with no actual concurrency this shouldn't have any real impact on execution time, but it _should_ trigger some debug logs from asyncio because we'd be blocking the event loop for 1 second at each un-wrapped blocking call.

Specifically, I'd _like_ to see asyncio tell us that the calls to `foo` and `bar` exceeded the 100ms threshold. Let's try running the script:

```
Executing <Task pending name='Task-1' coro=<main() running at script.py:23> wait_for=<Future pending cb=[_chain_future.<locals>._call_check_cancel() at python/3.11.4/lib/python3.11/asyncio/futures.py:387, Task.task_wakeup()] created at python/3.11.4/lib/python3.11/asyncio/base_events.py:427> cb=[_run_until_complete_cb() at python/3.11.4/lib/python3.11/asyncio/base_events.py:180] created at python/3.11.4/lib/python3.11/asyncio/runners.py:100> took 2.012 seconds
```

...what

The log suggests `main()` blocked for a long time (2.012 seconds), but doesn't tell us anything more specific.

> [!note]
> It's understandable that asyncio couldn't specifically catch the blocking call to `bar` – my guess is that it only records execution time when switching contexts at `await` statements.
> If that's the case, I would have expected at least `foo` to be identified by name as blocking for a long time.

These are also runtime-only checks, which means that unless your test suite's code coverage is close to 100% it's possible to miss issues before they make it to production.

---

# Monitoring in production

[[APM]] tools like [[Sentry]] and [[New Relic]] can help identify performance bottlenecks in

---

# Related

- I explore building a static analyzer for blocking asyncio calls in [[Static analysis of blocking calls in async Python]] to get around some of these issues
- [[AsyncIO's blocking call problem]]
- [[Rust]] seems to have the same problem: https://github.com/rust-lang/wg-async/issues/19
- https://stackoverflow.com/questions/57190115/discover-what-is-blocking-the-event-loop
