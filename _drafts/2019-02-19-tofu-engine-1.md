---
title: 'tofu-engine #1'
author: marco.lizza
layout: post
permalink: "/tofu-engine-1/"
comments: true
categories: 
  - tofu-engine
  - devlog
tags: 
  - scripting
  - benchmark
  - opengl
---
In the last month I've completed what can safely be called *the first milestone*: `v0.1.0`. Well, to be honest, due to the fact it took quite a bit to write this post, I reached also `v0.2.0` in the meanwhile.

Starting from the idea of a *lo-fi* 2D game-engine I reached the point of having some (scripted) sprites bouncing of the screen. Also, basic drawing primitives has been implemented along with text drawing. Once picked the media library and the scripting language it has been just a matter of organizing some code, which has been straightforward (although there's still need for abstraction and better modularization, but this will come over time).

## Embedding API

## Palette support

There's a limit, due to uniforms. Realistically, I find that a palette of 64 colors is more than enough (NES palette).

Image re-indexind during load. Exact match at first, nearest-color later. Can cause some problems, if very-dark greys are used. Should exploit alpha avoid clashes?

## Sample applications

Bunnymark as a benchmark. Not too bad, but need to optimized.

Doomfire, useful to spot some issue with the pixel-perfect rendering with OpenGL primitives.

## What's next?