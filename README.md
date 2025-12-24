# pirate-ship-demo
A short water simulation that served as a lesson in graphics and shaders

## Screenshots
- ...

## Overview
- I created this project throughout September and October 2025 with the goal of learning the basics of graphics, shaders, HLSL, and some interesting math along the way.

## Tech Stack
- Engine: Unity
- Language: C#, HLSL

## Project Setup
This repository contains the scripts from the Unity files to be run locally. It does not include exe and build files to reduce bloating.

## What I learned
- This project was my first Unity project. I started it with the intention of making a two-week long basic pirate ship game, in which a pirate ship travels around shooting cannonballs at sea monsters and plundering the seas. However, my ambitions quickly grew out of proportions, as they are wont to do. Since the art style was a simple pixel art, I wanted to try and tackle a gripe I had with other pixel art top-down water: faking it. I wanted water that dynamically changed, and had forces which would act on the player. As you'll soon discover, this went through a couple iterations.
  - Attempt one: My first try was making a grid of cells. If one cell grew in magnitude, the cells around it would also increase. All cells had a dampening factor and there was a "waves" equation that would pass through all cells, repeating over time. However, this wave propogation never quite worked, and trying to visualize a thousand cells with no knowledge of anything GPU failed. This first attempt quickly grew too laggy and I realized I needed to up my game.
  - Attemp two: I stuck with the same system, but tried something else: instead of the cells belonging to world position, they followed the player around. If the player moved, the values inside the cells moved dynamically too. Again, conceptually I don't think this was a terrible idea, I simply did not know enough code to achieve it satisfyingly. If I were to redo either of these, I'd use hashsets, dictionaries, compute shaders, and shadergraph.
  - Attempt three: This time, I completely started from scratch. I switched from true 2D to 2.5D, in which the world was composed of 3D models but the camera was top down and the resolution scaled dramatically down. I modeled a basic pirate ship in blender, then imported it into Unity and got to work. I knew know that this project would be an intro to graphics... and I was overwhelmed. The first thing I did was learn shadergraph. This seemed the most simple of the nice-to-know graphics features. However, I quickly fell into a trap: relying on tutorials. Since my previous attempts were unsuccessful, I tried using more resources. This included YouTube, StackOverflow, Reddit, and AI. Most of the sources I found were way beyond my scope. Instead of absorbing a tutorial, I rote copied a few. This led to complications quite quickly; before long I had problems with underlying systems that I didn't know how to change. I stuck with what I had, though, because it wasn't terrible. Not quality, but not an eyesore. I ended the project by writing a few more features that were entirely my own. The first was a buouyancy system for the pirate ship. This turned out to be mathematically challenging, as you cannot extract height of an x, z coordinate directly from Gerstner waves. Gerstner waves displace vertices in all three axes, which means that to get a precise point there has to be a degree of approximation. To do this, the initial point is chosen as a guess. ... The other system I wrote was a splashing function. This adds waves that propogate outward as though from a splash. They can interfere with the basic Gerstner waves, both amplifying and dampening. I attached this HLSL script to my ship, so that as it traversed the seas a dynamic wake would be created from the ship's stern.



## Future Improvements
- I plan to add some of the features of this simulation to my procedural planet generator, so I can make dynamic gravity with planets' water. When I do that, however, those features will be changed fairly significantly. 


## Sources
- ...
