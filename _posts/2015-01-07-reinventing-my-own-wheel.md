---
title: Reinventing my own wheel
author: marco.lizza
layout: post
permalink: /reinventing-my-own-wheel/
categories:
  - rants
tags:
  - development
  - engine
---
I totally agree with Ron Gilbert in his <a href="http://blog.thimbleweedpark.com/engine" target="_blank">recent Thimbleweed blog post</a>.

Upon starting [my latest project][1], I was pondering whether I should go and use an already existing engine or write a new one from scratch.

Several good scripted game engines are available on the net, such as <a title="LÖVE" href="http://love2d.org" target="_blank">LÖVE</a> (which is also open-source) and <a title="Unity3D" href="http://unity3d.com" target="_blank">Unity3D</a>. But...

... while peeking and messing others' code is a pivotal and funny way to learn new methods/ways/point-of-views, I always end in tailoring my own suit.

I'm just too exacting to find an existing engine that suits my needs. Moreover, whilst full-featured, those engines are mostly general purpose. They do feature things I simply don't need and they lack what I want (such as the scripting-side expressive power I seek).

*( I simply won't implement my path-finding algorithm with a scripted language, no matter its performances... the engine should abstract these kind of functionalities, not the script implement it )*

I'll just end in spending weeks tweaking the engine/framework at the point that it would be easier to write my own from scratch. In the meanwhile, I won't feel comfortable at all in being so far from the guts of the program.

Plus, starting an engine from scratch is always a good opportunity to try new solutions and technologies!

 [1]: http://blog.brainasylum.com/a-modern-one-please/ "A modern one, please"