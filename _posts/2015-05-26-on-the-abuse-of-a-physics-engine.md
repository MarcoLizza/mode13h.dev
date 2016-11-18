---
title: On the Abuse of a Physics Engine
author: marco.lizza
layout: post
permalink: /on-the-abuse-of-a-physics-engine/
thumbnail: balance-scale
categories:
  - rants
tags:
  - development
  - engine
  - retrogaming
  - tools
---
Do one need a full-fledged <a href="http://en.wikipedia.org/wiki/Physics_engine" target="_blank">physics engine</a> to implement a game?

Based on the current nowadays trend it definitely seems so.

However, back in the very beginning of the gaming era, things were different. Almost every game struggled with some of the aspects of dynamic and kinematic.

Racing-cars and space-ships required inertia, platform games&#8217; characters needed gravity and friction, pinballs a combination of all the above...

... oh, and we shall not forget collision detection!

None of those games featured a physics engine in the exact terms we are now used to talk about. CPUs were just too tiny and slow that the bold programmers ended in just *emulating* the behaviours they needed: things needed not to be realistic, they just needed to *seem* that!

> Two major exceptions were <a href="http://en.wikipedia.org/wiki/Thrust_(video_game)" target="_blank">Thrust</a> and <a href="http://en.wikipedia.org/wiki/Exile_%281988_video_game%29" target="_blank">Exile</a>, but they still weren't proper simulations due to hardware limits.

I fear that physics simulation is nowadays overused (and abused) just to be *cool*, with the resulting games resembling each other way too much.

If I were developing a Capcom-like side-scrolling fighting-game, I surely won&#8217;t use a <a href="http://box2d.org/" target="_blank">Box2D</a>, <a href="http://chipmunk-physics.net/" target="_blank">Chipmunk</a> or any other tool. Gravity and inertia are to be simply emulated, and this can be achieved quite easily. Collision detection can be simply performed with some basic math and/or pixel-overlap test. Deflection and bounces are to be approximated and in most case even absent.

Adopting a *realistic* physics engine would just make things harder to achieve. Let&#8217;s think about the mechanics of and old-school ladder in a platform game, or the the (fake) scene depth (that would just end in a plain implementation mess). Struggling with numerical stability and wasting cycles to prevent tunnelling effect when the all you need is integer/fixed-point representation and sprites just move some pixels per frame is inappropriate.

This doesn&#8217;t mean I won&#8217;t use any physics engine at all.

In fact although not directly needed in the current engine I&#8217;m working on, I&#8217;m experimenting with some of them since it&#8217;s something I will possibly need in the very next future.

I just think that one should use a hammer when a hammer is needed. Any other tool is inefficient, a waste, or just plain wrong.

Not to mention that the way physics was approximated in the glorious old days gave them that special flavour we all love and appreciate!