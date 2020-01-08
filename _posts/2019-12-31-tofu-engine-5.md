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
  - gamepad
published: false
---
End-of-year recap post. It's been (as usual) a while since the last one. Couriously enough, in the last year I wrote only posts related to `#tofuengine`. Well, that's not really strange since it has been the sole off-work project I've been following.

In its first 12 months of life, the engine has seen a lot of changes. From the initial *quick-and-dirty* implementation to the current more polished one, switching from C++ (only in the earliest prototype) to C99, moving from an GPU-based approach to an custom software renderer... it's been a lot of fun! :)

In the current state, the engine is almost feature complete (given that we can always add new features) with a major exception: **audio support**. It has been sketched by picking a support library (namely [miniaudio](https://github.com/dr-soft/miniaudio)) but nothing more. This is going to be the next step to be done.

Following a brief summary of the highlights of last months' of work.

## Windows support

A major milestone has been **porting** through cross-compilation the engine to **Windows**. This was mostly a matter of modifying adding suitable library binaries for GFLW3 (which are available for Windows in MinGW format), and changing the `Makefile` file in order to use the correct compiler/linker/flags. No relevant changes were needed to the engine itself. Finally being able to launch it on Windows is awesome and boosts the percentage of supported platform by a huge amount! :)

Quite surprisingly I found that FPS performances of the Linux VM I'm using as a development machine aren't as bad as I feared. Also, on my laptop, using the on-board Intel graphics card gives better performances than the nVIDIA one. :\

While I was there, I added also an experimental support for **Raspberry-PI**.

The single relevant desktop platform still missing is **MacOS**. Unfortunately I have almost no development experience with it. I imagine that the engine should be portable leveraging GCC... but I would probably use a helping hand from a fellow coder. :)

## Virtual file-system

I always loved *packed* file-system access. Since the mid '90s, when [Doom's WAD](https://en.wikipedia.org/wiki/Doom_WAD) format was documented, I wrote several WAD-like formats (even complete file-based file-systems with read and write access). Most games used a similar approach as a mean to protect from hacking and for ease of distribution. In fact, it's far more simple to pack all game data in a single file rather than hundreds of smaller files across a (sub)folder.

Initially I though about using an existing archive format such as ZIP or TAR. The immediate benefit is the ease of creation and handling since a plethora of archive-managers already exist. Unfortunately, I wasn't able to find a small, simple, and clean library that supports them. There are some good instances but they don't fit my needs. I had a peek of [PhysFS](https://icculus.org/physfs/) which is an amazing library *but* way too big for my purposes (I don't need to support lot of different formats, but a single one in the best/easiest way possible).

Yes, you can guess how it ended. I wrote a custom packed file-system access sub-system... and, as as usual, this turned into a really interesting and fun task. :)

I sticked to the simplest format I could thing of, with the following requested features:

* seamless integration with both folder- and file-based access,
* random read-only access (file can't be runtime changed by the application),
* low memory footprint,
* fast I/O operations.

At first I considered (and also implemented) to support compression. However, it turned to be both pretty much useless as most file entries (namely the bigger ones) are already stored in compressed native format (e.g. PNG and OGG). Adding compression would slow-down the I/O and require an additional memory buffer. Both two big no-no for me. Integrity check, also, isn't required as in the case a file is malformed/corrupted a error would eventually rise and detecting the corruption won't fix anything. The only benefit would be to prevent file tampering/hacking... but that's not something I want to protect from (you're welcome and hack my games if you like :)). Just for sake of fun I added a simple stream cipher, in order to enable a minor countermeasure for the ones that really bothers. The stream cipher doesn't require additional memory (encryption/decryption key excluded) and the processing overhead is small.

Having not to support updates of the archive made the things really easy. A packed archive begins with a `Pak_Header_t` structure

```c
typedef struct _Pak_Header_t {
    char signature[8];
    uint8_t version;
    uint8_t flags;
    uint16_t __reserved;
    uint32_t entries;
} Pak_Header_t;
```

No file-format is complete without signature/magic-number and a version indicators... and the `signature` and `version` fields fullfil this purpose (with `signature` equal to `TOFUPAK!` and `version` equal to `0` for the moment). `flags` reports additional information about the archive (e.g. whether the archive is encrypted), and `entries` the amount of file stored in the archive. `__reserved` is a -- erm -- reserved field we added in order to *think-ahead* and save us from future headaches in the case we need to store more data in the header.

