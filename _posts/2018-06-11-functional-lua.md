---
title: 'Functional Lua'
author: marco.lizza
layout: post
permalink: "/functional-lua/"
comments: true
categories: 
  - snippets
tags: 
  - languages
  - lua
  - scripting
---

I'm constantly struggling between situation-tailored code (as we did in the 8/16 bit days, with some minor exceptions), or reusable and generic code.

Generally, I'm in favour or the latter, but sometimes it feels like falling into the rabbit hole. I don't seem to be capable of setting a limit for myself, ending in over-generalizing almost every routine.

The extreme of this behaviour is represented by functional-oriented programming (meaning the usage of functional-like approach to algorithms). I really like the elegance and compactness of the resulting code, albeit fearing I'm wasting precious CPU cycles and memory resources for the sake of elegance (in a field like **#gamedev** where we should strive for performances).

However, I ended in writing my tiny-little *map/filter/reduce* library for *Lua*.

```lua
local MapFilterReduce = {
  map = function(input, callback) -- mapper(value, index, table)
    local output = {}
    for index, value in ipairs(input) do
      table.insert(output, callback(value, index, input))
    end
    return output
  end,
  filter = function(input, callback) -- filter(value, index, table)
    local output = {}
    for index, value in ipairs(input) do
      if callback(value, index, input) then
        table.insert(output, value)
      end
    end
    return output
  end,
  reduce = function(input, callback, initial_value) -- reducer(accumulator, value, index, table)
    local accumulator = initial_value
    for index, value in ipairs(input) do
      if accumulator == nil then
        accumulator = value
      else
        accumulator = callback(accumulator, value, index, input)
      end
    end
    return accumulator
  end
}
```

Despite being simple to implement, it's a nice opportunity for some Lua-specific optimizations. In particular, the performance of the algorithm is affected by _how the table is iterated_ and _how the values are stored in the resulting table_.

Since out tables are not dictionaries/hashmaps, but more like vectors/arrays, they can be iterated in three distinct ways:

* `for k, v in pairs(t) do ... end`,
* `for i, v in ipairs(t) do ... end`, and
* `for i = 1, #t do ... end`.

Letting the analysis out as an exercise for the reader :), it appears that the fastest method is the last one. So, as long as your table uses only integer sequential indexes it's suggested to use that strategy.

Once the mapper function is called, the resulting value `v` can be either stored in the resulting table `r`:

* by pushing it and the end of the table (`r[#r + 1] = v`),
* some again but with the auxiliary function (`table.insert(r, v)`), or
* by directly writing the indexed value (`r[i] = v`).

Lua programmers are typically accustomed to the first method since they have been taught it's faster than the second (which is true only in minor part). The real optimization resides in the **third** method, which is faster due to to the fact it skips the table length calculation in each loop.

> The fastest method would be reusing the table itself (that is, mapping the values *in place* in the table). We are not considering this method, since we want the original table to be preserved, however.

The resulting implementation is the following.

```lua
local MapFilterReduce = {
  map = function(input, callback) -- mapper(value, index, table)
    local output = {}
    for index = 1, #input do
      local value = input[index]
      output[index] = callback(value, index, input)
    end
    return output
  end,
  filter = function(input, callback) -- filter(value, index, table)
    local output = {}
    local length = 0
    for index = 1, #input do
      local value = input[index]
      if callback(value, index, input) then
        length = length + 1
        output[length] = value
      end
    end
    return output
  end,
  reduce = function(input, callback, initial_value) -- reducer(accumulator, value, index, table)
    local accumulator = initial_value
    for index = 1, #input do
      local value = input[index]
      if accumulator == nil then
        accumulator = value
      else
        accumulator = callback(accumulator, value, index, input)
      end
    end
    return accumulator
  end
}
```

While this can be seen as an instance of *premature optimization*, it's benefits are concrete (an x8 speed increase for the `map` and `filter` meta-functions) and the final code is not significantly less elegant than the former.

Will I be using it in my **#gamedev** endeavours? Let's see...
