---
title: 'tofu-engine #1'
author: marco.lizza
layout: post
permalink: "/tofu-engine-1/"
comments: true
categories: 
  - tofu-engine
  - devlog
tags: 
  - scripting
  - benchmark
  - opengl
---
Happy 1st day of Spring to everybody! :D

In the last month, I've completed what can safely be called *the first milestone*: `v0.1.0`. Well, to be honest, due to the fact it took me quite a while to write this post, I'm also reaching `v0.2.0` in the meanwhile. :P

From the initial the idea of creating a *lo-fi* and *easy-to-use* 2D game-engine I reached the point of having some (scripted) sprites bouncing on the screen. Also, drawing primitives have been implemented alongside with some (simple) text drawing.

To be honest, once I chose the media library and the scripting language of preference ([raylib](https://www.raylib.com/) and [Wren](https://github.com/wren-lang/wren), respectively) it has been just a matter of organizing some code. Really straightforward, although there's still need for better abstraction and modularization... but this will come over time.

In the present post, I want to highlight some of the peculiar aspects I've faced. In the future I'll try and write *#devlog* post more frequently (perhaps in a more succinct and basic form, more like development notes).

## Embedding API

**Wren**'s embedding API is really easy and seamless to use. I chose not to use and call *static* methods from the engine to the script. Rather, the game need include a `tofu.wren` script and an entry point, exposing a `Tofu` class as such.

```Dart
class Tofu {

    construct new() {
        // Initialize the game here. :)
    }

    input() {
        // Called at most once on each redraw loop, after the user input state has
        // been processed and is ready to be used.
    }

    update(deltaTime) {
        // Called several times if we are behind the requested loop frequency.
    }

    render(ratio) {
        // Called exactly once for each loop.
    }

}
```

I'm still debated whether the `input()` method makes real sense, whether the *input-processing phase* should be performed inside each `update()` call, or if we should call the former each time the latter is called. There's way too much room for choices!

Also, I'm evaluating a possible significant change in the above-mentioned API. Rather than implicitly calling pre-defined and hardcoded methods, we could use explicit *event registering* for the callback methods. It would be more versatile but would require a bit of boilerplate code that is pretty much unnecessary.

Back to the scripting API.

In order to load and initialized the main `Tofu` class instance, in Wren we just need to interpret this nice piece of bootstrap code:

```dart
import "./tofu" for Tofu
var tofu = Tofu.new()
```

Later, in the engine code, we just need to fetch the handle to the `tofu` variable and we are ready to go! No static/global classes are required! Easy-peasy!

As a side note, in the bootstrap code above we can spot how modules are accessed. That is when starting with the `./` prefix, they are treated as user-provided script files and searched relative to the main script file position (otherwise they are considered as built-in modules).

## Palette support

One feature I really loved back in the Amiga/DOS days is the use of palettes. Perhaps it's just a matter of habit and nostalgia, but there were a number of special effects that were achieved with ease (e.g. color cycling, color remapping, (cross)fading, fire, blobs, shade-BOBs, etc...) that I miss very much.

For this reason, it has been mandatory for me to support palettized rendering natively in my engine.

This can be easily done with a custom fragment-shader:

* the drawing primitives use the `red` component to select the palette index for the pixel (we can use any of the color components and for more colors, we could combine two components);
* at some point we send to the fragment-shader the content for a uniform array of `vec4`s (the palette);
* the fragment-shader accesses the palette and re-calculate the real pixel color.

This fits nicely in `#tofuengine` since we work on an offscreen rending texture during the drawing phase and transfer the whole texture to the display with a single `BitBlit()` operation (stretching the texture to match the final on-screen size). So the palette fragment-shader needs only to be used on this final "flipping" with reduced overhead.

One thing to notice is that OpenGL poses a limit to the maximum amount of elements in a uniform array (with the constant `GL_MAX_FRAGMENT_UNIFORM_VECTORS`). The value depends on the platform in use and it seems that a lower-bound is represented by Raspberry-PI with `136` elements. While it would have been nice to set `256` as a limit (like VGA's *Mode 13h* :D), realistically a palette of 64 colors is more than enough for the scope of the engine. As an example, the NES palette has been included as a pre-loaded reference palette.

> Another approach for palette support can be achieved by using a **texture buffer object**, that is a unidimensional texture carrying shared data from the application to the shader. However, at the moment *raylib* doesn's support it properly... perhaps I'll get back to this issue in the future and check if it might be something of interest for optimization.

In order for the palettized rendering to work, textures/images need to be "patched" runtime. Each pixel is replaced with an "indexing color" by searching the nearest-matching color in the current active palette. At the moment this requires (due to the library features) to allocate an additional temporary support buffer. It would be nice to have it work through a callback during the loading itself so that an additional (re)scan and support buffer won't be needed.

## Sample applications

The befits of writing tiny applications during the development are evident:

* they serve as examples/reference for the future,
* they can expose issue in the engine.

The first application I wrote is a **BunnyMark** re-implementation. While this is not a reliable benchmark for comparing the performances with other engines, it can be used to do some regression checks. Also, it is very cute. :)

The second application is more in the league of the *lo-fi* feel and is the **Doom Fire PSX** effect (that *Fabien Sanglard* described recently in his [blog](http://fabiensanglard.net/doom_fire_psx/). It is a very simple effect but served well to spot some issue with the pixel-perfect rendering with OpenGL primitives. This is due to OpenGL's *rasterization rules* and a little [fix](http://glprogramming.com/red/appendixg.html#name1) is required in order for it to properly work. Moreover, filled and wireframe polygons behave differently with regard to pixel positioning and size. Anyway, the application was easy to implement but posed also an interesting question: should a primitive `Grid` type be implemented to better handle 2D arrays? If yes, should it implemented in native (C) or interpreted (Wren) language?

## What's next?

There's a lot to do. :)

The next relevant thing to be implemented is **tiled mapping support**. It should support camera placement and scrolling, as well, and this is something I'd like to handle by using shaders (which is something that has a nice continuity the way scrolling was implemented in some 8- and 16-bit computers of the '80s/'90s, i.e. through very specific chipset registers tweaking).

I will also start to think about **collision detection**. Is this something that depends and changes on a per-game basis, or could be worth adding to the engine natively?
