---
title: Back on Track
layout: post
permalink: /back-on-track/
categories:
  - tofu-engine
  - devlog
tags:
  - libraries
---
The end of the year is just around the corner...

... and we are actually a little behind on devlogs. Once again. As usual.

I wish I could say that I've neglected the blog because I've been concentrating so much on **Tofu Engine** that I couldn't find the time to do it, but the harsh reality is different. That is, I have had other projects/commitments that sucked up my time and energy.

Nevertheless, I can see a glimmer of light on the horizon, having hopes (as well as a very strong urge and determination) to be able to devote some time and get my *game engine* back on track.

In the past days I resumed working on it. Not with the lengthy full-night sessions I was accustomed with, but it's still something.

I tackled down what I do periodically to "jumpstart" my activities on the project, that is I have updated the engine's *dependencies*. As I have said and reiterated several times, **Tofu Engine** was born with the idea of having (almost) no *run-time* dependencies. However, it does use other libraries (and it should) which I take care to keep as up-to-date as possible. In fact, if there's one thing I can't stand, it's projects where dependencies are neglected... not keeping them up-to-date, in the long run, can cause quite a few problems. Please, fellows developers, keep your dependencies up to date! =D

I also took the opportunity to ponder on whether the libraries I had chosen at the time are still the ideal option. I must say that, in principle, I would make the same choice again even today. I do, however, want to make a few remarks on some of them.

For a short time, I feared that [GLFW](https://www.glfw.org/) was about to fade away as a project. From both the website and the main channels, there were no updates and development seemed to be halted. A few pull-requests from contributors were crowding in, but without being integrated into the main code. Thankfully, however, the situation now seems to have improved and work is continuing (I must say also consistently, because it is rapidly approaching the long-awaited new release). In any case, for use in the game-engine what is now possible remains more than sufficient (with the exception of the input source management part, where I would particularly like to see support for *force-feedback* and *gamepad rumble* integrated).

In a similar vein, [Chipmunk2D](https://chipmunk-physics.net/) is in doubt as to whether or not to maintain it in the long term. Developments are at a standstill, considering that the library is stable and well established. However, [Eric Catto](https://github.com/erincatto), author of [box2d](https://box2d.org/), has been working for some time on [a C99-compliant version](https://github.com/erincatto/box2c) of his famous middleware (which is currently written in C++). I plan to try to integrate it very soon.

Other than that, the first and most immediate objective will be to make the product (finally?) usable by the community. The official documentation absolutely must be completed (in many ways... from technical description, to examples/tutorials, to API documentation)! :P To this end, I intend to elaborate a bit on the project "presentation" so as to make request (veiled or not) for help from external contributors.

Obviously, then, there are features to be added (quite a few) that I have been taking notes on over the past few months... and others that could potentially be refined, simplified or even (why not) eliminated because they are overabundant.

On the sidelines, I want to revisit some of the more intimate details of the engine. Without radically revamping the API (hopefully without modifying it in any way) I intend to rework the rendering sub-system, in order to optimize it and make it more sensitive toward power consumption.

In short, there is a lot of work to be done. :D

Let's get started!
