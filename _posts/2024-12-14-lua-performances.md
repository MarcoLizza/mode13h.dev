---
title: 'A Case for Lua Performance'
author: marco.lizza
layout: post
permalink: "/a-case-for-lua-performance"
comments: true
categories:
  - rants
tags: 
  - lua
  - optimization
  - embedding
---
When using an embedded scripting language in a game-engine (or in any other real-time operating software) one should always take into account **performances**.

Even in the case of a JIT interpreter, scripted code more often that not loses when compared to native code (e.g. resulting from compiled C99 code). This is not only due to the fact that [P-code](https://en.wikipedia.org/wiki/P-code_machine) can't be faster than executing machine code, but also because of the different impact the data structure and memory usage have on the CPU.

As a rule-of-a-thumb, common sense suggest that complex algorithms should be implemented natively. Python, for example, proves this on daily basis with OpenCV: one can dismiss the language as "slow" (which is not, anyway), but the OpenCV library runs as native code and offers near real-time performances.

There are, however, other cases where it's a tad more difficult to draw the line between what should be left on scripting side and what should be implemented as natively.

## A vector class

Any self-respecting game-engine almost certainly feature an abstraction of the (either 2D or 3D) vector concept. In the case of **Tofu Engine** this is a strictly a two-dimensional vector, implemented as a class, with an idiomatic form of this type.

```lua
local Vector = {}

Vector.__index = Vector

function Vector.new(x, y)
  local self = setmetatable({}, Vector)
  self.x = x
  self.y = y
  return self
end

-- Methods will be added here...

return Vector
```

> Actually, in **Tofu Engine** there is an easier way to implement a class (i.e. via the `class.lua` module), but the substance does not change.

As a direct consequence every instance of the `Vector` class will be a Lua table. Let's just keep this in mind and add some more methods.

```lua
local Vector = {}

Vector.__index = Vector

function Vector.new(x, y)
  local self = setmetatable({}, Vector)
  self.x = x
  self.y = y
  return self
end

function Vector:add(v)
  self.x, self.y = v.x, v.y
end

function Vector:dot(v)
  return self.x * v.x + self.y * v.y
end

function Vector:magnitude(v)
  local x, y = self.x, self.y -- Well know Lua tip! Use local variables to save table accesses!
  local m = x * x + y * y
  if m == 0.0 then
    return 0.0
  end
  return m ^ 0.5 -- Tip! Avoid FFI call to `math.sqrt()`!
end

function Vector:normalize(v)
  local m = self:magnitude()
  if m == 0.0 then
    return
  end
  self.x, self.y = x / m, y / m
end

function Vector:angle_to(v)
  return math.atan(v.y - self.y, v.x - self.x)
end

-- More methods would follow here...

return Vector
```

We have added only five methods, as they sum up pretty well three method categories of increasing computational impact:

- two methods that perform simple operations using only Lua code (`add()` and `dot()`),
- two more complex Lua-code-only methods (`magnitude` and `normalize()`, exploiting the equivalence `sqrt(x) == pow(x, 0.5)`),
- a simple method that relies on an FFI function (`angle_to()`).

Taking a look at the full [implementation](https://github.com/tofuengine/tofu/blob/master/src/kernal/tofu/util/vector.lua) straight from the game-engine shows that, in principle, all methods fall into one of these categories.

## Finding the costs

It well known that *<< premature optimization is the root of all evil >>*. We simply can't proceed and blindly *optimize* without proper data.

A simple (but effective) way evaluate the cost of a piece of code is to run it a given number of times and measure the time it took (also on average, if we are interested in that). In Lua, this can be achieved with a helper function.

```lua
-- `closure` (function) :- the function closure we want to profile
-- `tag` (string) :- string identifier used for debug
-- `rounds` (number) :- amount of iterations the function will be evaluated
-- `baseline` (number, optional) :- baseline value subtracted from the final result
local function profile(closure, tag, rounds, baseline)
  local s = os.clock()
  for _ = 1, rounds do
    closure()
  end
  local e = os.clock()
  local elapsed = (e - s) - (baseline or 0.0)
  print(string.format("`%s` took `%.3f` seconds", tag, elapsed))
  return elapsed
end
```

For example, we will use if as follows.

```lua
local ROUNDS <const> = 1000000

local a = Vector.new(0, 1)
local b = Vector.new(2, 3)

profile(function() a:add(b) end, "vector:add()", ROUNDS)
```

One may notice that there's an additional cost in handling a closure call. However, with a sufficiently big amount of iterations the overhead will be dampened and won't be that significant. Moreover, as long we profile everything in the same way we are good.

> Most importantly, 'though, we can measure the *baseline* of a `nop()` function. This will let us determine the (almost) exact amount of time spent due to the profiling overhead.

## Let's measure!

Now that we have a convenient way to profile our code, we can apply changes to `Vector` class and implement *natively* some existing methods. We need to mimics in C99 what we are doing in Lua. For example, the `Vector:add()` method is implemented in Lua as follows:

```lua
function Vector:add(v)
  self.x, self.y = self.x + v.x, self.y + v.y
end
```

which corresponds to the following P-code:

```dart
function <src/kernal/tofu/util/vector.lua:141,143> (11 instructions at 0x6187e71872a0)
2 params, 5 slots, 0 upvalues, 2 locals, 2 constants, 0 functions
	1	[142]	GETFIELD 	2 0 0	; "x"
	2	[142]	GETFIELD 	3 1 0	; "x"
	3	[142]	ADD      	2 2 3
	4	[142]	MMBIN    	2 3 6	; __add
	5	[142]	GETFIELD 	3 0 1	; "y"
	6	[142]	GETFIELD 	4 1 1	; "y"
	7	[142]	ADD      	3 3 4
	8	[142]	MMBIN    	3 4 6	; __add
	9	[142]	SETFIELD 	0 1 3	; "y"
	10	[142]	SETFIELD 	0 0 2	; "x"
	11	[143]	RETURN0
```

and is natively implemented like this:

```c
// I'm adopting a naming convention for the native Lua-exposed functions,
// and in general for the implementation, so bear with me if this seems a
// bit too verbose. :)
static int vector_add_native_2oo_0(lua_State *L)
{
    LUAX_SIGNATURE_BEGIN(L)
        LUAX_SIGNATURE_REQUIRED(LUA_TLOBJECT)
        LUAX_SIGNATURE_REQUIRED(LUA_TLOBJECT)
    LUAX_SIGNATURE_END

    lua_getfield(L, 1, "x");
    lua_getfield(L, 1, "y");
    lua_getfield(L, 2, "x");
    lua_getfield(L, 2, "y");

    float x0 = LUAX_NUMBER(L, -4);
    float y0 = LUAX_NUMBER(L, -3);
    float x1 = LUAX_NUMBER(L, -2);
    float y1 = LUAX_NUMBER(L, -1);

    lua_pop(L, 4);

    lua_pushnumber(L, x0 + x1);
    lua_pushnumber(L, y0 + y1);

    lua_setfield(L, 1, "y");
    lua_setfield(L, 1, "x");

    return 0;
}
```

> The `LUAX_` macros above are part of a custom-made Lua-to-C bridge module. It should be intuitively clear enough the sense of them, 'though.

In a similar fashion I wrote native implementations for the `magnitude`, `add`, `dot`, `normalize`, and `angle_to` methods. Profiling the methods over `10000000` rounds the following results have been obtained:

| Method    | Lua (d) | LuaO (d) | LuaC (d) | Lua (r) | LuaO (r) | LuaC (r) | gF/sF | f(x) |
|-----------|---------|----------|----------|---------|----------|----------|-------|------|
| magnitude | 2.918   | 2.697    | 2.814    | 1.170   | 1.142    | 0.670    | 2     | 1    |
| add       | 2.336   | 2.038    | 5.375    | 0.738   | 0.741    | 1.343    | 6     | 0    |
| dot       | 1.863   | 1.792    | 4.340    | 0.621   | 0.631    | 1.043    | 4     | 0    |
| normalize | 5.930   | 5.710    | 4.523    | 1.959   | 1.874    | 1.023    | 2+2   | 1    |
| angle_to  | 3.415   | 3.124    | 4.539    | 1.676   | 1.193    | 1.345    | 4     | 1    |

In the table above, `Lua` refers to the plain Lua `Vector` implementation, and `LuaC` to the one with the methods above implemented in native code. `LuaO` indicates a *slightly optimized Lua implementation* where

* library functions are cached/accessed through local references (e.g. `math.atan()` assigned locally as `local atan = math.atan`), and
* `math.sqrt(x)` have been replaced with `x ^ 0.5` to save a FFI boundary function-call (in favor of the `POWK` P-code instruction).

Within parentheses the `d` indicates the **DEBUG** build (with additional signature checks in the C99 methods), and `r` the **RELEASE** one (with full compiler optimizations are enabled).

In the last two columns `gF/sF` reports the amount of `lua_getfield()` and `lua_setfield()` call required, and `f(x)` the number of external function calls (such as `sqrt()` and `atan2()`) present.

As a final notice, results are stripped of the baseline overhead: this is crucial for a better analysis.

## Can we do better?

Yes, of course! We can implement the whole vector class as a native UDT (and leave to Lua only some minor *embellishment* methods).

The results are astonishing when compared to the (optimized) Lua implementation.

| Method    | LuaO (d) | C99 (d) | LuaO (r) | C99 (r) |
|-----------|----------|---------|----------|---------|
| new       | 20.419   | 4.223   | 7.501    | 2.239   |
| magnitude | 2.697    | 1.081   | 1.142    | 0.168   |
| add       | 2.038    | 1.773   | 0.741    | 0.215   |
| dot       | 1.792    | 1.738   | 0.631    | 0.253   |
| normalize | 5.710    | 2.074   | 1.874    | 0.270   |
| angle_to  | 3.124    | 1.707   | 1.193    | 0.264   |

Even the vector object creation is (way) faster in the full-native C99 implementation, which is coherent with the fact that it vector class is not modelled with a Lua table.

> This reminds us that the creation of an object is expensive (in general), and Lua makes no exception. It is advisable to **recycle and reuse** instances as much as possible when they are no longer needed, rather than always creating new ones. This concept is called [object pooling](https://en.wikipedia.org/wiki/Object_pool_pattern) and is one of the cornerstones in optimizing *realtime* systems (such as a video game).

Basing on the results above, we can draw some final observations:

1. **Compiler optimizations do matter** -- as expected, the release build is significantly faster than the debug one, running on average almost twice as fast.
1. **Optimize you Lua code** -- the Lua code runs marginally faster with the above optimization changes, and they are worth be taken into account.
1. **Plain Lua wins** -- in all cases except one the Lua implementation isn't the worst performing one, both in the debug ang release builds (with only two exceptional cases).
1. **Accessing tables is a bottleneck** -- this is not a surprise, but the major setback when implementing the methods in native C99 code is that the `lua_getfield()` and `lua_setfield()` calls (used to access the vector data, implemented as a Lua table) significantly impacts performances. At the same time, the Lua VM does an *amazing* job in accessing tables, and the plain Lua implementation as no such issue. This is evident by comparing the `add()` and `dot()` methods, with the C99 code significantly slower as the to field access calls grow (way more than the plain Lua code).
1. **Calling functions cost** -- Methods requiring complex mathematical functions have inherent performance penalties, and this is clear by comparing the `dot()` and `magnitude()` methods (identical, minus `sqrt()` call).

## Epilogue

This concludes our brief (but hopefully interesting) journey into the world of optimization as it relates to the implementation of a pseudo-native UDT.

It was an opportunity to look in a little more detail at some aspects of Lua's FFI API, and in particular to understand that some operations can hide unexpected complexities that significantly impact performance. As is often the case, then, the choice of how to model a data type significantly affects the final performance.

Before leaving, however, we can take a chance to answer a final question: << How do I decide whether to keep an algorithm implementation in Lua or convert it to a native implementation? >>

Essentially we can base our decision it in three points:

1. if the data structures are Lua-native, e.g. with extensive use of tables, it is better to stay within the Lua code (since table access via FFI is intrinsically slow). The performance gain offered by selective implementation of native methods is (on average) not significant enough to motivate the additional complexity.
2. if one still wants (or needs) to make a call to a native function, don't pass tables as formal arguments **but** unpack the values as separate arguments for the native call.
3. if performance is an absolute priority, implementing the entire class in native code (or at least the most significant portion of the UDT) is a mandatory requirement.

That's all for now. See ya next. :)