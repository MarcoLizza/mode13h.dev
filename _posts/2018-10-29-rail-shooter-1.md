---
title: rail-shooter #1
layout: post
permalink: /rail-shooter-1/
categories: 
  - devlog
  - rail-shooter
tags: 
  - retrogaming
  - game-design
  - ideas
---
Recently I thought on developing a rail/gallery shooter and today I was investigating on some of the *classics* of the past in order to study and define reference a baseline for the game mechanics.

Those games were notable first and foremost for their innovative (for the time) input method: a plastic *gun* replica caller [NES Zapper](https://en.wikipedia.org/wiki/NES_Zapper).

I started with one the very first games of the genre, by watching a gameplay view of *Nintendo*'s [Duck Hunt](https://en.wikipedia.org/wiki/Duck_Hunt) on the *NES*. Despite the fact I never owned a NES in my life (and never played the game first-hand) I'm familiar with it. However, watching a video of it is a pleasing refresh since the last time I checked the game was something like 30 years ago. *Wild Gunman* and *Hogan's Alley* were another two games of the period and were very similar.

While watching the video I noticed that, when the gun trigger is pressed, the screen flashes black-and-white for a tiny fraction of a second. That was something I didn't remember! I always assumed the gun was detecting the position tracing the raster-beam (in a similar way the C64's light-pen worked, a method called *cathode ray timing*... it's amazing that I still remember this from the vague albeit precise explanation that my father did to me when I was 8 y/o), but on a second thought it's pretty evident this method won't work as it requires the sensor to be very close to the screen (or feature an expensive magnifying scope to cope for the player distance).

Also, if the gun was detecting a position on the screen, some more steps are required to test for a successful shot:

* detect and store the position the gun is pointing to,
* scan the object list, and
* test which object's hit-box contains the aiming position.

Despite being this easy to implement, on general terms, we need to keep in mind that back in the early '80s game systems weren't very powerful and quick-and-dirty approximations most of the times were preferred. The way the *NES Zapper* works is an example of this.

> It turns out that in the [original patent](https://patents.google.com/patent/US4813682A/en), the *NES Zapper* is described to be able to detect the exact position of the screen, 'though.

With a "screen-flashing" technique the game code doesn't need any specific information on where the gun is pointing. Each time the trigger is pressed, the screen is turned completely black for a frame (to calibrate and put a baseline reference signal point for later comparison), followed by a black screen with a white box where the duck is located (if two ducks are displayed, their hit-boxes are displayed one for each successive frames). The gun needs only to test if the photo-diode detects a light shift in its active scanning/view region (and according to which frame lights the sensor determine which duck has been shot).

> The flashes are one frame long and, thanks to the '80s CRT technical features, invisible to most of the users since the "black-and-white" frames are always alternated with the game screen. Anyhow, they are noticeable from time to time, especially if one looks close enough and when two objects are to be detected. For this reason, this technique isn't of any practical use in a multiple players scenario (albeit *Duck Hunt* being a two player game where the second player moved the duck).

It's worth noticing the first black screen is an elegant solution to disable "cheaters" that could think of pointing the gun to a sufficiently strong light source in order to "fake" the target detection (w/o any change from the reference signal). If that's the case, it simply won't work, because the system doesn't detect any change in signal from the baseline. Thanks to its simplicity could potentially work on modern plasma/LCD/LED monitors... but it doesn't due to latency in modern displays that screws the timings up.

I won't be emulating the light-gun behaviour, but I think that implementing a (very basic) *Duck Hunt* clone is a perfect starting point.
