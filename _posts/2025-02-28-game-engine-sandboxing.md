---
title: 'Game Engine Sandboxing'
author: marco.lizza
layout: post
permalink: "/game-engine-sandboxing"
comments: true
categories:
  - tofu-engine
  - devlog
tags:
  - engine
  - lua
  - embedding
---
A game engine is much like an operating systems. Or, better, a (virtual) *guest* machine running inside a (physical) *host* machine.

This has been true since pretty much the very beginning of game-development and we could trace even in the '80s game in which the it's *core* was reusable (and reused) more than once. [Z-Machine](https://en.wikipedia.org/wiki/Z-machine) and [SCUMM](https://en.wikipedia.org/wiki/SCUMM) have been a forerunners on the topic (and, personally, one if not the single reason I wanted to learn how to code games in the first place... so, shame on them! :D).

However, it was with [DooM](https://en.wikipedia.org/wiki/Doom_(1993_video_game)) that the concept of a reusable game-engine could be said to have spread on a large scale (in addition to the concept of [WAD files](https://en.wikipedia.org/wiki/Doom_modding), another ‘passion’ of mine in those years).

With appropriate tools and means, anyone was given the opportunity to modify the basic logic of the game at will and even change it almost radically. In the vast majority of cases, there was a [DSL](https://en.wikipedia.org/wiki/Domain-specific_language) in the way, suitably constructed to fit like a glove.

Among the fundamental characteristics of the *host* machines of that time was that they were (1) isolated from other computers (we were still a long way from today's state in which every electronic device is potentially continuously connected to a network) and (2) strictly single-user. Security problems, therefore, were not many... except for a few sporadic computer viruses that caused a sensation, spreading like a cold by exchanging infected disks (I remember the first experiences with *boot sector viruses* in the Amiga, like *SCA Virus* and *Byte Bandit*).

For this reason, all things considered, it is not surprising that for a long time, in the context of game-engines, very few were ever interested in the possible related security problems.

However, with the advent of *general purpose* scripting languages such as [Lua](https://www.lua.org/), a little more care is required: what is known as **sandboxing** should be implemented, i.e. the idea of providing the user (in this case the programmer) with a controlled environment isolated from the *host* so that the integrity of the latter is guaranteed in any case.

For [Tofu Engine](https://tofuengine.org/) this was achieved gradually and naturally.

The first step concerns an appropriate access management for the file system. To this end, we set about writing an abstraction library which (among other things) completely inhibits the use of absolute *paths*. In the case of Lua's virtual machine, this also meant defining a custom *reader* and *searcher* (used for the resolution of the `require()` instruction). More specifically, we had to redefine the default Lua behaviour:

```c
// Lua default searchers are stored as four entries in the `package.searchers` table, as follows:
//
//   - a searcher that looks for a loader in the `package.preload` table,
//   - a searcher that looks for a loader as a Lua library,
//   - a searcher that looks for a loader as a C library,
//   - a searcher that looks for an all-in-one, combined, loader.
//
// The function modifies the table by clearing table entries #3 and #4. The first one is kept (to enable module
// reuse), and the second one is overwritten with the given `searcher`.
//
// As a result the module loading process is confined to the custom searcher only.
//
// See: https://www.lua.org/manual/5.4/manual.html#pdf-package.searchers
void luaX_overridesearchers(lua_State *L, lua_CFunction searcher, int nup)
{
    lua_getglobal(L, "package");        // A ... A -> A ... A T
    lua_getfield(L, -1, "searchers");   // A ... A T -> A ... A T T

    // Move the `searchers` and `package` tables *before* the upvalues so we can "close" them into
    // a function-closure. Then use the closure to override the 2nd searcher (keeping the "preloaded" helper).
    lua_insert(L, -(nup + 2));          // A ... A T T -> T A ... A T
    lua_insert(L, -(nup + 2));          // T A ... A T -> T T A ... A
    lua_pushcclosure(L, searcher, nup); // T T A ... A -> T T F
    lua_rawseti(L, -2, 2);              // T T F -> T T

    // Discard the others (two) searchers.
    lua_pushnil(L);
    lua_rawseti(L, -2, 3); // package.searchers[3] = nil
    lua_pushnil(L);
    lua_rawseti(L, -2, 4); // package.searchers[4] = nil

    lua_pop(L, 2); // Pop the `package` and `searchers` table.

    // Upvalues have already been consumed by `lua_pushcclosure()`. No need to clear the stack.
}
```

The second fundamental aspect concerns the functions (Lua) leaves available to the programmer. As already mentioned, since Lua is a *general-purpose* language, it also makes available APIs that are potentially dangerous (among all those relating to file-system access) and definitely not suitable in the context of a *sandboxed* game-engine we long to.

When implementing the FFI interfacing module (called `LuaX` in the context of my game-engine), I initially simply made a clone of the `luaL_openlibs()` function:

```c
void luaX_openlibs(lua_State *L)
{
    static const luaL_Reg libraries[] = {
        { LUA_GNAME, luaopen_base },
        { LUA_LOADLIBNAME, luaopen_package },
        { LUA_COLIBNAME, luaopen_coroutine },
        { LUA_TABLIBNAME, luaopen_table },
#if !defined(LUAX_NO_SYSTEM_LIBRARIES)
        // System libraries can be disabled to have a proper "sandbox" environment.
        { LUA_IOLIBNAME, luaopen_io },
        { LUA_OSLIBNAME, luaopen_os },
#endif  /* LUAX_NO_SYSTEM_LIBRARIES */
        { LUA_STRLIBNAME, luaopen_string },
        { LUA_MATHLIBNAME, luaopen_math },
        { LUA_UTF8LIBNAME, luaopen_utf8 },
#if defined(DEBUG)
        // Debug module is loaded only for the `DEBUG` build, of course.
        { LUA_DBLIBNAME, luaopen_debug },
#endif  /* DEBUG */
        { NULL, NULL }
    };
    for (const luaL_Reg *library = libraries; library->func; ++library) {
        luaL_requiref(L, library->name, library->func, 1);
        lua_pop(L, 1); // Remove the library (table) from the stack.
    }
}
```

The idea of replicating an existing function, although not optimal, seemed decent to me. In fact, for quite a while I continued to use it without any problems. Or any doubts it would be "enough".

Little by little, however, I realised that this approach was not the best possible: while the `io` library was certainly to be avoided at all costs, parts of the `os` and `debug` libraries might guarantee better reuse of existing code. Specifically, I realised this when trying out a [famous profiling module](https://2dengine.com/doc/profile.html).

Hence the idea to look into the matter. Almost by chance, at some point, I came across the description of how [Luau](https://luau.org/sandbox) implements *sandboxing*. So, why not take a cue from those who have already made considerations in this regard, not least because of the enormous diffusion of the language itself and its use in a context where security is mandated (i.e. [Roblox](https://en.wikipedia.org/wiki/Roblox))?

Consequently, instead of making a ‘personal variant’ of the `luaL_openlibs()` function, I opted to directly modify the code of the Lua implementation included in the game-engine, making the following changes:

* the `io.` library has been removed in its entirety;
* in the `os.` library the only supported functions are `clock`, `date`, `difftime`, and `time`;
* the `debug.` library has stripped down almost completely with only the `getinfo`, `sethook`, and `traceback` functions available;
* the `package.` library, as well, has been removed trimmed down a bit (in `Luau` it has been removed entirely) leaving only the `require` function and the `searcher` table-index available;
* the `dofile`, `loadfile` and `collectgarbage` Lua standard functions have been removed.

Unlike in Luau, however, no changes have been made to support for pre-compiled sources (i.e., included in pre-compiled P-code format). These can be included without any problems as generated by `luac` since the programmer is given ‘full confidence’ to include lawful code within the game he is making.

Although I don't particularly like making changes to the libraries I use, it is something I have occasionally found myself having to do in the course of a project (if only to resolve compilation warnings). In principle, therefore, I find this kind of approach tolerable given the (fundamental) objective of having a system as sandboxed as possible.

After all, the fact that Lua's codebase changes very rarely helps us in this endeavour. :)
