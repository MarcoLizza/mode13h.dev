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
Some years ago ([it seems like](/happiness-is/) like almost five years ago... man, how fast time flies!) I was working on a custom game engine. Needless to say, the project didn't see the light of day and was left abandoned somewhere in my hard-drives. To be honest, while I was super-exited of developing my own wheel (I'm always been the kind of developer that prefer to code his own version of algorithm/library/piece-of-tech to be in control... pretty much a waste of time, one could say), I wasn't willing to spend months of coding just to have something inferior to what the (abundant) market offered. Let's remember that my primary aim was to code games, not to code the engine itself.

So, left the game-engine idea in the corner, I spent the last years in trying and evaluating almost every engine/framework available. Initially, I was intrigued by the idea of a full-fledged IDE. However, on second thought, I'm not willing to sacrifice precious hard-disk space for something that I would use (to be generous) for less than 10% of its capabilities. Also, I'm not very keen on dealing with tools that require me to follow online courses only to create a simple application. So I discarded [Unity](https://unity3d.com/), [GameMaker: Studio](https://www.yoyogames.com/gamemaker), [Defold](https://www.defold.com/), [Xenko](https://xenko.com/), and the rest of the clan.

> In particular, I was really disappointed by *Game Maker*. I'm aware that a lot of amazing and successful games have been created with it... but I find its scripting language really disappointing. As a DSL it perfectly does its job, but it's way too basic. It can be perfect for a novice/intermediate programmer, but since I've been working with programming languages for some decades I'm willing to write in a more sophisticated one.

An alternative is [Godot](https://godotengine.org/), which is somewhat less famous than the others but it's quite interesting. Despite being very lightweight in requirements (only a handful of MiBs for the IDE and the engine), I really disliked learning a custom scripting language.

So, I ended up in picking one between [Corona](https://coronalabs.com/) and [LOVE](https://love2d.org). Both using **Lua** (a language I like and whose performance is undoubtful), compact in requirements, and multi-platform. On a final choice, I picked **LOVE**, which is really pleasant to use: fast, with everything one could need. Problem is that **LOVE** is more a *framework* than and *engine*, and over the years I modeled a custom engine over it, by encapsulating its API in my own personal one, to better suit my needs. At one point I started asking myself what the point of this, and the idea to resurrect my own game-engine started to tingle in my mind. On top of that I need to say that, despite liking *LOVE* there were some things I really disliked:

* the very slow development process, way too "tight" and controlled/limited by the authors,
* the dependency on SDL (I would rather switch to something more modern, such as *SFML*),
* the fact that it was using an old version of *Lua* (version 5.1, since *Luajit* development halted with this version),
* its name and the overall name usage in the global codebase.

