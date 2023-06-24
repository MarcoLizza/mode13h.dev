---
title: 'Silver bullets'
author: marco.lizza
layout: post
permalink: "/silver-bullets/"
comments: true
categories: 
  - snippets
tags: 
  - lua
  - algorithms
  - sorting
---
There is no silver bullet. Never.

In the context of **#gamedev** this could mean that there's not a single [sorting algorithm](https://en.wikipedia.org/wiki/Sorting_algorithm) that can suit every case. Well, to be honest, this is something that anyone with a basic *Computer Science* education of some sort should know. Yep, I'm saying that all those hours spent learning all the various sorting algorithms in [that Robert Sedgewick's book](https://algs4.cs.princeton.edu/home/) will eventually turn useful, sooner or later. :)

Let's take, for example, a little demo I was writing some days ago with **#tofuengine**. Nothing too fancy, just a small *old-skool* demo/intro to testing some engine's features. At one point, there is a bunch of stars rotating and falling from above, divided into five different depth levels. The star objects are maintained in a random-generated list, and once a start reaches the bottom of the screen a new one is created with random velocity/position/depth.

Just as any just-above-noob-level **gamedev** knows, to properly render something like this, the [painter's algorithm](https://en.wikipedia.org/wiki/Painter%27s_algorithm) is to be applied... and the most straight-forward (naive?) way to accomplish it keep the list sorted, by (re)sorting it each time an object is added (but not removed, as removal preserves ordering). In [Lua](), which is the scripting language used by the game-engine, this can be accomplished by calling the `table.sort()` function.

```lua
local function spawn(objects, name)
  local object = {
      -- Fill object's attributes...
      name = name,
      depth = math.random(1, 5)
    }
  table.insert(object, object)
  table.sort(objects, function(a, b) return a.depth < b.depth end) -- In real-world cases, one shouldn't use an unnamed function.
end

local function render(objects)
  for _, object in ipairs(objects) do
    draw(object)
  end
end
```

Presto!

Well, not really.

By doing this we'll soon be noticing that something isn't going as we expect. Occasionally, objects at the same depth level appear as "fighting" for the drawing precedence. No, that's not due to some sort of depth-resolution (as in [Z-fighting](https://en.wikipedia.org/wiki/Z-fighting)) but because Lua's sorting routine **is not [stable](https://en.wikipedia.org/wiki/Sorting_algorithm#Stability)** and, when (re)sorting the list of objects it happens that already sorted items are "moved" in the list. Run the following code, as an example of this behaviour.

```lua
function dump(t)
  local o = {}
  for _, v in ipairs(t) do
    table.insert(o, string.format("[%s %s]", v.name, v.depth))
  end
  print(table.concat(o, " "))
end

local T = {
  { name = 'a', depth = 0 },
  { name = 'b', depth = 1 },
  { name = 'c', depth = 2 },
  { name = 'd', depth = 0 },
  { name = 'e', depth = 1 },
  { name = 'f', depth = 2 },
  { name = 'g', depth = 0 },
  { name = 'h', depth = 1 },
  { name = 'i', depth = 2 }
}

print("> unsorted")
dump(T)

table.sort(T, function(a, b) return a.depth < b.depth end)
print("> sorted")
dump(T)

table.sort(T, function(a, b) return a.depth < b.depth end)
print("> re-sorted")
dump(T)
```

You'll get this output. It's evident that in the first `sorted` result the relative ordering of the items is not preserved. This is not necessarily an issue per-se since once depth-clustered the objects would be drawn properly. The issue appears when we add and re-sort the list, which is evident in the second `re-sorted` result, where some items are re-positioned in the list.

```txt
> unsorted
[a 0] [b 1] [c 2] [d 0] [e 1] [f 2] [g 0] [h 1] [i 2]
> sorted
[a 0] [g 0] [d 0] [h 1] [e 1] [b 1] [c 2] [f 2] [i 2]
> re-sorted
[a 0] [d 0] [g 0] [b 1] [e 1] [h 1] [f 2] [c 2] [i 2]
```

In Lua, the [`table.sort()`](https://www.lua.org/source/5.2/ltablib.c.html#auxsort) function implements Sedgewick's [quicksort](https://en.wikipedia.org/wiki/Quicksort) algorithm. The behaviour is intrinsically "by-design" and unpredictable, but will eventually occur. Don't get me wrong. Hoare's *quicksort* is an amazing algorithm and it's somewhat a "jack of all trades"... but it simply can't be the *best* choice in every context. In this perspective, the best choice is the one made with Python's [Timsort](https://en.wikipedia.org/wiki/Timsort).

> Another personal favourite of mine is [Merge sort](https://en.wikipedia.org/wiki/Merge_sort), which is even "older" than quicksort. It's stable, highly parallelizable, and fast. Consider that it was conceived in an age when data was stored on magnetic tapes which were not random-accessible and very slow. How cunning to find an algorithm efficient for them?

Cool. Back to the initial issue, how do we solve it? Well, we have at least two viable options.

In the first one, we can keep on using Lua's original sorting and make it work! :) This can be achieved by adding another property to our objects, in the form of a strictly monotonous (i.e. always increasing) identifier.

```lua
local _id = 0

local function next_id()
  local id = _id
  _id = _id + 1
  return id
end

local function spawn(objects, name)
  local object = {
      -- Fill object's attributes...
      id = next_id(),
      name = name,
      depth = math.random(1, 5)
    }
  table.insert(object, object)
  table.sort(objects, function(a, b) return a.depth < b.depth or (a.depth == b.depth and a.id < b.id) end)
end
```

With a bit of additional boilerplate code, we are tracking the object with an identifier (but we could have used a time-stamp with enough resolution, probably) and exploiting it to sort equally depth objects (older objects come first). This is a legitimate solution, but unless one is already tracking the objects (for example, due to the presence of an [entity-component system](https://en.wikipedia.org/wiki/Entity_component_system)), the additional complexity is not worth the effort. Also, as the comparator is more complex, the general actual cost for each call will increase.

We can do better by making some considerations on the problem we are solving itself: we are sorting a list of `n` objects where `n - 1` of them is always sorted, except for the newest one (which is almost certainly out-of-place). The **optimal** approach, then, is just to scan the list searching for the first object *greater-or-equal* than the new one and insert it there.

```lua
local function add(table, item, comparator)
  local lower_than = comparator or _lower_than
  for index, other in ipairs(table) do
    if lower_than(item, other) then
      table.insert(table, index, item)
      return
    end
  end
  table.insert(table, item)
end

local function spawn(objects, name)
  local object = {
      -- Fill object's attributes...
      name = name,
      depth = math.random(1, 5)
    }
  add(object, object, function(a, b) return a.depth < b.depth end)
end
```

In fact, this is what [insertion sort](https://en.wikipedia.org/wiki/Insertion_sort) does. Behind its apparent simplicity, it provides quite a bit of advantage and it's perfectly fine for smaller datasets. Also, it's [adaptive](https://en.wikipedia.org/wiki/Adaptive_sort), and in the case of an almost ordered list like in ours, it behaves (almost) exactly as the optimal algorithm above.

```lua
-- Naive implementation of insertion-sort. Cormen-Leiserson-Rivest's optimized version doesn't fit well
-- with Lua's `for ... do end` idiom.
local function insertion_sort(table, lower_than)
  local length = #table
  for i = 2, length do
    for j = i, 2, -1 do
      if not lower_than(table[j], table[j - 1]) then -- Preserve stability! Swap only if strictly lower-than!
        break
      end
      table[j - 1], table[j] = table[j], table[j - 1] -- Swap adjacent slots.
    end
  end
end
```
