---
title: 'tofu-engine #7'
author: marco.lizza
layout: post
permalink: "/tofu-engine-7"
comments: true
published: false
categories:
  - tofu-engine
  - devlog
tags:
  - sound
  - feature
  - trackers
  - mixing
---
How deep does the rabbit hole go?

Apparently, quite a bit. :)

It all started in early August, when a fellow **#gamedev** wrote a post about a prototype for a game he was working on, highlighting the presence of a tracker-based background music. Holy [Paula](http:www.paula.com)! How could I simply oversee the support for these in my engine? Tracker-music is compact, efficient.

Started to search for nice tracker player-library. Starting from JAR, used in Raylib. Cute, single header... but it was just a "facade" for [libxm](https://github.com/Artefact2/libxm). So, I spend a week working on **libxm** by extending the I/O API to my purposes, only to find it has some inaccuracies in the player routine.

Ditched.

Scanned to net for other libraries, after many tries I landed on [xmp](http://xmp.sourceforge.net/), which does support many tracker formats. It also can be packed in a "lite" version that includes only XM, S3M, IT, and MOD support. It seems no longer actively supported (the last commit was done 15 months ago), but being started many years ago it has most likely reached a more than decent stability. It's apparently also inspired by modplug and openMTP.

Cool!

I worked in adapting the I/O API, also here, to add callback-drive I/O. I ended in rewriting this part almost interely (not it's a single file, with no code duplication). I also tweaked the code by simplifying some parts and removing many warnings (some are still there, but I'm not sure to want to fix them all).

Streamed approach, like FLAC streaming. 1 second of streaming is probably too low.

Fixed PAK I/O, seeking out of data was possibile.

Mixing was wrong, clipping errors.

Added 2D mixing matrix, twin-pan. Precalculation required some work, too costy otherwise.

FLAC streaming is worth?
