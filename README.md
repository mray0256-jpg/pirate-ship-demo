# pirate-ship-demo
A short water simulation that served as a lesson in graphics and shaders

## Overview
- I created this project throughout September and October 2025 with the goal of learning the basics of graphics, shaders, HLSL, and some interesting math along the way.

## Tech Stack
- Engine: Unity
- Language: C#, HLSL

## Project Setup
This repository contains the scripts from the Unity files to be run locally. It does not include exe and build files to reduce bloating.

## What I learned
- This project was my first Unity project. I started it with the intention of making a two-week long basic pirate ship game, in which a pirate ship travels around shooting cannonballs at sea monsters and plundering the seas. However, my ambitions quickly grew out of proportion, as they are wont to do. Since the art style was simple pixel art, I wanted to try and tackle a gripe I had with other pixel art top-down water: faking it. I wanted water that dynamically changed, and had forces which would act on the player. As you'll soon discover, this went through a couple iterations.
  - Attempt one: My first try was making a grid of cells. If one cell grew in magnitude, the cells around it would also increase. All cells had a dampening factor and there was a "wave" equation that would pass through all cells, repeating over time. However, this wave propogation never quite worked, and trying to visualize a thousand cells with no knowledge of anything GPU failed. This first attempt quickly grew too laggy and I realized I needed to up my game.
  - Attemp two: I stuck with the same system, but tried something else: instead of the cells belonging to world position, they followed the player around. If the player moved, the values inside the cells moved dynamically too. Again, conceptually I don't think this was a terrible idea, I simply did not know enough code to achieve it satisfyingly. If I were to redo either of these, I'd use hashsets, dictionaries, compute shaders, and shadergraph.
  - Attempt three: This time, I completely started from scratch. I switched from true 2D to 2.5D, in which the world was composed of 3D models but the camera was top down and the resolution scaled dramatically down. I modeled a basic pirate ship in blender, then imported it into Unity and got to work. After switching to designing in 3D, I dedicated this project to be an intro to graphics... and I found myself quickly overwhelmed. The first thing I did was learn shadergraph. This seemed the most simple of the nice-to-know graphics features. However, I quickly fell into a trap: relying on tutorials. Since my previous attempts were unsuccessful, I tried using more resources. This included YouTube, StackOverflow, Reddit, and AI. Most of the sources I found were way beyond my scope. I didn't fully absorb tutorials, and was soon walking out of finished scripts or sub graphs feeling even *less* sure than before. This led to complications quite quickly; before long I had problems with underlying systems that I didn't know how to change. I stuck with what I had, though, because it wasn't terrible. Not quality, but not an eyesore. I decided to wrap up the project, ending it by writing a few more features that were entirely my own. The first was a buouyancy system for the pirate ship. This turned out to be mathematically challenging, as you cannot extract height from an x, z coordinate directly while using Gerstner waves. Gerstner waves displace vertices in all three axes, which means that to get a precise point there has to be a degree of approximation. The other system I wrote was a splashing function. This adds waves that propogate outward as though from a splash. They can interfere with the basic Gerstner waves, both amplifying and dampening. I attached this HLSL script to my ship, so that as it traversed the seas a dynamic wake would be created from the ship's stern. I wrote in HLSL and made a custom node function, as well as some scripting in C#. Walking out of this project, this was the feature I was most proud of.
  - **How it's done**
    - This project was primarily math (for the waves, buouy script, & ship movement) and graphics. I'll start with the math.

    1.) Gerstner Wave Equations:
    $\ x = \alpha - \sum_{M}^{m = 1}\frac{k_{x,m}}{k_m}\frac{a_m}{\tanh(k_mh)}sin(\theta_m) $
    $\ y = \sum_{M}^{m = 1}a_mcos(\theta_m) $
    $\ z = \beta - \sum_{M}^{m = 1}\frac{k_{z,m}}{k_m}\frac{a_m}{\tanh(k_mh)}sin(\theta_m) $

    where

    $\ \theta = k_{x,m}\alpha + k_{z,m}\beta - \omega_mt-\phi_m $
    $\ \omega_m = \sqrt{gk_m\tanh(k_mh)} $

    m = Wave Component
    $\ \alpha $ = Vertex Position.x
    $\ \beta $ = Vertex Position.z
    $\ k_m $ = Wave Direction
    $\ a_m $ = Amplitude
    h = Depth

    ![Trochoidal_wave](https://github.com/user-attachments/assets/0c171173-b8d6-4a05-99e4-0bbe8cf903f0)<br>
    *Figure 1: Trochoidal (Gerstner) Waves*

    These equations were plugged into shader graph, and make a decently convincing set of waves. To spruce up our ocean, we add multiple Gerstner waves, with different variables. Playing with them is quite fun, but it never truly makes a convincing sea; for that, we would need many, many waves. There are two solutions to this. One: we switch to fast Fourier transform, or two: add normal maps and textures. I went with the second option out of ease, but I intend to learn Fourier transforms for Unity at some point (A project idea is decoding the sound waves of various instruments to create basic synths!). The waves are now decently convincing, but the ship still isn't. The next thing I added was a buouyancy feature. For this to work, I had to approximate the height as described earlier. 
    
## Future Improvements
- I plan to add some of the features of this simulation to my procedural planet generator, so I can make dynamic gravity with planets' water. When I do that, however, those features will be changed fairly significantly. I would also like to note that the procedural planet project was an answer to the problems from this project; it's a modular project so any underlying problems *don't* effect every other system directly, whereas with the water system here changing something in a fundamental script with introduce problems with later scripts. If you would like to reach out, please email me at mray0256@gmail.com. Thanks!


## Sources & Inspirations
- Ship Buouyancy: [Tom Weiland on YouTube](https://www.youtube.com/watch?v=eL_zHQEju8s)
- Pirate Game Inspiration: [V.A. on youTube](https://www.youtube.com/watch?v=uwU9ZaQ9PhY)
- Gerstner Waves: [Zicore on YouTube](https://www.youtube.com/watch?v=Awd1hRpLSoI)
- Pirate Ship Model: [PolyRem on YouTube](https://www.youtube.com/watch?v=MoqXP7tpL1o&t=500s)
