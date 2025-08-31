---
title: Reinventing my own wheel
layout: post
permalink: /reinventing-my-own-wheel/
categories:
  - rants
tags:
  - development
  - engine
---
I totally agree with Ron Gilbert in his [recent Thimbleweed blog post](https://blog.thimbleweedpark.com/engine).

Upon starting [my latest project][1], I was pondering whether I should go and use an already existing engine or write a new one from scratch.

Several good scripted game engines are available on the net, such as [LÖVE](https://love2d.org) (which is also open-source) and [Unity3D](https://unity3d.com). But...

... while peeking and messing others' code is a pivotal and funny way to learn new methods/ways/point-of-views, I always end in tailoring my own suit.

I'm just too exacting to find an existing engine that suits my needs. Moreover, whilst full-featured, those engines are mostly general purpose. They do feature things I simply don't need and they lack what I want (such as the scripting-side expressive power I seek).

*( I simply won't implement my path-finding algorithm with a scripted language, no matter its performances... the engine should abstract these kind of functionalities, not the script implement it )*

I'll just end in spending weeks tweaking the engine/framework at the point that it would be easier to write my own from scratch. In the meanwhile, I won't feel comfortable at all in being so far from the guts of the program.

Plus, starting an engine from scratch is always a good opportunity to try new solutions and technologies!

[1]: /a-modern-one-please/ "A modern one, please"