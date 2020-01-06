---
title: 'tofu-engine #5'
author: marco.lizza
layout: post
permalink: "/tofu-engine-5"
comments: true
categories:
  - tofu-engine
  - devlog
tags:
  - porting
  - file-system
  - gamepads
published: false
---

## Input handling

Gamepad input almost complete. Sticks and triggers are deadzone-filtered. Left sticks emulates the DPAD, right stick the mouse. Multiple controllers are supported, fallback to keyboard/mouse. `F1` key to switch.

## Interpreter

The Lua integration has been polished. The module initialization has been refined and the (module-level) C API *upvalues* have been cleared; it is no longer used a single container structure grouping all the sub-systems instances, but separare upvalues. Also, the `io` and `os` predefined libraries have been removed (as they are potentially dangerouse since the permit to go outside the engine's sandbox environment). The boot script has been reworked and extended (and a geeky "Guru Meditation" crash-screen has been added in the debug build), the argument checking has been optimized (types are no longer tested with a function), and script processing is more robust. I also tested Lua scripts pre-compilation, which I was courious about, and it isn't worth the effort since the final bytecode file is larger than the plain script (which potentially complicate things up).

## Graphics pipeline

Graphics reworked, non-monochrome font support, final drawing offset. Useful for screen effects (e.g. shaking or scrolling). Will add stencil next, probably. Custom fragment shader is a candidate to be removed.

## Virtual file-system

Implemented custom packed file-system access. Very simple, read-only, with encryption.

## Windows support

A major milestone has been **porting** through cross-compilation the engine to **Windows**. This was mostly a matter of modifying adding suitable library binaries for GFLW3 (which are available for Windows in MinGW format), and changing the `Makefile` file in order to use the correct compiler/linker/flags. No relevant changes were needed to the engine itself. Finally being able to launch it on Windows is awesome and boosts the percentage of supported platform by a huge amount! :) While I was there, I added also an experimental support for **Raspberry-PI**. The sole relevant platform missing is MacOS... but, unfortunately, I haven't a sufficient experience with it. Probably I would use a helping hand from a fellow coder.

## Next to come

This brings us to the next topic: *going public*. **#tofuengine** has been developed with no ambitions of being a freely available and usable game engine. Surely, it's open in nature and I'm more than happy to share it with the community. However, it's a project tailored on the needs and ideas of a single coder (me). It still isn't feature complete, it lacks proper documentation... but...

... what if someone else find it useful and/or is willing to collaborate? The idea begins to tantalize me...

Major feature missing: audio support.
