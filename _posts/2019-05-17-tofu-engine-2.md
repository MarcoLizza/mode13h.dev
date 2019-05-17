---
title: 'tofu-engine #2'
author: marco.lizza
layout: post
permalink: "/tofu-engine-2"
comments: true
categories: 
  - tofu-engine
  - devlog
tags: 
  - raylib
  - glfw
  - sdl
  - wren
  - javascript
  - lua
---
During the last two months, the game-engine did a lot of progress. Several key features have been added, and this is a matter for another #devlog entry (which I'll post in the next days, hopefully).

It also has been a period of re-thinking.

Since I'm writing **#tofuengine** for personal use and out of external obligations (and without any given restriction or deadline) I can afford to be very picky with pretty much any aspect of it, namely the third-party software I'm integrating. Among the many, the most important parts of the engine are the *multimedia* ([RayLib](https://www.raylib.com/)) and the *scripting* ([Wren](https://wren.io/)) libraries. Unfortunately, I've been encountering some "hiccups" with both of them.

> I'm not saying they are *bad*, quite the contrary! It is just that they don't fully meet *my needs*. ;)

## RayLib

As a multimedia library, **RayLib** is really straightforward to use and also quite powerful. It supports out-of-the-box modern features. However, it is mostly a *framework* rather than a third-party library. It offers pretty much everything to create a game in no time but imposes a lot to the point that your application's structure is a bit too forced for me. Also, despite being actively developed, there are some features still missing that won't be added in the near future (e.g. passing a texture as uniform to a fragment-shader).

I'm considering to switch to [GLFW](https://www.glfw.org/) or [SDL](https://www.libsdl.org/) which leave more room for doing the things the way I really want. They both don't offer the same level of abstraction of **RayLib** (meaning I would require quite a bit of development), but it's something that would be really beneficial.

They both lack audio management, in a way, but that is something achievable with other libraries (e.g. [OpenAL](https://openal.org/) and [SDL_sound](http://icculus.org/SDL_sound/)).

## Wren

Wren is a **neat** scripting language. The language is nice, embedding API is a breeze to use, the interpreter is performant, the source-code is so compact and well-documented that easily enables customary changes...

... but it's pretty much SCONOSCIUTO.

The lack of proper development tools and library of scripts can be an issue in the long run.

It seems that switching to [Lua](https://www.lua.org/) would be the most intuitive thing to do, but while I really dig the language itself, its embedding API is way too verbose for my tastes. I could use an integration library such as [AutoLuaC](https://github.com/orangeduck/LuaAutoC/) (which is really great) but I'm debated in adding another third-party library for something that the scripting language itself should offer.

Another possibility is offered by [JerryScript](http://jerryscript.net/), and while I'm not really fond of *JavaScript*, the sheer number of available tools would be a huge plus... not to mention the change to write in [TypeScript](https://www.typescriptlang.org/).

## Conclusion

I'll take some more time to ponder about the aforementioned aspects. Presumably, the first step will be moving away from **RayLib** to **GLFW** (the best candidate for the moment being). For now, Wren is still the best option.