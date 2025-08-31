---
title: On the Abuse of a Physics Engine
layout: post
permalink: /on-the-abuse-of-a-physics-engine/
categories:
  - rants
tags:
  - engine
  - retrogaming
  - tools
---
Do one need a full-fledged [physics engine](http://en.wikipedia.org/wiki/Physics_engine) to implement a game?

Based on the current nowadays trend it definitely seems so.

However, back in the very beginning of the gaming era, things were different. Almost every game struggled with some of the aspects of dynamic and kinematic.

Racing-cars and space-ships required inertia, platform games' characters needed gravity and friction, pinballs a combination of all the above...

... oh, and we shall not forget collision detection!

None of those games featured a physics engine in the exact terms we are now used to talk about. CPUs were just too tiny and slow that the bold programmers ended in just *emulating* the behaviours they needed: things needed not to be realistic, they just needed to *seem* that!

> Two major exceptions were [Thrust](http://en.wikipedia.org/wiki/Thrust_(video_game)) and [Exile](http://en.wikipedia.org/wiki/Exile_%281988_video_game%29), but they still weren't proper simulations due to hardware limits.

I fear that physics simulation is nowadays overused (and abused) just to be *cool*, with the resulting games resembling each other way too much.

If I were developing a Capcom-like side-scrolling fighting-game, I surely won't use a [Box2D](http://box2d.org/), [Chipmunk](http://chipmunk-physics.net/) or any other tool. Gravity and inertia are to be simply emulated, and this can be achieved quite easily. Collision detection can be simply performed with some basic math and/or pixel-overlap test. Deflection and bounces are to be approximated and in most case even absent.

Adopting a *realistic* physics engine would just make things harder to achieve. Let's think about the mechanics of and old-school ladder in a platform game, or the the (fake) scene depth (that would just end in a plain implementation mess). Struggling with numerical stability and wasting cycles to prevent tunnelling effect when the all you need is integer/fixed-point representation and sprites just move some pixels per frame is inappropriate.

This doesn't mean I won't use any physics engine at all.

In fact although not directly needed in the current engine I'm working on, I'm experimenting with some of them since it's something I will possibly need in the very next future.

I just think that one should use a hammer when a hammer is needed. Any other tool is inefficient, a waste, or just plain wrong.

Not to mention that the way physics was approximated in the glorious old days gave them that special flavour we all love and appreciate!