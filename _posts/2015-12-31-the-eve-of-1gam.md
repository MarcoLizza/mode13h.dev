---
title: 'The Eve Of #1gam'
author: marco.lizza
layout: post
permalink: "/the-eve-of-1gam/"
comments: true
categories: 
  - 1gam
tags: 
  - 1gam
  - gamedev
  - indiegame
  - love2d
---

It's been a while since I decided to take the challenge and try and develop one game in a month (for a year). Today is New Year's Eve and we are approaching the starting line, finally. Since it's my first experience with a third party engine, I've given myself a head-start. In the last weeks I've been making myself comfortable with the game engine ([L&Ouml;VE][1]) and the tool-chain I'll be using, while waiting to begin.

Speaking of which, I'm also trying to keep it to a bare minimum, that is

 * [PiskelApp][6] (for rapid on-the-fly pixel-art prototyping),
 * [PyxelEdit][7] (for pixel-art editing),
 * [Tiled][8] (for tile-map editing, although PyxelEdit does support this in some way),
 * [L&Ouml;VE][1] (of course),
 * [ZeroBrane Studio][9] (awesome IDE with integrated debugger, please support the developer),
 * [Sublime Text 3][10] (great editor).

As for the sounds, I'll be using [bfxr][11]-generated waveforms. Also, I'm thinking about using online available sample whenever needed (e.g. [Freesound][2]).

Also, I was thinking from time to time to the first game "theme". I'm forcing myself not to work on it too much, since I don't want to spoil the fun once the new year kicks in, but some ideas have been tracked down.

A really interesting aspect I've been thinking a lot is the **camera** implementation. There are [so many ways][4] to implement a view-camera and spotting the best one for the game can really make a huge difference in terms of game-play and user-satisfaction and experience. I'm not going to choose an over-complicated (albeit cool) camera type if the game doesn't call for it. Every feature of the games I'll be implementing won't be *unnecessarily complicated* (for example, I always loved [Defender][3]'s camera implementation, but as a little kid it was a bit of challenge from me getting used to it and I failed miserably in advancing in the game very much).

Also, scrolling (which in fact is intimately close to the camera) is a interesting subject. Back in the C64's and Amiga's days a lot of clever tricks were needed to achieve fluid scrolling with such a tiny amount of memory and CPU power available (luckily those computers were provided with custom chipsets that made those tricks possible, like addressable screen memory and/or coprocessor and/or hardware registers and/or bitplane pointers).

In my first game I surely will be using scrolling and camera. But I'm going to keep it [simple][5].

To be honest, one of the greatest difficulties I'm struggling with is keeping the general "hype" low and not changing my plans every day or so. Every time I read a post on twitter or anywhere on the net some new ideas pop up like as a matter of chain reaction. I'm forcing myself to write down those ideas for a later use, although my first impulse is to put aside my current project and begin elaborating a new one.

Nope.

Remember, one game a month.

Keep yourself focused.

Good luck (to everybody)!


  [1]: http://love2d.org "L&Ouml;VE"
  [2]: https://freesound.org "Freesound.org"
  [3]: https://en.wikipedia.org/wiki/Defender_(1981_video_game) "Defender"
  [4]: http://gamasutra.com/blogs/ItayKeren/20150511/243083/Scroll_Back_The_Theory_and_Practice_of_Cameras_in_SideScrollers.php "Camera"
  [5]: http://divillysausages.com/2013/12/07/colony-tech-update-1-creating-a-game-camera-part-1/ "Creating a Game Camera (Part 1)"
  [6]: http://www.piskelapp.com "PiskelApp"
  [7]: http://pyxeledit.com "Pyxel Edit"
  [8]: http://www.mapeditor.org "Tiled"
  [9]: http://studio.zerobrane.com "Zerobrane Studio"
  [10]: http://www.sublimetext.com "Sublime Text"
  [11]: http://www.bfxr.net "bfxr"