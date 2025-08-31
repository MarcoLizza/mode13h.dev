---
title: The making of a 2D Software Blitter #1
layout: post
permalink: /the-making-of-a-2d-software-blitter-1/
categories:
  - tofu-engine
  - devlog
tags:
  - 2D graphics
  - algorithms
  - technical
---
Some days ago, a fellow **#gamedev** (hi, [Mathew](https://twitter.com/mattymariani)! :)) posed me a interesting question. He was interested in investigating with more detail the raster algorithms, technologies, and tricks that were in use in the '90s... and asked me for a direction on a book on the subject. To my knowledge, such a book doesn't exist. Moreover, most of the information on the subject is very difficult to dig from the Internet. Back in the last '90s and early '00s one could find long precise dissertations on many raster algorithms (they were the backbone of pretty much every *Mode 13h* game of the VGA era). But nowadays? Sadly most of this information has been lost.

One good source of information is the source code for some multimedia libraries (like [SDL](https://www.libsdl.org/) and [SFML](https://www.sfml-dev.org/), just to name two of the most famous), but it does take time (and a good amount of effort) to extract the core logic, as the code is convoluted due to optimizations.

Long story short: since in the last two-years-and-a-half I've been working on [**#tofuengine**](/tofu-engine) (but if you are reading this page probably you already know that) why don't we take the chance to describe some of its internals in a more detailed fashion? So, I took  my notebook out and scribbled a potential outline of the algorithms I'd like to present. Among the many I found that I'd like to describe things like:

* blitting
  * fast-copy
  * clipping
  * sprite-sheets
  * transparency
  * shifting
  * scaling
  * rotations
  * stencils
  * blending
* exotic operations
  * software fragment shaders
  * copperlists emulation
  * mode7
  * pixel-perfect-collision
* serialization
  * loading and palettizing an image
  * transferring to the framebuffer
* optimizations
  * dirty-rects
  * LUTs
  * SIMD-like operations
  * working with RGBA pixels

Without further ado, let's get started!

## Pixel format

First things first: how we are going to represent our images?

In **#tofuengine** every image is [palettized](https://en.wikipedia.org/wiki/Indexed_color). That is, a pixel is stored into a byte, which in turn refers to an entry into a **CLUT** (*C*olor *L*ook-*U*p *T*able -- or **palette** -- holding the actual [RGB](https://en.wikipedia.org/wiki/RGB_color_model) color for every pixel).

> Having used a byte to represent a single pixel, our palette can have **at most** 256 distinct colors. Could we use 16 bits to have larger palettes? Sure... but handling palettes of up to 65536 entries would be unpractical (other than carrying performance drawbacks in the way the CPU cache would be impacted).

This frame-buffer organization variant was common during the '90s and it was colloquially called **chunky organization** (more formally, **packed organization** but this second one is a more broad term), in contrast with the **planar organization**. Each of the two has advantages and disadvantages, and none of the two can be determined and the *best* one... but, honestly, chunky pixels make writing a software-renderer easier! :)

Since a raster image is bi-dimensional are we required to use bi-dimensional arrays?

```c
#include <stddef.h>
#include <stdint.h>

typedef uint8_t gfx_pixel_t; // We'll be using `gfx_` as a common prefix for our library.

int main(int argc, const char *argv[])
{
    const size_t width = 320;
    const size_t height = 240;
    gfx_pixel_t image[height][width]; // Row-major order, array indices are row-first.

    image[17][23] = 42; // Set pixel at `<23, 17>` to `42`.

    return 0;
}
```

We could. But on the long run, when we'll be working with dynamically allocated images, this would turn out being very awkward. We are going to use a uni-dimensional array and a bit of math (since, as a general rule, every N-dimensional array can be flattened and mapped to a 1-dimensional array).

```c
#include <stddef.h>
#include <stdint.h>

typedef uint8_t gfx_pixel_t;

int main(int argc, const char *argv[])
{
    const size_t width = 320;
    const size_t height = 240;
    gfx_pixel_t image[width * height]; // We'll retain row-major order as access.

    image[17 * width + 23] = 42; // Set pixel at `<23, 17>` to `42`.

    return 0;
}
```

## Blitting a Sprite

Or, said in other terms, copying a bidimensional region of an image to another image. **Blitting** (short for **block transfer**) is a somehow *old-skool* term that just means "copying". A **sprite** is a general term that indicates a moving object (or, at least, something that is drawn/glued on a second time, much like a sticker) over a background.

```c
#include <stddef.h>
#include <stdint.h>

typedef uint8_t gfx_pixel_t;

typedef struct gfx_surface_t {
    size_t width, height;
    gfx_pixel_t *pixels;
} gfx_surface_t;

// We expect these two functions to be defined somewhere into some other translation-unit.
extern gfx_surface_t *gfx_surface_create(size_t width, size_t height);
extern void gfx_surface_destroy(gfx_surface_t *image);

void gfx_surface_blit(gfx_surface_t *destination, int x, int y, const gfx_surface_t *source)
{
    gfx_pixel_t *dptr = destination->pixels + y * destination->width + x;
    const gfx_pixel_t *sptr = source->pixels;

    const int dskip = destination->width - source->width;

    for (int height = source->height; height; --height) {
        for (int width = source->width; width; --width) {
            *(dptr++) = *(sptr)++;
        }
        dptr += dskip;
    }
}

int main(int argc, const char *argv[])
{
    gfx_surface_t *background = gfx_surface_create(320, 240);
    gfx_surface_t *sprite = gfx_surface_create(16, 16);

    gfx_surface_blit(background, 23, 17, sprite); // Draw `sprite` on the `background` with upper-left corner at `<23, 17>`.

    gfx_surface_destroy(sprite);
    gfx_surface_destroy(background);

    return 0;
}
```

The logic in the `gfx_surface_blit()` function is simple: seek the initial to-be-written address in the destination buffer, then proceed in row-major order and copy the source data, skipping to the beginning of the next row every time a row is copied (for a long time I used to call the `dskip` amount *modulo* as a nod to Amiga's `BLTxMOD` registers which fulfilled this exact role at a hardware level during [Blitter](https://en.wikipedia.org/wiki/Blitter)'s operations).

We are not doing anything too fancy, for the moment. Also we are not performing any check and/or clipping at all. For that reason, the `source` image needs to be smaller than `destination`, and copying the sprite over the background boundaries would corrupt the result (and possibly present us a segmentation-fault in some cases).

## Next to come

In the next entry we'll see how to better arrange and handle multiple sprites on a *sprite-sheet* (also called *banks*, *maps*, or *atlases*). Then we'll spice things up a bit by adding more advanced features to out **Blitter**.
