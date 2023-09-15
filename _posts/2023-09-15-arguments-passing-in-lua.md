---
title: 'Function Arguments Passing'
author: marco.lizza
layout: post
permalink: "/function-arguments-passing"
comments: true
categories:
  - tofu-engine
  - lua
tags:
  - lua
  - design
---
Picture yourself working at your desk, tinkering on the design of some of the internals of your game engine (as I wrote in the [previous](/tofu-engine-v_0_13_0) devlog post), toward a significant redesign of the game-engine core.

That's a lot of work, and you should try and focus as much as possible and limit yourself to what's strictly related to what's required.

But, alas, you already know from the very beginning that's not going to happen and you'll find some other interesting topic to let your mind wander about...

... and, of course, this time I'm making no exception to this by tinkering about *arguments passing*.

## The Current Way

Since its inception, I invested a significant portion of time in making the game-engine scripting core *efficient*, *elegant* and *clear* to use. One feature I sought to implement it's a reasonably performant *function overloading* as it was something I liked a lot in [Wren](https://wren.io/method-calls.html#signature), which I was using when I was developing **Tofu Engine**. Lua doesn't support overloading straight out-of-the-box (as in C++ or Java, for example) but can be implemented with some minor effort by leveraging the fact that arguments can be optional (they are set to `nil` when missing from the actual arguments list).

Let's see and basic example and write a hypothetical function in Lua that writes some text on screen leveraging another (internal and low-level) function:

```lua
local _default_color = { 255, 255, 255 }

function draw_text(x, y, message, color)
  local actual_color = color or _default_color -- This is the Lua idiomatic for the C ternary operator.
  _draw_text_internal(x, y, message, actual_color)
end
```

We are using the "missing arguments are nil" feature to handle the case when `color` is not provided... but we can achieve the same result using *overloading by arity*

```lua
local _default_color = { 255, 255, 255 }

function draw_text(...)
  local args = { ... }
  if #args == 3 then
    _draw_text_internal(args[1], args[2], args[3], actual_color)
  elseif #args == 4 then
    _draw_text_internal(args[1], args[2], args[3], args[4])
  end
end
```

In this case, we are differentiating the behaviour by checking the number of actual arguments (i.e. the function arity). It might appear to be more "redundant" and less clever as an approach but, in fact, is much more manageable in the long run as the resulting code is less convoluted and easier to read.

There are occasions, however, when we just can't limit ourselves to using the arity to obtain different/specific behaviours, but we need to take into account also the type of the arguments. Let's refer to this as *overloading by type*. An example of this is if we want to write a `vec2:add(...)` method that adds both a vector or a scalar to another vector. Since we have the same amount of arguments in the signature arity is not a sufficient discriminator, so we need to check the type of the actual argument to perform the correct operation:

```lua
function vec2:add(self, vector_or_scalar)
  if type(vector_or_scalar) == 'number' then
    return vec2.new(self.x + vector_or_scalar, self.y + vector_or_scalar)
  else
    return vec2.new(self.x + vector_or_scalar.x, self.y + vector_or_scalar.y)
  end
end
```

We could go even further and combine both approaches (I'll leave that as an example for the reader :-P) making things more and more complex (and hairier).

> I'm not suggesting that function overloading is the suggested way to write code. Quite the contrary, I'm an advocate of "descriptive code". I favour using clear and explicit names for functions/methods, describing explicitly what they do rather than leveraging intuition. For example, I'd rather write separate `vec2:add_scalar()` and `vec2:add_vector()` methods, as they don't require that additional boilerplate "dispatching" code which hinders performances (when we are using interpreted languages, as in compiled languages the overloading is resolved at compile-time) and artificially increases the code complexity. However, there are occasions where a well thought and placed overloaded function/method makes the code *better* (see, for example, the first example).

One thing I'm proud of is the relatively simple but clever and efficient way I implemented overloading-by-arity in **Tofu Engine**, with straight Lua C FFI API. With a sprinkle of ATL-inspired (who remembers it's way to define and implement window-messages handling?) macro usage, the specific sub-function dispatch is called according to the actual number of arguments.

```c
// Overloaded constructor for the `Bank` object. Dispatch is based upon
// only on the number of actual arguments.
//
// See `src/modules/bank.c`.
static int bank_new_v_1o(lua_State *L)
{
    LUAX_OVERLOAD_BEGIN(L)
        LUAX_OVERLOAD_ARITY(1, bank_new_1o_1o)
        LUAX_OVERLOAD_ARITY(2, bank_new_2os_1o)
        LUAX_OVERLOAD_ARITY(3, bank_new_3onn_1o)
    LUAX_OVERLOAD_END
}
```

