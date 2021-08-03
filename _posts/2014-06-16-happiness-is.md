---
title: 'Happiness is...'
author: marco.lizza
layout: post
permalink: /happiness-is/
categories:
  - development
  - engine
tags:
  - arm
  - engine
  - lua
  - windowsce
---
... writing a brand new app-engine! :)

In my spare time, I'm currently developing a common basic framework as an aid for my daily professional job.

I'm sick and tired in not having a rapid and optimized framework to use. Writing native applications is always fun, but leaves low to no margin of customization once delivered. Higher level frameworks (such as .NET Compact Framework or HTML5) are certainly versatile, but lacks in optimization in resource usage. So, after dilly-dallying for quite a long time, I started and developed my own personal app-engine.

Well, to be honest this is going more like a game engine in strict terms, since it features many of the characteristics of the latter.

It's targeted (at first) to Windows CE ARM-based handheld mobile devices where memory and CPU usage is a **major** concern. For this reason it needs to be as compact and performant as possible. The main highlights are

* event-driven architecture;
* using Lua as scripting engine ([my Windows CE porting](http://luace.codeplex.com));
* easy-to-use internal HTTP client;
* optimized 2D graphic module with full alpha-blending support (16bpp bitmaps with 5 bit alpha channel);
* audio module with real-time multi-channel mixer, waveform synthesis, and playback;
* common formats support (PNG, JPEG, BMP, WAV, MP3);
* written in C++ as much portable as possible, by encapsulating platform dependencies in easy-to-port layer classes.

On top of this, some extra features are needed. Due to the strict dependency with the hardware itself, they will be certainly added as modules (possibly Lua modules?). Haven't figured yet what's the best approach. :)

It's quite a rewarding activity to create such an app-engine, and in the next weeks I'll be describing in more in details its development.

Stay tuned! :)