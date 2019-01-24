---
title: 'StarField (Analysis)'
author: marco.lizza
layout: post
permalink: "/starfield-analysis/"
comments: true
categories: 
  - analysis
tags: 
  - 1gam
  - onegameamonth
  - gamedev
  - indiegame
published: false
---

One of my goals for the #1GAM challenge is to experiment different mechanics and game-type every month. Due to time constraint, it has occurred to me that deploy a full-and-complete game in a month is nearly impossibile. However, I working and refinable prototype is a more realistic target. Considered that I've chosen (LOVE)[http://love2d.org] as the base framework, at each month iteration I'm adding and improving with features the codebase.

In the previous month (an article on the subject is under construction) I've enriched the entity managament module. However, it was still very basic. The player, the list of foes, and the game objects (keys, doors, flares and such) where kept in separate variables. This wasn't going to be very scalable at all. So, for the March iteration I planned to extend it in order to be more generic and handle collisions, too. Also I wanted to implement some kind of particle effets, add sound playing, and spice a bit the camera handling.

As for the game itself, I found that a (static) space-shooter could fit the theme. The player would stay at the center of the *arena*, rotating and shooting. Much like (Asteroids)[http://].

( at the beginning I had the idea to develop something similar to (Atari Combat)[https://en.wikipedia.org/wiki/Combat_(1977_video_game)]. Oh, my! The "tank game" was so much fun desplite that weird collision detection routines! The "tank pong" level and the controllable projectiles! Perhaps I will reuse this "idea" for a future game )

To some degeee, I did something that was quite frequent back in the old days of the #gamedev, that is the game is developed and the game was pretty much a "cloak" for some ground-breaking technical innovation or feature ((Shadow of the Beast)[] come immediately to mind, technically-wise marvellous but the gameplay was lame to the least).

## Entities

When working with dynamic languages such as Lua, polymorphism and late-binding are much more easier to exploit. That is, if an *object* has a method "m()" you can call it, irregardless of the object heritage. There are not strict type checks that guides (forces?) you to define precise hierarchies.

( granted that in Lua, the term *object* is pretty much a programmer abstraction and interpretation more that a language feature )

However, this freedom could lead to a lot of duplicate code. With regard to this, the object-oriented way to abstract and collect commong functionalities in base classes is useful. When designing the entities managament system I was facing this issue. I already implemented classes in Lua in the most straightforward a simple way possible (see ...) for my previous projects, since I really find pointless to "slow down" the language by introducing an overhead to sophysticately implement something that the language natively lacks. So, on this league, I needed a simple a fast solution. After some thought I ended in implementing this piece of code:

```
function prototype(base)
  local proto = {}
  -- If a base class is defined, the copy all the fiels (mostly functions)
  -- and store a reference to the base class. This is an instant snapshot,
  -- any new field defined runtime in the base class won't be visible in the
  -- derived class.
  if base then
    for key, value in pairs(base) do
      proto[key] = value
    end
    proto.__base = base
  end
  -- This is the standard way in Lua to implement classes.
  proto.__index = proto
  proto.new = function()
      local self = setmetatable({}, proto)
      return self
    end
  -- Provide a help function to access the base reference.
  proto.base = function(self)
      return self.__base
    end
  return proto
end
```

That is a prototype-cloning function that, given a optional reference module (which implements a class, in some way), creates a prototype. The prototype is a special kind of module that can be used to create instances of the same class. The interesting thing is that any *base* function/property is cloned (we shold avoid, or deep-clone, tables). Also a special `base()` method is created to access the reference module functions and implement overloading where needed.

Pro: fast via base module copying, basic inheritance
Cons: it's not a full-fledged solution

## Movements and Collisions

Each and every entity can be described with a table such as the following:
```
local entity = {
  position = { 2, 5 },
  angle = math.pi, -- in radians
  speed = 5 -- in pixels per second
  radius = 3,
}
```
At every update step, the current position is updated by moving along the direction indicated by `angle`. This has been implemented in the `Entity:cast()` method, that is similar to this
```
-- [dt] is the amount of seconds since the last update
local distance = entity.speed * dt
local vx, vy = math.cos(entity.angle) * distance, math.sin(entity.angle) * distance
local cx, cy = unpack(entity.position)
entity.position = { cx + vx, cy + vy }
-- Update also entity life, if needed.
```
In the `Entities:update()` method
* the entities list is scanned, updating each entity position (and any other attribute, such as the life for the autoremoved ones)
* the "dead" entities (whose life has dropped to zero some of them have limited life and disappears with time, others' life is set to zero when destroyed) are removed
* add any incoming "new" entity. Since when updating an entitity, new entities could be spawned (e.g. bullets), we keep a separate "incoming" entities list which is merged outside the entities updating loop

Once every entity has be moved, we check for the collision, building a list of overlapping entities with a naive `O(n^2)` algorithm. Now, the `radius` property is very helpful. We both use it to draw the entities (that, for simplicity, appears as plain circles) and to check for collision.
```
-- An *axis-aligned bounding box" could also be used, however.
function overlaps(a, b)
  local ax, ay = unpack(a.position)
  local bx, by = unpack(b.position)
  -- As an optimization, we can store the squared-radius for each entity, and save for the square-root calculation when computing the distance.
  local dx, dy = ax - bx, ay - by
  local distance = math.sqrt(dx * dx + dy * dy)
  return distance <= (a.radius + b.radius)
end

function resolve_collisions(entities)
  local colliding = {}
  for _, this in ipairs(entities) do
      for _, that in ipairs(entities) do
        -- We avoid testing an instance with itself
        if this ~= that and overlaps(this, that) then
          colliding[#colliding + 1] = { this, that }
        end
    end
  end
  return colliding
end
```
The game implementation, however, is a bit more complicated. Some entities are defined as "ephemeral", that is does not collides with anything (e.g. sparkles, clouds, debris, etc...), and they are skipped for collision check. Also, some entities (such as the floating red/white numbers indicating the points and the damage) does not have the `radius` property and are skipped as well.

The colliding entities list is then scanned and, according to the relatve collision type (player with enemy-bullet, enemy with player or player-bullet) an appropriate action is perfomed (spawn sparkles, destroy entity, play sounds, etc...).

> Since the entities moves relatively slow and the update delta-time is small, we will unlikely experiment (tunnel effect)[http://fremycompany.com/BG/2012/Understanding-the-Tunnel-Effect-with-intuition-only-910/]. However, if the FPS drops either due to a clogged system or anything else, this is going to be a problem. For that reason, the collision resolution routine should implement a "prediction" algorithm. That is, it should check if give the current delta-time, any entity pair will "cross" their paths and, in the case, find the intersection point. This is left for future enhancemente of the core framework.

## Particles

To spice up a little the game experience it's very common to spawn particles in several cases. For example, while a player walks in a wet floor water little tiny drops can be spawned to simulate the foot splashing in the water. Or, in a car driving game, when the player abruptly steer the vehicle small clouds should appear due to wheels' friction with the road.

In the game I decided that particles should be spawned when a bullet collides with something (sparkles) and when the player or a foe is destroyes (debris and clouds). Also, small text messages should also be spawned as a particular kind of particle do tell the used the amount of damage taken/delivered and the score points got after destroying an enemy.

Differentiating the particles groups is quite straightforward.

Sparkles move very quickly. They are drawn as little line segments (with length proportional to the sparkle speed) and change in color during the time (going from white to yellow, then orange, and brown). In the while they also gracefully fade out. When a bullet collides it is remove and is replaced by a random bunch of sparkles with random direction, speed and lifespan.

Debris clouds, on the contrary, moves very slowly. The are drawn as (random colored) greish circles that shrink over time. When the player or the enemy is destroyed it is removed and replaced by a "blossom" of clouds with random color, speed, and lifespan. The direction is nor random, since the are equally distributed along 360 degrees by discrete steps.

The text messages are quite interesting. They consist of a single particle drawn as fixed color text that fades out over time, moving along a random direction. The speed and the size of the particle depend on the "amount" it describes (bigger scores are bigger in size a move faster and farther). My original idea was to mimic the "bonus points" caption that a lot of games displayed in the Eighties.

## Camera 

This game project marks also the first steps in developing a camera management module. So far, in the previous projects, I always opted for a static viewport. Also this game features a static non-scrolling viewport...

... but I made it a bit more interesting by adding *camera shaking* that occurs when

## Parallax

## Sound

This was also the occasion for implementing a basic sound-manager. Not very fancy, 

## Model

## Conclusions

That's all for March 2016 game.

The development took only 2 weeks, which is pretty good. The game is simple but, with the current technology, I always feel like I'm "wasting" resources. All it becomes *too* easy to implment, at the expense of resource usage (CPU and memory)- We really do over-elaborate things, nowadays.

You can found the full source-code (here)[https://github.com/MarcoLizza/starfield]. You are welcome to have a peek of the project. If have any question and/or suggestion don't hesitate to contact me!

See ya!


----

parallax
particles effects, not using the engine API. Pretty versatile, with minor adaptation it handles from sparkes to bubbles.
sound, simple framework
OOP, but in form of prototype-mixins
camera shaking, next one scroller?
simple geometric shapes
collision, simple but with dynamic updates
