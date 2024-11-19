#python #asyncio #performance
I use [[FastAPI]] at work, which is an asynchronous python web server framework built for performance. It natively supports [[asyncio]], and there's a growing [ecosystem of asyncio-based libraries](https://github.com/timofurrer/awesome-asyncio) for various parts of the web stack.

[[asyncio]] is built around the notion of an [[event loop]], which lets concurrent functions co-operatively give up control when they need to just wait around for a while (e.g. for a network operation to finish, or for some resource to become available). This can help reduce CPU cycles wasted doing 'nothing' – one concurrent function can yield control to others while it's waiting on something like a network call to complete.

![[Drawing 2023-09-29 22.40.52.excalidraw.dark.svg|600]]
%%[[Drawing 2023-09-29 22.40.52.excalidraw.md|🖋 Edit in Excalidraw]], and the [[Drawing 2023-09-29 22.40.52.excalidraw.light.svg|light exported image]]%%

While a powerful tool, there's a couple of things that keep developers from making the most of asyncio's capabilities:

1. Real-world codebases are not fully asyncio-compatible, meaning there’s a mix of async and non-async (blocking) functions.
2. Calling a blocking function in an async function can cripple an asyncio app by hogging the entire event loop – no other coroutines can run while the blocking function is just waiting for something like a network request, even if there’s plenty of CPU cycles to go around.
3. There's no easy way to figure out whether a given `async` function might be calling a blocking function.

There’s certainly ways to run blocking tasks in asyncio – offloading to a thread using asyncio's `loop.run_in_executor()` (for CPU-bound tasks you’d want to offload to a separate process instead because [[Python GIL|GIL]]). But the lack of tooling around this makes it very easy to shoot yourself in the foot by accidentally calling a blocking function in an async context.