The archive header is, then, followed by first entry's header.

```c
typedef struct _Pak_Entry_Header_t {
    uint16_t __reserved;
    uint16_t name;
    uint32_t size;
} Pak_Entry_Header_t;
```

Once again we have `__reserved` field (used both for alignment purposes and because we never know if in the future we'll need it). The entry header is followed byt `name` bytes indicating the entry name, and `size` bytes (the entry proper data).

No directory is stored in the archive. During the initialization phase the archive is scanned and the resources directory is built in-memory. This is surely add ups to the engine memory footprint, but it ensure really fast entry look-up once we (q)sort and binary-searched the entries (reaching a more than respectable `O(nlogn)` complexity).

> Please note that the filenames are case-**in**sensitive. Case-sensitiveness file-systems are *evil* to me. :)

In order to create `TOFUPAK` archives a basic Lua script has been written. Nothing fancy, it just scan and pack a folder into a file (encrypting if necessary).

When launched with no arguments, the engine search in the current folder (not recursively) for any `.pak` file and attaches them all as mount-points. It also attaches to the current folder as last (most recent) a mount point.

> The maximum amount of entries per archive is huge (`2^32 - 1` entries) and a single archive can store all the required data for any game. However, since multiple mount-points/archives are supported by the engine resources can be split in different archives. This also has the side-effect that, if an entry with the same name is present in more than one archive, the lexicographically last one is used enabling this way for *resource override*.

## Interpreter

The integration with Lua's VM has been polished.

The module initialization has been refined and the (module-level) C API *upvalues* have been cleared; we are no longer using a single container structure that groups all the sub-systems' instances, but separate upvalues.

Also, the `io` and `os` predefined libraries have been removed (as they are potentially dangerous since the permit to go outside the engine's sandbox environment).

The boot script has been reworked and extended (and a geeky "Guru Meditation" crash-screen has been added in the debug build), the argument checking has been optimized (types are no longer tested with a function), and script processing is more robust.

Scripts loading has also been enhanced, by leveraging Lua's `lua_Reader` API. When a script is loaded and interpreted (when a `require` statement is encountered), we no longer need to load the whole file into a temporary buffer and *then* pass it to the interpreter, but the VM can load-and-process the file autonomously requesting smaller chunks as needed. This fit quit well with the file-system virtualization made.

I also tested Lua scripts pre-compilation, which I was courious about, and it isn't worth the effort since the final byte-code file is larger than the plain script (which potentially complicate things up).

Once more I'm really pleased I chose not to use any Lua-to-C integration library. Lua's C API is really easy and powerful to use, once you grasp some of the key concepts (mostly the stack). After some time it just feels natural... and in some way I fell (pleasant) echoes of assembly programming when I'm dealing with it. :)

## Input handling

For the very beginning I aimed for a simple out-of-the-box and uniform input handling.

I designed the keyboard input to mimic a modern controller (think of the PS3/PS4 and Xbox-360/Xbox-ONE ones... minus the thumb-sticks). Adding the mouse input was a not neccesary, but welcome, addition.

However, input handling could be complete without supporting real-life gamepads... and, fortunately, GLFW helps in handling them (and starting from `v3.3` the gamepad support has been really improved).

Gamepad input almost complete. Sticks and triggers are deadzone-filtered. Left sticks emulates the DPAD, right stick the mouse. Multiple controllers are supported, fallback to keyboard/mouse. `F1` key to switch.

## Graphics pipeline

Graphics reworked, non-monochrome font support, final drawing offset. Useful for screen effects (e.g. shaking or scrolling). Will add stencil next, probably. Custom fragment shader is a candidate to be removed.

## Next to come

This brings us to the next topic: *going public*. **#tofuengine** has been developed with no ambitions of being something that someone else besides me could use. Surely, it's open in nature and I'm more than happy to share it with the community. However, it's a project tailored on the needs and ideas of a single coder (me).

It still isn't feature complete, it lacks proper documentation... but...

... what if someone else find it useful and/or is willing to collaborate? The idea begins to tantalize me...