An extension to this is the implementation of overloading-by-type.

```c
// Overloaded constructor for the `Font` object. We are using a combination of
// the amount of actual arguments and their types. Please not how the type-based
// overloading check is done first, as the same arity-based check would trap it
// erroneously.
//
// See `src/modules/font.c`.
static int font_new_v_1o(lua_State *L)
{
    LUAX_OVERLOAD_BEGIN(L)
        LUAX_OVERLOAD_ARITY(2, font_new_3osS_1o)
        LUAX_OVERLOAD_SIGNATURE(font_new_4onnS_1o, LUA_TOBJECT, LUA_TNUMBER, LUA_TNUMBER)
        LUAX_OVERLOAD_ARITY(3, font_new_3osS_1o)
        LUAX_OVERLOAD_ARITY(4, font_new_4onnS_1o)
    LUAX_OVERLOAD_END
}
```

By having a look on how [overloading is implemented](https://github.com/tofuengine/tofu/blob/6a52389e578c53fc5695e97b9913508b03c27ea4/src/libs/luax.h#L98) one can guess that this second approach is slower than the first. This is correct, as the `LUAX_OVERLOAD_SIGNATURE` check is far more complex than `LUAX_OVERLOAD_ARITY`. Anyhow, when used in seldom-called functions/methods (like, for example, object constructors), this can give some benefits without significantly impacting the overall performance.

## A Third Way?

It seems like we are acceptably satisfied with the results, so far. Why should we search for another way?

Well... because there can be cases where both arity- and type-based overloading are not enough. That's the case, for example, when we have two different overloads of the same function with the same signature.

> Back to the disclaimer I made above, I don't think we should necessarily insist on overloading at this point. Probably a separate, specific, method/function would be preferable. However, we are mostly speculating here... and it's always worth doing it! :)

I'm not particularly fond of Python but I like it's *name arguments* feature. Arguments can be optional, too, and this opens the road to some quality seamless code... for example in Python's `requests` module: when transmitting data we don't have separate `post_json()` and `post_bytes` methods, but a single `post(json=None, bytes=None)` that applied a distinct behaviour according to the actual provided (named) arguments.

Is something like this possible in Lua?

Well... sort of. We *just* need to use **tables as arguments*.

The idea is as simple as the name implies: we move the arguments' whole list and pack it into a table. Then we pass it as the sole argument for the function/method call.

Easy peasy! :)

Let's see a basic example, by rewriting the `draw_text()` function above mentioned

```lua
function draw_text(args)
  _draw_text_internal(args.x, args.y, args.message, args.color or _default_color)
end
```

which we would call as follows

```lua
draw_text({ x = 0, y = 0, message = "Hello, World!", { 0, 255, 0 } })
```

It's definitely not rocket science, and it doesn't seem like a huge improvement, but it's an approach that gives a more clear and more open approach to extendibility. We can add new arguments and use them to discriminate the function behaviour accordingly, even when they are of the same type, just like in Python thanks to the named arguments feature. Also, we don't have to stick to a rigid argument order and -- overall -- the code is more self-documenting. Moreover, unless we did otherwise on purpose, API backward compatibility is easier to implement.

That's an interesting bunch of **silver linings**. Are there any **black clouds**? Yes, of course, and unfortunately they are pretty annoying!

*First and foremost*, the inner implementation of the functions/methods can quickly become intricate. We need to test the size of the table, the presence of some fields, and maybe their type (in a similar fashion as we did in the initial arity/type overloading examples). It's not something that can be easily generalized and needs to be tailored to the case in order not to write dull/unoptimized code.

*Secondly*, tables are to be carefully used in Lua. They are the core of the language, deeply optimized in their usage and made as performant as possible... but they waste resources, nonetheless. When used as arguments they can hinder performance *a lot*. Access times for the table fields tend to be slower (albeit optimizable in plain C code with the FFI API), and creating anonymous tables for each call is expensive (both in space and in time).

For these reasons, they would end in causing bad performances if not properly used.

## The Verdict

As usual, there's no silver bullet. However, we can make some considerations and decide accordingly.

Since we are in the context of a game-engine, our top priority is not wasting resources (mostly CPU but not only) and having as good performance as possible.

I would relegate table-based overloading to less frequently used calls (e.g. class constructors)... but I like consistency while coding, so I would either move *all* my code to this approach or ditch it for good, despite being more versatile.

All being said, even an eight arguments long signature is not that bad when the codebase reaches maturity and you have a well-documented API! ;-)

( note for my future self: please commit yourself to ending the engine API documentation :-P )
