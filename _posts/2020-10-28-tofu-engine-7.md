---
title: tofu-engine #7
layout: post
permalink: /tofu-engine-7/
categories:
  - tofu-engine
  - devlog
tags:
  - sound
  - trackers
  - mixing
---
How deep does the rabbit hole go?

Quite a bit. :)

It all started in early August when a fellow **#gamedev** wrote a post about a game-prototype. It was nice and polished, with a nice CRT-like shader. However, the single thing that struck me the most was the tracker-based background music. Holy [Paula](https://en.wikipedia.org/wiki/Original_Chip_Set#Paula)! How could I overlook to add support for [module-files](https://en.wikipedia.org/wiki/Module_file) in **#tofuengine**?

I rapidly started to search for a module-file player library, with an initial clue leading to the one used in [Raylib's audio sub-system](https://github.com/raysan5/raudio). It turned out to be [jar_xm](https://github.com/kd7tck/jar), a cute single-header library (which I'm rapidly growing tired of using). Being just a "facade" for [libxm](https://github.com/Artefact2/libxm), I spent some time (re)working the latter (most notably to support callback driven-I/O and to optimize the player). Unfortunately, the more I was working on it, the more inaccuracies I found and was required to fix... what a pity!

( I'm not saying it's a bad project, but I didn't want to spend a significant time in writing/fixing the player library... notwithstanding, it has been an opportunity to learn about the deep intricacies of the module-file formats )

After some more searches, I landed upon [xmp](http://xmp.sourceforge.net/), which supports a vast amount of module-file formats and includes (being a derivative work of projects like [MikMod](http://mikmod.sourceforge.net/), [libmodplug](https://github.com/Konstanty/libmodplug), and [OpenMPT](https://openmpt.org/)) almost every format quirk. Most of all, it can be packed into a "lite" player, including XM, S3M, IT, and MOD formats. Currently, it seems no longer actively maintained (some occasional contributions excluded), but that's common among module player libraries. Also, that's not necessarily an issue as the development started many years ago, and the library has most likely reached a more than decent stability.

While adding the library, I took some time to polish it a bit. I ended in rewriting the I/O module almost entirely (refactoring into a single source file, w/o any code duplication) and adding the usual callback-driven I/O. I also refactored part of the library by making some simplifications and removing many warnings (luckily, it was was pretty much C99 compliant... but I left some -- disabled -- warnings still there as I'm not sure to want to fix them all for the moment).

( by the way, this was also the occasion to fix a lurking *packed* I/O issue, where seeking beyond a file end-of-data was possible )

Coding the audio and sound sub-system for a game-engine is (probably) the single most difficult task... and, of course, I found some issues lurking in the shadows. The most serious one was in the **mixing** routine, which was over-simplified and didn't take into account (possible) clipping errors when playing several tracks at the same time.

While I was there, I also added **2x2 matrix mixing**. That is, for every single source/group, the mixing parameters appears as a 2x2 matrix

```
| L/L R/L |
|         |
| L/R R/R |
```

with `X/Y` indicating the the ratio (i.e. "how much") of channel `X` channel transfers to channel `Y` (with `X` and `Y` that can be either left or right). That way, a single sample can be treated as a vector `[L R]` and processed with a matrix-vector product.

```
| L/L R/L |   | L |
|         | * |   | = | L/L * L + R/L * R, L/R * L + R/R * R |
| L/R R/R |   | R |
```

This is something that I initially felt unnecessary... however, it uncovers a wide range of effects (like stereo-panning and channel inversion), with panning and balance as corner cases. With some minor rework, I managed to pre-calculate the mixing matrixes for each group and source (given that each group and source can also have a specific *gain* level... albeit I'm not sure I'll leave it as a distinct parameter in the future) with not relevant performances drop.

Despite being efficient, I opted for a buffered approach when playing module files. Much like (FLAC) music, the sound data is periodically pre-generated into a buffer (one second long, for the moment). No additional computational and I/O overhead is required when the audio device requests sound data (mixing excluded, but I could also pre-mix into a buffer... umm...).
