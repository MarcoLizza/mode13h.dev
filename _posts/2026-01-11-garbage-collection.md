---
title: Garbage Collection
layout: post
permalink: /garbage-collection/
categories:
  - tofu-engine
  - devlog
tags:
  - engine
  - lua
  - garbage-collector
---
Recently, I upgraded [Lua](https://www.lua.org/versions.html#5.5) from version `5.4.x` to `5.5.0`. It has been a long-awaited minor release for our beloved embeddable scripting language. While running a few demos (just to make sure everything was behaving correctly) I noticed that, in at least two cases, a significant amount of memory kept accumulating without being released... to the point where the `boids` demo would freeze in less than a minute.

A closer inspection revealed that the temporary objects created during the `update()` step were continuously piling up. But why?

As it turned out, the engine's *continuous* garbage-collection mode was poorly implemented. First of all, it was unnecessarily overcomplicated: there is no real need to track an arbitrary time window just to decide when to trigger a GC step. Moreover, the `GC_MODE_CONTINUOUS` mode itself was flawed, because the initial implementation did not take into account that executing a single `LUA_GCSTEP` roughly every ~15 seconds is not sufficient to keep up with aggressive allocation patterns. Memory is effectively reclaimed only at the end of a full collection cycle, so the backlog could easily grow out of control.

After some simplification and a fair amount of testing, I came to the conclusion that the **generational** strategy works well when it is left entirely to the virtual machine, but when finer control is required the **incremental** garbage collector is once again the better choice. By performing a single collection step at a lower-priority update frequency (for example, one step every four `update()` calls), memory usage remains stable even when large numbers of short-lived objects and tables are created.

I also experimented with running the garbage-collection in short "bursts", instead of single steps. In theory, `LUA_GCSTEP` can be invoked in a loop until it returns a non-zero value, signaling that a full cycle has completed. However, with the *generational* collector this loop only terminates when a *major collection* is executed. If the GC keeps operating in *minor collection* mode, the loop may never end, effectively spinning forever. This means some form of capping is required, for example:

```c
void luaX_gccycle(lua_State *L, int max_steps)
{
    for (;;) {
        if (max_steps >= 0 && max_steps-- == 0) { // To be used in generational GC mode
            break;
        }
        int result = lua_gc(L, LUA_GCSTEP);
        if (result == 1) {
            break;
        }
    }
}
```

That said, the actual behavior remains configurable (see the `config.h` file), but the policy of choice is:

* Lua's GC is stopped immediately, in order to disable automatic and uncontrolled garbage-collection triggers.
* The *incremental* garbage-collection mode is explicitly selected.
* A single `LUA_GCSTEP` is executed during the low-priority update step.

A nice additional feature is that, when compiled with `TOFU_INTERPRETER_GC_MODE` set to `GC_MODE_CONTINUOUS`, the game engine automatically checks the state of the garbage collector and performs a `LUA_GCSTEP` only when the GC is stopped. This makes it possible, if needed, to switch back to an "automatic" mode at run time with minimal effort, without the engine continuing to drive the collection explicitly.

Finally, a useful byproduct of this work has been a refinement of the run-time debug information related to heap usage. The internal period counter was slightly off, and the VM memory usage was not being tracked accurately (nor was it accessible from Lua). These issues have now been cleaned up, and `System.heap()` correctly reports both system and VM memory consumption.

( I'm also considering to add a graphical overlay to report the current performance information )

This naturally led to a further consideration: when dealing with time-varying metrics, it is often useful to apply a [moving average](https://en.wikipedia.org/wiki/Moving_average)... but which variant should one choose? Let's explore that in the next post! :)
