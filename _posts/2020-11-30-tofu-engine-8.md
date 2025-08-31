---
title: tofu-engine #8
layout: post
permalink: /tofu-engine-8/
categories:
  - tofu-engine
  - devlog
tags:
  - opengl
  - software-renderer
  - roto-scaling
  - blitter
---
Last month's development recap. Not an over-productive month, but some interesting things have been achieved.

Configuration-file support is something that I reworked numerous times. I began with a JSON file, but I didn't like to use a third-party library just for parsing a simple configuration file. So, I migrated to a Lua script... which made sense since Lua is the scripting language I'm using for the engine (courios that Sol, from which Lua descends, was created many years ago as a dynamic configuration language), but it over-complicated the engine code and gave no real benefit over non-interpreted configuration files. So, I settled on a basic and plain key-value format for which a custom parsed can be coded in a few lines. Over the last weeks, I reworked the file format once again, approaching something more similar to [`.ini` files](https://en.wikipedia.org/wiki/INI_file). This is a format that, despite being brain-dead simple, I always like to use because *it just makes things done*. No-fuss. Also, it's interesting the fact that "regions" (called "contexts" in `#tofuengine`) are used to group related parameters. This goes along with a recent refactoring, where I refactored the code to use *nameless structs* as a way to (again) group related parameters and make the code clearer.

( I case you are asking, using nested nameless `struct`s isn't going to slow down execution times since the compiler is clever enough to optimize this kind of access as a single -- cumulative -- offsetting from the outer `struct` base address )

The engine API has been partially reworked, by making the `Bank.blit()` method overloading a tad clearer. Also, the `canvas` formal argument for `Canvas`, `Bank`, `Font`, and `XForm` object constructors is now *mandatory* (but it can also be changed dynamically). A `Canvas.copy()` method has been added (which copies a canvas region, ehm...), and the already existing `Canvas.process()` method has been generalized: a function is passed as the first actual argument, where the final pixel is computed. This feature can be seen as a somewhat "cheaper" variant of a (Lua-scripted) fragment-shader.

```lua
function Game:render(ratio)
  local canvas = Canvas:default()
  local width, height = canvas:size()

  canvas:clear()

  -- Your canvas drawing code, here...

  canvas:process(function(x, y, current, new)
      -- You can calculate the resulting pixel at `< x, y >` as a function
      -- of its `current` and `new` value....
      return new -- ... or return the `new` value to perform a straight, but un-optimized, copy.
    end,
    0, 0, -- Target position.
    0, 0, width, height) -- Source rectangle.
end
```

( an interesting generalization would be to attach a similar "pseudo-shader" to a canvas and use it to filter every drawing operation )

As a minor optimization, an [OpenGL's vertex array object](https://www.khronos.org/opengl/wiki/Vertex_Specification) is used to perform the (one and only) texture draw operation, i.e. the one that moves the visual data from an offscreen texture to the frame-buffer. While this change doesn't seem to boost performances significantly, it makes nonetheless the code cleaner.

At last the most time-consuming activity for this month worth of coding.

I (finally) fixed the roto-scaling algorithm for good. This has been bugged since the very beginning, as with a proper combination of the actual arguments, a pixel row and/or column would be missing due in the final result to rounding errors. Despite having pretty clear the origin of the issue, I wasn't able to fix the *optimized* algorithm... so I had to opt for a different route.

For the sake of my mental health (and since this isn't going to be the most significant part of the game engine), I went for a simpler/naive technique... but that handles rounding errors better! Yep, you guessed right: I'm (inverse) rotating by applying the rotation formula to each pixel individually.

The starting point was to compute the destination roto-scaled area. Instead of rotating the four (unscaled) sprite's corners, I consider that, given the rotation anchor point, the sprite always resides inside a disc. The radius of the disc is equal to the distance between the anchor point and the farthest corner of the sprite rectangle, i.e. the magnitude of a vector with

* the biggest horizontal distance between the anchor point and the rectangle left/right margin (as width), and
* the biggest vertical distance between the anchor point and rectangle top/bottom margin (as height).

If the anchor point is set to a corner, for example, the radius is the full diagonal length of the sprite rectangle.

Then, we just iterate across the disc's axis-aligned bounding-box (and we inverse roto-scale the pixels). As an optimization, we calculate the distance between each pixel and the disc's center, to discard the pixels that lay outside the disc and save a useless operation (since roto-scaled pixels are tested for inclusion within the source texture and they would end being discarded eventually).

A nice feature of this approach is that horizontal and vertical flipping can be optimized because can be *"fused"* in the roto-scaling matrix (and no additional branches for offsets calculations are required in the inner loop).

Of course, crucial for this algorithm to properly work is considering the coordinates to be *"sub-pixel"* in nature, that is every point is placed at **half-pixel (mid-center) position**.

Speaking of which, a similar issue was present (albeit in a less prominent way) also in the scaling algorithm. We need to correct the calculations by considering the mid-center position of each pixel and use this formula for proper scaling coordinates:

```lua
x_s = round((x_r + 0.5) / S_x - 0.5) = floor((x_r + 0.5) / S_x)
y_s = round((y_r + 0.5) / S_y - 0.5) = floor((y_r + 0.5) / S_y)
```

In practical terms, we can rewrite the formula in a recurring fashion if we proceed by moving inside the resulting rectangle, and we increment by `1 / S_x` and `1 / S_y` steps.

These aren't the best roto-scaling and scaling algorithms by any stretch, but for the moment being, I'm happy with the result. Perhaps I'll go back and rewrite/optimize/fix this part of code once more, in the future... but that's more than enough for a small 2D game engine. If I were to use prominently rotated and scaled sprites, I'll write a different rendering system that fully exploits the GPU rather than being entirely software-based.
