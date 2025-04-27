---
title: 'Multimedia Backend Shenanigans'
author: marco.lizza
layout: post
permalink: "/multimedia-backend-shenanigans"
comments: true
categories:
  - tofu-engine
  - devlog
tags:
  - engine
  - backend
  - architecture
---
When I started working on **Tofu Engine** (way before its name was decided... and even before I had my first dog who, coincidentally, is also named Tofu) the first decision I made was that it *had to be multi-platform*.

Beyond more or less debatable personal choices on the technology stack (indeed, it would make sense to write a post describing it a little more in detail, with a nice description of the architecture), in order to make a game-engine work on different platforms (although now, much more than in the past, they significantly resemble each other) one fundamental problem must be solved: how to manage the multimedia compartment in a uniform way.

Because... well... in layman terms, all a video game down in its core has to do is:

* receive inputs from the user,
* display images, and
* play sounds.

( and, of course, another ton of things :D )

If we were only to code on Windows (or derivative systems, such as the [Xbox](https://it.wikipedia.org/wiki/Xbox)), we would natively have at our disposal [Direct-X](https://en.wikipedia.org/wiki/DirectX) which, despite its age (it is now almost thirty years old) and above any criticism, absolves these three functions excellently and uniformly. Once you come to terms with DCOM and its idiosyncrasies, there is no need for any additional third-party components, as everything have already been included in the operating system itself since Windows 95-OSR2.

But, needless to say, it would have been too simple to just support Windows, with its huge market share in gaming. We also want to support Linux[^1]. And maybe some ARM architecture like the one running Raspberry and Ambernic devices. Or Android devices! And why not [WebAssembly](https://en.wikipedia.org/wiki/WebAssembly) to have games our running into a web-browser?

( yes, for the moment that's all the support macOS is getting :D )

Luckily, we do not have to work like hell and write a distinct and different multimedia support for each of the target platforms. There are valid third-party libraries that allow us to relieve ourselves of this task.

Initially, I chose to rely on [SFML](https://www.sfml-dev.org/). At that early stage, I took it for granted that I wanted to use C++ for writing code, and this library seemed the most appropriate. When I had discarded C++ in favour of C99 (the reason why I did this is a topic for another post) I also moved to [Raylib](https://en.wikipedia.org/wiki/Raylib) to have as quick as possible a proof-of-concept of what later became **Tofu Engine**. A while later, I finally settled to [GFLW](https://www.glfw.org/) which is still in use today.

( If you are curious to read some of the details of this initial phase of development of the **Tofu Engine**, I suggest you read [this post](/tofu-engine-3) ).

GLFW is an amazing piece of software, and fulfils its task really well... however, over the the past year and a half I have grew increasingly uncomfortable. I began to question my initial choice of using GLFW as I observed that the library support from the authors (and contributors) was becoming increasingly slower and scarce at the point that the latest (minor) update dates more than one year ago. Which is not necessarily an issue when a project is stable and feature complete... but in the case of GLFW there is a conspicuous number of flaws awaiting resolution, and many missing features that I begin to fear will never see the light of day.

I held on for a long time, fighting my urge to find a solutions to this. Few things are worse that realizing you can't trust the tech on top of which you have build your game-engine. It's an obsession of mine to keep libraries always up-to-date so I forced myself to ignore that little voice inside me that kept complaining about it... until one day....

... on 21 January 2025, after a very long wait, [SDL3](https://discourse.libsdl.org/t/announcing-the-sdl-3-official-release/57149) was released.

I initially had chosen not to use SDL because it seemed a bit (too much?) overabundant in the amount of features (and APIs) offered. I feared to bloat my codebase too much... and a tiny part of me didn't want to be spoiled that much, as I didn't want to deprive myself completely of the fun of reinventing the wheel! :D

( just kidding... sort of... :P )

Anyhow, SDL3 just seems *way too interesting* to be ignored, in particular:

* it is written with full C99 support in mind, thing that not even GLFW does (I'm not proud of this, but I had to resort to a few tweaks to its code to selectively "silence" some of the more pedantic warnings that I personally enable by default and treat as blocking errors);
* it has a far better gamepad support than ANY OTHER library (especially on Linux). I realised when I was working to the source code of another backend ([RGFW](https://github.com/ColleagueRiley/RGFW)) with the idea of extending it. Taking GLFW as a reference, I noticed that the gamepad support on Linux is really very (but very) primitive compared to SDL's which is Steam-compliant.
* it has native support for the [always-up-to-date database of gamepad definitions](https://github.com/mdqinc/SDL_GameControllerDB). Also GLFW support it, but only at a very basic level without really taking into account the exceptions represented by special input devices (such as those on consoles);
* it has full support for **haptic feedback**, the lack of which (I have complained about it before)[/back-on-track];
* it is supported by [Emscripten](https://en.wikipedia.org/wiki/Emscripten), which would comfortably open the door to [WebAssembly](https://en.wikipedia.org/wiki/WebAssembly) and to browsers support;

But, above all, the fact that it is used by a **huge** number of commercial products makes us safe from the possibility of unexpected flaws and lack of continued and future support.

So... well, all that's left to do is to start reworking the innermost part of the game-engine code and perform a delicate surgery. The doubts are still many (how complicated will it be to include and compile the code of SDL3 so as not to have external dependencies? Will it be possible to exclude certain “sub-modules” of the library?)...

... but it will definitely be a lot of fun. :)

[^1]: Actually, I have always considered **of primary importance** supporting Linux. Nowadays, Linux is a more interesting and stimulating reality from the gaming point of view. Thanks to Valve's SteamDeck the interest to the idea of *Gaming-on-Linux* (for the past three years) has shifted and attracted a lot of attention.
