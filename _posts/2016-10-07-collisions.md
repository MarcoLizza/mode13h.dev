---
title: 'Collisions'
author: marco.lizza
layout: post
permalink: "/collisions/"
comments: true
categories: 
  - rants
tags: 
  - development
  - engine
---
It's been quite a while since the last blog post. Summer has passed, and due to job/family/life/whatever I ended in not working on game development as much as I intended. I procrastinated a while, also indulging in some console games (note for the future myself, I'm beginning to grow tired of current sandbox games... is the genre which has become stale and can be refreshed or is it doomed?). I simply wasn't able to follow the monthly #1GAM jams, as well.

However, I devised some intriguing ideas and worked on my game framework. I enhanced the entity management module (not in the [ECS](https://en.wikipedia.org/wiki/Entity_component_system) sense) I'm evolving over time. For the record, it's purpose is to handle a collection of so-called *entities* (clever and original name, I know) by driving the updating and drawing steps. It also takes care of collision detection, which was the main area of this enhancement.

In the previous iteration I optimized the framework by adopting a grid-based partitioning policy (and ending in implementing a [spatial hash-map](https://conkerjo.wordpress.com/2009/06/13/spatial-hashing-implementation-for-fast-2d-collisions/) technique without prior knowledge of it). Now it's time to tackle the *collision resolution*.

I was fairly reluctant in doing it, since I always strived to keep the development approach as much coherent with the old-days techniques as possible. However, I deemed that this was simply too stubborn from me. Back on the C64/Amiga/Atari/whatever platform, collision detection and resolution were definitely implemented in a simpler fashion than nowadays (due to both slower CPUs and lower level languages use, most notably assembly)... but they were also based upon and cleverly exploited peculiar hardware features of the platforms themselves (e.g. hardware interrupts, co-processors, etc...) to aid the tiny slow central processing unit. For that reason, algorithms were specifically tailored to handle the specific game needs. From time to time this also ended in being faulty in some corner cases resulting in funny (and sometimes famous) glitches.

Today things are outright different. We no longer have such chipsets and their peculiar features to exploit. On the contrary, CPUs are much more powerful (paired with some neat GPUs) and we can implement more sophisticated algorithms that don't trip on these "faults" and can be developed as **reusable modules** (e.g. a collision detection-and-resolution module we can use both in a platform game and shooter).

Will this mean we are going to loose the *vintage genuinity* of the early games?

In my option not necessarily. The result will certainly be more evolved (and we would be crazy in not trying to evolve it), and some will end in an abuse of it, but I think we can achieve a good level of "faithfulness" even by adopting newer algorithms and techniques... just keep in mind that we are coding games to *simulate* real-life behaviours, and not *emulate* them.

*( this is also related to the use and [abuse of physics-engine](/on-the-abuse-of-a-physics-engine/) that plagues the game scene... I'm simply too fed up with them... )*

Much like cartoons do; they are funny and interesting because the approximate and exaggerates reality. Speaking of which, I strongly suggest to adopt some of the [12 principles of animation](https://en.wikipedia.org/wiki/12_basic_principles_of_animation) to spice up your games.

Enough with this. Let's move on in coding some concrete game! :)

*( By the way, I think I devised quite a nice and reusable piece of code to handle the "actors" and I'm happy with it )*