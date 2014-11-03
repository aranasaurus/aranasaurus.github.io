---
layout: post
title: "Thoughts on Engine Design and Implementation"
date: 2014-10-25T19:21:16-07:00
tags: game-dev, SDL
request_feedback: true
---

Today I wanted to get it figured out where the line will be drawn as to which logic is written in C/C++ and what
gets done in Lua.

<!-- more -->

I started off with reading the relevant chapters in [Programming in Lua][pil]. From that I basically came to the
conclusion that the differences between making C functions callable from Lua vs executing Lua scripts from within
C are pretty negligible. So it becomes a matter of preference: Do I want the engine to be rooted in C/C++ with the
ability to delegate game logic implementation off to Lua? Or do I want the engine (and game logic) to be rooted in
Lua and the only C/C++ code being there to expose SDL, OpenGL, and any other C libraries I end up using for physics,
audio, animation, etc to Lua?

Let's examine these two options in a bit more detail.

# Engine in C/C++

Doing the engine in C will of course mean more C code needing to be written by me, which I don't particularly
enjoy. However, it'll also mean that the hard part of the game(s) will be a bit more straight forward, as there
will only be one level of abstraction, and it'll be more cleanly separated from the game logic. Basically meaning
I'll be writing more C/C++ at first, but later on, when I'm doing mostly game-specific implementation, I'll be in
Lua land.

Let's take a look at how this might be modeled.

### C/C++ Responsibilities

  * Set up window and OpenGL Context.
  * Manage game loops.
  * Maintains a list of game objects.
  * Render loop uses properties on game objects to determine how to draw them.
  * Update loop calls an update function on game objects.
  * Handle input and pass it off to game objects that care about it.

### Lua Responsibilities
    
  * Game objects are defined as Lua tables with attributes for location and dimensions and possibly a texture name.
  * Game objects have an update method which allows for updating their attributes every tick.

Basically this set up boils down to the plumbing and architecture stuff being done in C/C++ and then there being a
communication layer between that scaffolding and all of the game objects, which are implemented in Lua.

After writing this all out I actually like this method a bit more than I did before doing so. Let's see what that
process does on the other side.

# Engine in Lua

Doing the engine in Lua has the upside of everything being done in Lua and there being no extra layer between the
engine and entities. However, it does involve figuring out a way to get everything, everywhere bound to Lua, which
probably wouldn't be too hard, but it just feels kinda dirty and like it will actually hinder dev speed a little
because it'll be making the hard part of this project a bit more complicated without a lot of payoff.

Anyway let's map it out.

### C/C++ Responsibilities

  * Glue code which exposes SDL and OpenGL calls to lua.

### Lua Responsibilities

  * Everything.

This was less helpful than the C/C++ analysis, but I think I've come to my decision at any rate.

# Conclusion

Looks like I'll be doing an engine based in C/C++ (which is another decision I'll need to make very soon) with
game objects/entities implemented in Lua. Hopefully I'll be able to get started on actually implementing some
functionality along these lines tonight after the kids go to bed and/or tomorrow!


[pil]: http://www.lua.org/pil "Programming in Lua, AKA The Lua Bible"

