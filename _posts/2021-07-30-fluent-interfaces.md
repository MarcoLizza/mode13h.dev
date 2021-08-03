---
title: 'Fluent interfaces'
author: marco.lizza
layout: post
permalink: "/fluent-interfaces/"
comments: true
categories: 
  - rants
tags: 
  - lua
  - design
  - patterns
  - scripting
---
Over the last decade, more and more object-oriented APIs are adopting the so-called [fluent interface](https://en.wikipedia.org/wiki/Fluent_interface). In particular, it can be found as coupled with the [build pattern](https://en.wikipedia.org/wiki/Builder_pattern). It's not uncommon, then, to write code like this:

```cpp
WindowBuilder *builder = new WindowBuilder();
builder.setWidth(640)
    .setHeight(480)
    .setFullscreen(true)
    .setDoubleBuffered(true)
    .setFrameRate(60);
Window *window = builder.build();
```

This is far more clear than using (possibly overloaded) constructors with many arguments (but using a single argument as structure/class/container for the options is a legit alternative). We won't be discussing this in detail, however.

I'm an advocate of consistent and clear APIs, and a fluent interface surely states the exact purpose of every single method.

All that glitter *isn't* necessarily gold, however. Let's focus for a moment on the signature of one of the methods from the example above:

```cpp
WindowBuilder *WindowBuilder::setWidth(int width);
```

Without an intimate knowledge of its implementation, we cannot be sure of the method's exact behaviour. Is it also creating a new instance (i.e. treating the class as immutable, despite being unlikely)? Probably not, as it's not a `const` method, but that's an "annotation" that only C++ permits. What if we were coding in Java? A method signature is a *contract* and it shouldn't be debatable.

## Enter Lua

Let's open a parenthesis and move to a scripting language, for a moment.

Recently I was implementing the *physic sub-system* for [**#tofuengine**](/tofu-engine), and when creating the `Body` UDT it seemed like a valid case for adopting a fluent interface. A body has many properties (size, shape, type, mass, position, momentum, etc...) and upon creation, we might be interested in setting only a sub-set of them (the others being left to their respective default values). A possible approach is to provide in the constructor the "basic" properties and letting the programmer set the other one later on:

```lua
local body = Body.new("dynamic", 32, 32, 100, 50)
body:position(100, 100)
```

The devil hides in the details, however, and it should be obvious where the problem lies, here. Yep, the constructor is everything but self-explanatory. What do the arguments mean? At which position is the mass specified? Why the body position is left out of the constructor? Having many complex overloaded constructors is unpractical and leaves room for inconsistent code.

Let's simplify the constructor a bit, shall we?

```lua
local body = Body.new("box")
body:size(32, 32)
body:type("dynamic")
body:mass(100)
body:momentum(50)
body:position(100, 100)
```

Now the constructor is used to create a "basic" body, with a single argument which is the sole property that cannot be changed later on (i.e. the body's shape). Everything else is controlled with properties (we'll call them *modifiers*).

## Is redundancy costly?

One major complaint about the code above is that we are required to re-type the `body` symbol on every line. With a fluent interface, through *methods chaining*, would re-restyle the code a follows

```lua
local body = Body.new("box")
  :size(32, 32)
  :type("dynamic")
  :mass(100)
  :momentum(50)
  :position(100, 100)
```

Is it better? Dunno... but I don't like it for sure! :) That's just [pouring some syntactic sugar over the code](https://www.youtube.com/watch?v=0UIB9Y4OFPs) with no relevant benefit whatsoever! The Lua compiler and VM are clever enough to handle the case of repeated method calls over the same object by effectively chaining the calls. That is, the `body` symbol is put on the stack only the first time and then reused in the successive calls. Let's examine this sample code:

```lua
local Counter = {}

Counter.__index = Counter

function Counter.new(value)
  return setmetatable({ value = value or 0 }, Counter)
end

function Counter:inc()
  self.value = self.value + 1
end

function Counter:inc_chained()
  self.value = self.value + 1
  return self
end

local function test()
  local object = Counter.new()
  object:inc()
  object:inc()
  object:inc()
  print(object.value)
end

local function test_chained()
  local object = Counter.new()
  object:inc_chained():inc_chained():inc_chained()
end

test()
test_chained()
```

If we peek at the code generated for the functions `test` and `test_chained` (using the `luac -l` command) we get the same code, that is

```txt
	1	[14]	GETTABUP 	0 0 -1	; Counter "new"
	2	[14]	CALL     	0 1 2
	3	[15]	SELF     	1 0 -2	; "inc"
	4	[15]	CALL     	1 2 1
	5	[16]	SELF     	1 0 -2	; "inc"
	6	[16]	CALL     	1 2 1
	7	[17]	SELF     	1 0 -2	; "inc"
	8	[17]	CALL     	1 2 1
	9	[18]	RETURN   	0 1
```

There's no additional benefit for the VM in chaining the calls. Moreover, the `inc_chained()` carries the additional cost of returning the `self` reference (and wastes a VM stack slot for storing it).

## Is fluency worth using?

It depends.

From a purely conceptual standpoint, I don't like that fluent-interfaces try and solve and solve a *syntax* issue with a design pattern (unlike Pascal and that solved this at a grammar level back in the '70s with the `With` statement). This adds unnecessary complexity and overhead. It might appear as "natural", but there's nothing natural in trying and "verbalize" a procedural-like language. It's just awkward, especially when only a subset of the language's data-type system follows this pattern.

Nonetheless, when designing an*immutable* data type (e.g. a `String` class) it may come in handy. Unfortunately, implementing *immutable* UDTs in Lua is just expensive (due to the continuous creation of new tables to store the objects' instances that thrash the stack).

In conclusion, [I agree with Guido Van Rossum](https://mail.python.org/pipermail/python-dev/2003-October/038855.html) and I'm going to avoid using them in **#tofuengine**'s API.