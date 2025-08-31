---
title: Faster Inverse Square Root
layout: post
permalink: /faster-inverse-square-root/
categories: 
  - rants
tags: 
  - lua
  - math
  - optimization
  - scripting
---
On August 19th, 2021 [Quake's remastered edition](https://en.wikipedia.org/wiki/Quake_(video_game)#Remastered_edition) has been released worldwide.

I'm old enough to remember the day the original version was published. Or better that day, back in the early summer of 1996, when I was informed by a friend of mine (we both were CS students at the university) to go and buy that month *The Games Machine* issue, as in the attached CD (they were very uninspiringly called *Silver Disk*) there was the playable demo of "a game that it's way better than DooM".

I did play DooM over the previous year. Yep, I know, I was a bit late at the party... but, as a partial excuse, I have been a **loyal** Amiga owner and lover (and I still love it, a lot) until 1995, when I bought my first PC (running Windows 95... that's when I learned how to reinstall an OS from scratch the hard way). Anyway, much for this digression. I did like DooM, mostly for its technical details and less for the gameplay (FPS has never been my favourite type of game, and still isn't). I was so enamored with DooM's internals! My first encounter with binary-space-partitioning! And how many WAD-like archive formats I coded[^1].

With that premise, I had to try that new game, so I flew to the newsstand to grab a copy of the magazine. 

Once again, the gameplay and aesthetics were not really my thing, but the game was undoubtedly (as its predecessors) a ground-breaking masterpiece.

But, for most of the gamedevs and indiedevs, Quake is famous also for incorporating a [very clever optimization](https://en.wikipedia.org/wiki/Fast_inverse_square_root) trick to calculate the **inverse square root** of a floating-point number. Back then, floating-point operations were computationally demanding for the CPUs and it was common to exploit any possible trick to avoid their usage (e.g. by using [fixed-point arithmetic](https://en.wikipedia.org/wiki/Fixed-point_arithmetic)) and use only integer numbers.

Having passed 25 years it may be natural to [investigate and check if that trick is still relevant](https://www.linkedin.com/pulse/fast-inverse-square-root-still-armin-kassemi-langroodi/). As we should suspect, the algorithm is more a beautiful piece of antique than of practical application. Current compilers can optimize the operation to a single assembly instruction for both [x86](https://www.felixcloutier.com/x86/rsqrtss) and [ARM](https://developer.arm.com/documentation/100076/0100/a64-instruction-set-reference/a64-simd-scalar-instructions/frsqrte--scalar-) CPUs (and that two account to pretty much all modern platforms).

But, how much slower can this algorithm be? Does this apply also to Lua? We can take this chance to analyze Lua's VM behaviour and define some rational guidelines for the design of future **#tofuengine**'s API.

## The comparison

There are many ways to calculate the inverse square root of a number, in plain Lua 5.3+ code or with a combination of Lua code and FFI exposed C functions. Additionally, we are going to profile Lua-to-Lua and Lua-to-C function calls, to evaluate if any difference arises between these two.

| shortname | code                 |
| :---      | :---                 |
| plain-v0  | `(x ^ 0.5) ^ -1.0`   |
| plain-v1  | `1.0 / math.sqrt(x)` |
| plain-v2  | `x ^ -0.5`           |
| api-v0    | `Math.invsqrt(x)`    |
| api-v1    | `Math.finvsqrt(x)`   |
| api-v2    | `Math.qinvsqrt(x)`   |
| api-v3    | `qinvsqrt(x)`        |
| luanull   | `luanull(x)`         |
| cnull     | `cnull(x)`           |

Where the exported C APIs have been defined a follow:

```c
int invsqrt(lua_State *L)
{
    lua_pushnumber(L, 1.0f / sqrtf(luaL_checknumber(L, 1)));
    return 1;
}

static inline float _Q_rsqrt(float number)
{
    const float x2 = number * 0.5f;
    float y = number;
#pragma GCC diagnostic push
#pragma GCC diagnostic ignored "-Wstrict-aliasing"
    int32_t i = *(int32_t *)&y;             // evil floating point bit level hacking
    i = 0x5f3759df - (i >> 1);              // what the fuck?
    y = *(float *)&i;
#pragma GCC diagnostic pop
    y = y * (1.5f - (x2 * y * y));          // 1st iteration
//	y = y * (threehalfs - (x2 * y * y));    // 2nd iteration, this can be removed
    return y;
}

int finvsqrt(lua_State *L)
{
    lua_pushnumber(L, _Q_rsqrt(luaL_checknumber(L, 1)));
    return 1;
}

int l_cnull(lua_State *L)
{
    return 0;
}
```

 and the Lua functions are the following:

```lua
function Math.qinvsqrt(x) -- This function is declared inside the `Math` module of #tofuengine.
  return (x ^ -0.5)
end

local function qinvsqrt(x) -- This function is defined locally in the scope of the benchmark routines.
  return (x ^ -0.5)
end

local function luanull()
end
```

## The results

Each variation has been profiled over `100000000` iterations, both in *debug* and *release* (with full `-O3` optimization enabled) mode. The results are summarized in the following table:

| shortname | code                 | debug  | release |
| :---      | :---                 |   ---: |    ---: |
| plain-v0  | `(x ^ 0.5) ^ -1.0`   |  4.695 |   2.774 |
| plain-v1  | `1.0 / math.sqrt(x)` |  9.855 |   3.286 |
| plain-v2  | `x ^ -0.5`           |  2.694 |   1.225 | 
| api-v0    | `Math.invsqrt(x)`    |  9.273 |   2.307 |
| api-v1    | `Math.finvsqrt(x)`   |  9.696 |   2.380 |
| api-v2    | `Math.qinvsqrt(x)`   |  8.302 |   3.649 |
| api-v3    | `qinvsqrt(x)`        |  6.835 |   2.973 |
| luanull   | `luanull(x)`         |  3.437 |   1.432 |
| cnull     | `cnull(x)`           |  3.741 |   1.589 |

We can observe that:

* the overhead difference between Lua-to-Lua (`luanull`) and a Lua-to-C (`cnull`) calls is negligible, with the former being a tad faster;
* calling a function exposed "inside" a module namespace (e.g. `Math.invsqrt()`) can amount to a significant overhead;
* the **fast** inverse square-root is as fast as a simple `1 / sqrt()` and, given that it's just an approximation, it's not worth using on modern CPUs;
* the "combined" `invsqrt()` function is faster than `1.0 / math.sqrt()`, as the latter sums up the cost of an FFI call and the division;
* the fastest approach is to use the `^` operator to compute the inverse square-root in plain Lua.

## What we learn from this

Basing on the results above, we can confirm some well-known Lua optimizations[^2], that is:

* favour native Lua code and operators over library functions, when possible (FFI overhead will be avoided);
* **localize** Lua functions, i.e. store and reuse them as references into a local variable, especially in case of namespace/module (table field access wil be avoided);
* combine "complex" (i.e. non-trivial) operations into a single FFI accessible function;
* function calls (either Lua-to-Lua or Lua-to-C) do cost -- avoid them for time-critical code.

As for the inverse square root computation, the compiler and CPU make a hell of a good job in optimizing such a complex operation. In modern architectures is far better to skip the "fast" approximation and use the straight computation.

We'll just keep it in the engine's API an [Easter egg](https://en.wikipedia.org/wiki/Easter_egg_(media)), then! :)

---

[^1]: This proved to be an invaluable exercise, as in my professional career I ended adopting many of that concepts several times. Even while designing and coding **#tofuengine**'s packed format I reused some of the ideas I elaborated ~25 years earlier.

[^2]: see [this](http://lua-users.org/wiki/OptimisationCodingTips) and [this](https://springrts.com/wiki/Lua_Performance).
