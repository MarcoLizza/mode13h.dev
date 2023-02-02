---
title: A modern one, please
author: marco.lizza
layout: post
permalink: /a-modern-one-please/
comments: true
categories:
  - engine
tags:
  - scripting
  - lua
  - squirrel
---
Some months ago I started developing a [(yet another) engine][1]. Needless to say, the boundary conditions changed so I moved to another one. This time will be a more **game oriented** engine.

Most of all, it will be a *modern* engine as much as possible.

*( the precise use of the engine will be revealed in the next future )*

In the next posts I'll dwell into the subjects that are mattering the most now that I'm designing it.

For now, let's start with...

### The Scripting-System

I'm very fond of Lua (maintaining a [Windows CE][2] port myself). Small memory footprint, great performances and a RISC-like approach which is one of its strongest aspect. However, implementing something which is not available out-of-the-box may lead to very cluttered scripts.

*( from time to time, some syntactic-sugar is appreciated )*

As an example, OOP can be implemented for sure, but the resulting code is no way intuitive for someone looking at the script for the first time (seasoned programmers included).

I'd like to have compact and easy-on-the-eye script so I opted for something different (agreed that I'm not going to implement my own domain-specific scripting language this time). Possibly a portable, actively developed, mature enough and (to some extent) general purpose language.

*Alberto Demichelis* apparently faced the same very problems (and some more) when conceiving [Squirrel][3]. Sharing most of the basic concept with Lua and featuring a similar stack-based VM, Squirrel is a valid "relative" to use: it's compact, portable, memory-nice and fast.

*( and it features coroutines, too! )*

The main drawbacks are the lack of documentation and serious debugging tools (like [ZeroBrane Studio][4]), its relatively small community and somelike cumbersome API leading to redundant code when interacting with the native part of the application.

Perhaps the integration API is something I still would prefer to be more C++ oriented. [ChaiScript][5] is a possible alternative, but it would lead to a far bigger executable (due to its dependency upon Boost), higher memory and CPU (see [this][6] and [this][7]). Performances are a lower priority issue, since nobody should be planning to do heavy computing on the scripted side of their engines.

*( in my wildest dreams I go and use [TinyScheme][8], however )*

To sum things up, for now let's go for Squirrel!

### What's Left

Of course, there are several other aspects that need to be clarified and a decision need to be made:

 * **multimedia**: should I use SDL? SFML? FMOD? Bass? Or implement the sub-system by myself?
 * **cross-platform**: will it be a Windows-only engine? Or I plan to port it to other platforms, too?
 * **architecture**: old-fashioned OOP hierarchies or component based design?

 [1]: //mode13h.dev/happyness-is
 [2]: //luace.codeplex.com
 [3]: //www.squirrel-lang.org
 [4]: //studio.zerobrane.com
 [5]: //chaiscript.com
 [6]: //codeplea.com/game-scripting-languages
 [7]: //pastie.org/1721408
 [8]: //tinyscheme.sourceforge.net