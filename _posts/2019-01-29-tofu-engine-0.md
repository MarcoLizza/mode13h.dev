---
title: 'tofu-engine #0'
author: marco.lizza
layout: post
permalink: "/tofu-engine-0/"
comments: true
categories: 
  - devlog
  - engine
tags: 
  - wren
  - raylib
---
Some years ago ([it seems like](/happiness-is/) almost five years ago... man, how fast time flies!) I was working on a custom game engine. Needless to say, the project didn't see the light of day and was left marooned someplace in my hard-drives. To be fair, while I was thrilled of creating my own wheel (I'm always been the kind of developer that prefer to code his own version of an algorithm/library/piece-of-tech to be in control... pretty much a waste of time, one could say), I hadn't a clear aim and wasn't enthusiastic to spend months of coding just to have a over-generic engine inferior to what the (abundant) market offers.

And the (inevitable) hiatus came.

The next few years flew while trying and evaluating almost every engine/framework available. Initially, I was intrigued by the idea of a full-fledged IDE. However, on second thought, I'm not willing to sacrifice precious hard-disk space for something that I would use (to be generous) for less than 10% of its capabilities. Also, I'm not very inclined in dealing with tools that require me to follow online courses only to create a simple application. So I discarded [Unity](https://unity3d.com/), [GameMaker: Studio](https://www.yoyogames.com/gamemaker), [Defold](https://www.defold.com/), [Xenko](https://xenko.com/), and the rest of the clan.n

> In particular, I was really disappointed by **GameMaker: Studio**. I'm aware that a lot of amazing and successful games have been created with it... but I find its scripting language really disappointing. As a DSL it perfectly does its job, but it's way too basic. It can be perfect for a novice/intermediate programmer, but since I've been working with programming languages for some decades I'm willing to write in a more sophisticated one.

An alternative is [Godot](https://godotengine.org/), which is somewhat less famous than the others but it's quite interesting. Despite being very lightweight in requirements (only a handful of MiBs for the IDE and the engine), I really disliked learning also a custom scripting language.

So, I ended up in picking one among [Corona](https://coronalabs.com/), [Amulet](https://www.amulet.xyz/), and [LÖVE](https://love2d.org). All using **Lua** (a language I like and whose performance is undoubtful), compact in requirements, and multi-platform. On a final choice, I picked **LÖVE**, which is really pleasant to use: reasonably fast, with everything one could need. Problem is that it's more a *framework* than and *engine*, and over the years I modeled a custom engine over it, by encapsulating its API in my own personal one, to better suit my needs. I added automatic display auto-scaling, a decent 2D matrix/vector library, resource hot-reload, automatic sprite-sheet cutting, and tiled-map support... so I started asking myself what the point of developing an engine over a framework. So, one day while I was thinking about implementing a shader-based palette-driven graphic simulation (similar to VGA's `Mode 13h`), I asked myself *<< Why don't you simply write your own game-engine >>*.

And here we are.

On top of that I need to say that, despite liking **LÖVE**, there are some things I would change:

* the very slow development process, way too "tight" and controlled/limited by the authors,
* the dependency on **SDL** (I would rather switch to **SFML**),
* the fact that it was using an old version of **Lua** (version *5.1*, since **Luajit** development halted with this version),
* its sometime not coherent and (too) generic API, in term of features, that can adapt to every developer's need but requires a decent amount of boilerplate code, and
* its name and the overall mood of the community (infested with too much frat humor).

The game-engine requisites for my game-engine haven't changed much over the years, that is

* it must be 2D pixel-art oriented,
* it must be in the form of a single executable w/o dependencies from third-party shared libraries (DLLs and/or SOs),
* it needs to support more than one platform (at least *Windows*, *Linux*... possibly also *macOS*),
* it will have a simple API,
* it will support additional features, such as fragment shaders and resource hot-reloading.

In order to speed up the development process, I decided to leverage on other libraries. Most of all, as a multimedia library I picked [raylib](https://www.raylib.com/) since it can be compiled as a single standalone static library with no other external dependencies (apart from the obvious OS libraries). As an embedded scripting language I choose [Wren](https://wren.io/) for its performances and since it features probably the most seamless and easy-to-use API. Other other modules/libraries are going to be used (e.g. to parse JSON files), of course.

To spice things up, I'm also writing the engine in plain [C99](https://en.wikipedia.org/wiki/C99) without using any IDE or development tools ([Visual Studio Code](https://code.visualstudio.com/) apart). :)