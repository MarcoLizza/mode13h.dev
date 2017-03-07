---
title: Stateful Callbacks in Lua
author: marco.lizza
layout: post
permalink: /stateful-callbacks-in-lua/
thumbnail: flask
categories:
  - embedding
tags:
  - engine
  - lua
  - scripting
---
Lua offers quite a neat API to interact with, both for embedding and extending it.

However, when it come to extending all we have access to is a single callback defined as follows.

```c
typedef int (*lua_CFunction) (lua_State *L);
```

This is more than enough to add a single-shot function with no side-effects whatsoever that just depends upon the arguments themselves (as it happens with the shipped functions, such us the `math.h` module).

Sadly, in a more generic and broad context, this might seem lacking of something.

# Let's proceed by steps

Consider, for example, the common case in which you want to expose some kind of functionality implemented by the host for the Lua scripts the be accessed.

```c
#include <lua.h>
#include <stdlib.h>

static int l_salute(lua_State* state)
{
    printf("Hello!\n");
    return 0;
}

int main(int argc, char *argv[])
{
    lua_State *state = luaL_newstate();

    lua_register(state, "salute", l_salute);

    luaL_dostring(state, "salute()");

    lua_close(state);

    return 0;
}
```

So far, so good. Our application can salute us nicely, albeit every time in the same way. What happens if we want to output a non-constant string, for example to greet us by using our name?

The naive solution is to use a global variable to hold the user name and change the variable content prior calling the function.

```c
#include <lua.h>
#include <stdlib.h>

static const char _name[32] = { '\0' };

static int l_salute(lua_State* state)
{
    printf("Hello, %s!\n", _name);
    return 0;
}

int main(int argc, char *argv[])
{
    lua_State *state = luaL_newstate();

    lua_register(state, "salute", l_salute);

    strcpy(_name, "Bruce");
    luaL_dostring(state, "salute()");

    lua_close(state);

    return 0;
}
```

That is going to work... but quite frankly nobody likes global variables. Even solving the problem with more exotic approaches, such as using the *Singleton Pattern* or some variant, just plain sucks!

# So, what's missing?

If we ask to ourselves what's missing in the aforementioned approach we will eventually point out that the callback function does not have some kind of *generic* argument that goes along with the callback itself and defines its context.

We would like to have something like that

```c
typedef int (*luaEx_CFunction)(lua_State*, void*);
```

And, in a similar fashion, the registering function should display a signature as follows

```c
void luaEx_register(lua_State*, const char*, luaEx_CFunction, void*);
```

This would eventually lead to the following code.

```c
#include <lua.hpp>
#include <stdlib.h>

static int l_salute(lua_State* state, void* parameter)
{
    char* name = (char*)parameter;
    printf("Hello, %s!\n", name);
    return 0;
}

int main(int argc, char *argv[])
{
    lua_State *state = luaL_newstate();

    char name[32] = { '\0' };
    luaEx_register(state, "salute", l_salute, name);

    strcpy(_name, "Bruce");
    luaL_dostring(state, "salute()");

    lua_close(state);

    return 0;
}
```

# How to solve this?

Simply speaking, the concept of sticking a *function* to a *context* is called *closure*, and Lua does support them since version 3.1, also from the embedding API. With that in mind, we can register a function and bind the generic parameter to it by means of a closure.

```c
// Register a new global [function], by creating a closure with the passed
// name bound to the common dispatcher, encapsulating the real function pointer
// and the passed parameter.
//
// We are using the light-userdata datatype since we are not going to need to
// interact with the garbage-collection for its management.
void luaEx_register(lua_State* state, const char* name, luaEx_CFunction function, void* parameter)
{
    lua_pushlightuserdata(state, parameter);
    lua_pushlightuserdata(state, function);
    lua_pushcclosure(state, luaEx_dispatcher, 2);
    lua_setglobal(state, name);
}
```

A peculiar aspect of this approach is that we are needed to provide a *dispatcher*, that is a common utility function that unpacks the closure and calls the user-provided callback with the additional argument on (in a similar way as described [here][1] and [here][2]).

```c
// This is a static (common) dispatching function that bounces to
// the intended callback function, passing the additional parameter.
static int luaEx_dispatcher(lua_State* state)
{
    void* parameter = (void*)lua_touserdata(state, lua_upvalueindex(1));
    luaEx_CFunction callback = (luaEx_CFunction)lua_touserdata(state, lua_upvalueindex(2));
    return callback(state, parameter);
}
```

> It is important to note that we are binding the parameter to the Lua function identifier. Every call will use the very same pointer. Please, keep it in mind when dealing with it in order not to end with a dangling memory pointer!

# That's it!

This solution is quite basic and cannot hold a candle to other more complex (and more comprehensive) extension libraries.

It may seem *too naive* but, in fact, our aim was to implement the single simplest method to have access to a parameterizable *context* in the callback function.

We didn't want to implement a complex library, expose a C++ class to be used in Lua as a native object, or something on this league. I honestly prefer to avoid this approach, but expose simpler C-like primitives and create the object-oriented abstractions script-side.

 [1]: http://www.gamedev.net/page/resources/_/technical/game-programming/exporting-c-functions-to-lua-r2629
 [2]: http://lua-users.org/lists/lua-l/2000-01/msg00051.html