---
title: 'tofu-engine #9'
author: marco.lizza
layout: post
permalink: "/tofu-engine-9"
comments: true
categories:
  - tofu-engine
  - devlog
tags:
  - milestone
  - physics
  - webassembly
---
Ready for milestone **v0.10.0**!

Lot of things have changed in the last nine months. I'm still unsure whether the API can be considered *almost-final-for-the-moment*, but we are reaching some consistency. Also, a lot of new features and changes have been done. Here's a brief list, divided into macro-areas.

## Graphics

Let's start with what is probably (well, at least to me) the most interesting feature... that is **display programs**. This is heavily inspired to Amiga's (Copper)[https://en.wikipedia.org/wiki/Original_Chip_Set#Copper] and to any dinosaur game-developer out there should ring a bell. For those who are new to this, the Copper is a co-processor (hence is name), part of the Amiga's chipset and included in the (https://en.wikipedia.org/wiki/MOS_Technology_Agnus)[https://en.wikipedia.org/wiki/MOS_Technology_Agnus] integrated circuit, which runs in parallel with the CPU and whose finite-state machine is programmed through the so called *copper-lists*. Those are a list of instruction that, synchronized with the raster beam, can be used to alter most of Amiga's chipset registers. A lot of (cool effects)[http://vikke.net/index.php?id=copperbars-1] can be achieved... but, of course, among the many only a strict subset of "register" have been made available. Differently from Amiga's Copper, the changes are not permanent but the modifications to the "registers" are reset on each new frame, and starts from default "neutral" values (e.g. the palette colors always starts with the current display palette). In *#tpfuengine*, a **display-program** is what defines how the framebuffer is transferred to the display.

On a similar topic, the SNES's inspired **Mode-7** feature have been reworked in some of its parts. Nothing too fancy, I've just remove the *transparency* in the drawing operations and in case the image's size a *power-of-two* the **REAPEAT** clamping mode is now faster. Speaking of which, **mirrored** variants of the clamping modes have been added.

Post-effects shaders have been supported since the very first versions, but they required the programmer to provide his/her own fragment-shader. Also, the shaders could change run-time. This is no longer supported and the shader to be used is specified into the `tofu-config` file (with the `display.effect` parameter). Also, some shaders have been bundled in the engine to perform LCD, scanline, and color-blindness emulation.

Minor refactoring also for the `Font` class. Previously when a fixed subset of the ASCII alphabet was mandatory to be used, but that was a bit to of hassle if one (for example) just wanted to create a bitmap-font to display digits. That way it now possible to specify the font alphabet when creating the object.

Wh

  - stencils, blend modes

When drawing tiled environments (i.e. pretty much every background graphics in a retro-inspired game) the drawing process can ben optimized, since it unrealistic to think that a map-tile is going to be rotated and/or scaled at non integer values. This resulted in additional optimized/simplified "tiled" drawing functions that supports horizontal/vertical flipping, integer scaling, and horizontal/vertical "shifting" (that is, the tile content can be circular-shifted).

* **GIF recording** and **screen-capture**: 
Occasionally it's very useful capture snapshots and to record videos. Mostly for promotional/sharing purposes. Cool, let's add it! Unfortuntely, it turns out that each frame need to be transferred back from the frame-buffer in order to serialize it to file and this halts the GPU through-put heavily. For that reason 

I decided that being *#tofuengine* palette-based, there's not need at all to complicate things and requires colors to be specified in *packed* **R8G8B8** format. Either a pixel is referred using a palette index or a color with separate (not a tuple) red-green-blue components (in the range `[0, 255]`). For better separation-of-concerns a new `Palette` data-type has been added.

I was about to remove primitive's drawing support (lines, rectangles, circles, etc...), because they can't be as fast as I would like. However, I reconsidered it as they are not mean to be a high-speed operations but more like some kind of *debug* tools. Also, some cool effects can be implemented, for example a *segmented-bolt-laser* effect.

## I/O & Resources

  - performance, faster decoding
* The [*packed*](/tofu-engine-5) file-system support have been reworked. The original format was very straight-forward but not so efficient, the whole file need to be scanned sequentially to build the directory. This has been changed and now the ready-to-use directory is appended at the end of the archive itself. Also, every entry is now stored as a MD5 digest to keep the directory entries name constants, and the RC4 encryption has been replaced with a (simpler and faster) XOR. We could improve this even better by storing the directory sorted into the archive and performing a binary-search without the need to load the directory into memory.
  - configuration
  - command-line
  - user-space
- loading resource from Base64 encoded string.

## System



  - overloading
  - name mangling and RTTI
- Raspberry-PI support is now really working.

## Next to come

Next steps for the future

* WebAssembly output
* physics engine
* textured line
* documentation