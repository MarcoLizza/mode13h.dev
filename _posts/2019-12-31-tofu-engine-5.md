---
title: tofu-engine #5
layout: post
permalink: /tofu-engine-5/
categories:
  - tofu-engine
  - devlog
tags:
  - multi-platform
  - file-system
  - input
---
End-of-year recap post. Oh! It seems that this year I've been writing exclusively about `#tofuengine` (averaging in a post every two months)... which, to be honest, doesn't surprise me as it has been the only off-job project I've been following.

Anyway, it's been a while since the last update and many things have changed in this iteration... and as any self-respecting newborn, in its first years of life, the engine has grown and changed *a lot*. :)

In its current state, the engine is almost feature-complete (given that we can continue adding features endlessly) with a single major exception: **audio support**. This has been only sketched by picking a support library (namely [miniaudio](https://github.com/dr-soft/miniaudio)) but nothing more. This is going to be the next step to be done.

Here's a summary highlighting last months' activity.

## Windows support

Honestly, when I decided to use Linux and GCC as a development environment I wished not to be forced to use [Visual Studio](https://visualstudio.microsoft.com/) (despite knowing it well and using it since twenty years) to carry on the Windows porting of the engine. Well, guess what? My wish came true! :)

With GCC's MinGW cross-compiler this was just a matter of modifying adding suitable library binaries for [GFLW3](https://www.glfw.org/) (which are available for Windows in MinGW format), and changing the `Makefile` file to use the correct compiler/linker/flags. No relevant changes were needed to the engine itself, which was already as much platform-independent as possible.

On the first launch on Windows I was holding my breath... and it just worked! Whoo-ooh!

Quite surprisingly I found that FPS performances of the Linux VM I'm using as a development machine aren't as bad as I feared. Also, on my laptop, using the onboard Intel graphics card gives better performances than the nVIDIA one. :\

While I was there, I added also experimental support for **Raspberry-PI**.

Having landed on Windows boosts the total percentage of supported-platforms by a huge amount! The single relevant desktop platform still missing is **MacOS**. Unfortunately, I have almost no development experience with it. I imagine we could be leveraging GCC once again... but I would probably use a helping hand from a fellow coder. :)

## Virtual file-system

I always loved *packed* file-system access. Since the mid-'90s, when [Doom's WAD](https://en.wikipedia.org/wiki/Doom_WAD) format was documented, I wrote several WAD-like formats (even complete file-based file-systems with read/write access). Most games used a similar approach for ease of distribution, as it's far more simple to pack all game data in a single file rather than hundreds of smaller files across a (sub)folder.

Initially, I thought about using an existing archive format such as ZIP or TAR. The immediate benefit is the ease of creation and handling since a plethora of archive-managers already exists. Unfortunately, I wasn't able to find a small, simple, and clean library that supports them. There are some good instances but they don't fit my needs. I had a peek of [PhysFS](https://icculus.org/physfs/) which is an amazing library *but* way too big for my purposes (I don't need to support a lot of different formats, but a single one in the best/easiest way possible).

Yes, you can guess how it ended. I wrote a custom packed file-system access sub-system... and, as usual, this turned into a really interesting and fun task. :)

I stuck to the simplest format I could think of, with the following requested features:

* seamless integration with both folder- and file-based access,
* random read-only access (file can't be runtime changed by the application),
* low memory footprint,
* fast I/O operations.

At first, I considered (and also implemented) to support compression. However, it turned to be both pretty much useless as most file entries (namely the bigger ones) are already stored in compressed native format (e.g. PNG and OGG). Adding compression would slow-down the I/O and require an additional memory buffer. Both two big no-no for me. Integrity check, also, isn't required as in the case a file is malformed/corrupted an error would eventually rise and detecting the corruption won't fix the error (unless we would be doing some sort of parity-check). The only benefit would be to prevent file tampering/hacking... but that's not something I want to protect from (you're welcome and hack my games if you like :)). Just for sake of fun, I added a simple stream cipher, to enable a minor countermeasure for the ones that bother. The stream cipher doesn't require additional memory (encryption/decryption key excluded) and the processing overhead is small.

Having not to support updates of the archive made things easy. A packed archive begins with a `Pak_Header_t` structure

```c
typedef struct _Pak_Header_t {
    char signature[8];
    uint8_t version;
    uint8_t flags;
    uint16_t __reserved;
    uint32_t entries;
} Pak_Header_t;
```

No file-format is complete without signature/magic-number and a version indicators... and the `signature` and `version` fields fulfil this purpose (with `signature` equal to `TOFUPAK!` and `version` equal to `0` for the moment). `flags` reports additional information about the archive (e.g. whether the archive is encrypted), and `entries` the amount of file stored in the archive. `__reserved` is a -- erm -- reserved field we added to *think-ahead* and save us from future headaches in the case we need to store more data in the header.

The archive header is, then, followed by the first entry's header.

```c
typedef struct _Pak_Entry_Header_t {
    uint16_t __reserved;
    uint16_t name;
    uint32_t size;
} Pak_Entry_Header_t;
```

Once again we have `__reserved` field (used both for alignment purposes and because we never know if in the future we'll need it). The entry header is followed by `name` bytes indicating the entry name, and `size` bytes (the entry proper data).

No directory is stored in the archive. During the initialization phase, the archive is scanned and the resources directory is built-up in memory. This surely adds ups to the engine memory footprint, but it ensures fast entry look-up once the entries are (q)sorted and binary-searched (reaching a more than respectable `O(n log n)` complexity).

> Please note that the filenames are case-**in**sensitive. Case-sensitiveness file-systems are *evil* to me. :)

In order to create `TOFUPAK` archives a basic Lua script has been written. Nothing fancy, it just scans and packs a folder into a file (encrypting if necessary).

When launched with no arguments, the engine search in the current folder (not recursively) for any `.pak` file and attaches them all as mount-points. It also attaches to the current folder as last (most recent) a mount point.

> The maximum amount of entries per archive is huge (`2^32 - 1` entries) and a single archive can store all the required data for any game. However, since multiple mount-points/archives are supported by the engine resources can be split in different archives. This also has the side-effect that, if an entry with the same name is present in more than one archive, the lexicographically last one is used enabling this way for *resource override*.

## Interpreter

The integration with Lua's VM has been polished in several spots.

The module initialization has been refined and the way *up-values* are passed to the (C side) modules have been cleared; we no longer use a single container structure that groups all the sub-systems' instances, but separate up-values.

As a security measure, the predefined `io` and `os` Lua libraries have been removed. They are potentially dangerous as they enable access to the OS outside the engine's sandbox environment.

The boot script has been reworked, extended, and a geeky [Guru Meditation](https://en.wikipedia.org/wiki/Guru_Meditation) crash-screen has been added (in the debug build). Also, the (debug build) C API argument checking has been optimized (types are no longer tested with a function), and the script loading and processing is more robust.

By leveraging Lua's `lua_Reader` API, when a script is loaded and interpreted (when a `require` statement is encountered) we no longer need to (pre)load the whole file into a temporary buffer and *then* pass it to the interpreter. Using a custom reader, the VM can load-and-process the file autonomously, requesting smaller chunks as needed. This fits well with the file-system virtualization made.

Lua script can be provided in two flavors: plain text, or pre-compiled byte-code. Support for pre-compiled scripts was missing until now. To be honest, I don't think script pre-compilation is worth the effort. I was curious about it, but the pre-compiled script is far larger than the original one. Although the VM should process it faster, the loading process slows down a bit. This adds also an additional layer of complexity, potentially complicating the game packing process. One benefit of the pre-compilation is the static check of the script, but this is something that [luacheck](https://github.com/mpeterv/luacheck) does even better and can be integrated in the IDE (for example, in [Visual Studio Code](https://code.visualstudio.com/) there exists an [proper extensions](https://marketplace.visualstudio.com/items?itemName=trixnz.vscode-lua)) or in the building tool-chain. Anyway, this is something the engine does support so if one wants to pack pre-compiled scripts... it is possible.

Once more I'm happy I chose not to use any Lua-to-C integration library. Lua's C API is really easy and powerful to use, once you grasp some of the key concepts (mostly the stack). After some time it just feels natural... and in some way, I fell (pleasant) echoes of assembly programming when I'm dealing with it. :)

## Input handling

For the very beginning, I aimed for a simple out-of-the-box and uniform input handling. Switching from keyboard to gamepad and back should be seamless, and for this reason, I started and designed the keyboard input to mimic a modern controller (think of the PS3/PS4 and Xbox-360/Xbox-ONE ones... minus the thumb-sticks) with D-PAD and the full set of A/B/X/Y/L1/R1/L2/R2 buttons.

Despite being the engine oriented to "arcade" gaming, mouse input has been included. It can be possibly useful in certain circumstances and supporting it was plain easy.

As for gamepad handling and support, GLFW makes it easy (starting from `v3.3` the gamepad support has been improved). For the digital axes (i.e. thumb-sticks and triggers) all I had to do was to filter the input with some *deadzone* logic. For the buttons, the mapping is straightforward and handled by SDL's `gamecontrollerdb.txt`.

As additional features, thumb-sticks can be optionally used to emulate the D-PAD (left thumb-stick) and the mouse (right thumb-stick). For the former, a configurable threshold is used to detect the direction, for the latter a cursor-speed factor.

When multiple gamepads are detected, they can be switched by pressing the `F1` key (keyboard/mouse included). Gamepads and keyboard/mouse are mutually exclusive and, for the moment, **multiple input** is not supported.

## Graphics pipeline

The graphics pipeline has been partially reworked. Non-monochrome font support has been added, and offscreen-to-framebuffer drawing offset has been introduced (useful for simple effects like shaking or scrolling).

Will add stencil support next, probably.

The custom fragment shader support is a candidate to be removed.

## Next to come

This brings us to the next topic: *going public*. **#tofuengine** has been developed with no ambitions of being something that someone else besides me could use. Surely, it's open to anyone interested since it's birth and I'm more than happy to share it with the community. However, it's a project tailored to the needs and ideas of a single coder (me).

It still isn't feature-complete, it lacks proper documentation... but...

... what if someone else finds it useful and/or is willing to collaborate? The idea begins to tantalize me...