---
title: Going Mobile
layout: post
permalink: /going-mobile/
categories:
  - tofu-engine
  - devlog
tags:
  - platforms
---
The other day I came across the [Anbernic RG35XX+](https://anbernic.com/en-it/products/rg35xx-plus), which I have to admit I had never heard of before (I'm not really into retro gaming, to be honest).

I've always been curious about mobile gaming... and I'm not talking about the kind that's all the rage these days with the ubiquitous use of smartphones. I'm talking about the old-fashioned kind, where the gamer picks up a real console with the sole purpose of entertaining him/her.

The machine has more than its fair share of features: a quad-core 1.5GHz ARM 64-bit CPU, paired with an OpenGL/ES capable dual-core GPU, running on a 3.5-inch 640x480 display. Directional pad, buttons, and a Linux-derived operating system complete the picture.

What if I want to run Tofu Engine on it?

Oh... I need to buy one of those thingamabobs, first... and done! While waiting for it to arrive, I can tackle the first aspect: cross-compile to ARM.

From the beginning, I worked with the idea that Linux should be the one and only development/build environment for **Tofu Engine**. With little or no effort (thanks to *GCC* and *MinGW*) I managed to get both the *Linux* and *Windows* builds on the same development machine (or docker container). I have also tested the (interactive) build on a *Raspberry Pi* device and it executes flawlessly; it required a *Pi4* device for this purpose, however.

I was aware that GCC is more than capable of cross-compiling to ARM. All that was needed was the "recipe" to make it work. In a word, **multiarch**. Taking advantage of this feature, I installed the `arm64` and `armhf` (for the 64-bit and 32-bit ARM builds respectively) library dependencies and build tools... and with a bit of Makefile-fu... ta-da!

This also proved to be a good opportunity to reorganise the Makefile and unify the Raspberry Pi build with the RG55XX one (as they are both ARM based)... and lo and behold, it works on both Raspberry Pi 4 and 5! :D

Having reached this point, however, another problem becomes apparent to me: would the currently used OpenGL version (which is fixed at version 2.1 for simplicity's sake) work on the RG35XX+ w/ GarlicOS.

According to the specifications of the [Allwinner H700 chipset](https://www.allwinnertech.com/uploads/pdf/2021070513595227.pdf) (the beating heart of the RG35XX+), the [ARM Mali G31 GPU](https://developer.arm.com/Processors/Mali-G31) supports OpenGL ES 3.2 (and Vulkan 1.1, but I will not *even* explore this unknown territory for the moment).

I admit that the difference between OpenGL and OpenGL/ES is only clear to me from a conceptual point of view, so my current goal is to study the two variants closely and (hopefully) find a common point of contact.

It is also an opportunity to review and improve the integration with OpenGL in general, in anticipation of a (possible) future move to more intensive use of the GPU.

See you soon. :)
