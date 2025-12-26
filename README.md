# pirate-ship-demo
A short water simulation that served as a lesson in graphics and shaders

<img width="2462" height="1234" alt="Screenshot 2025-12-24 202310" src="https://github.com/user-attachments/assets/4b4cbe34-099b-4f05-adc3-5dbbca82c681" /><br>
*Figure 1: Pirate Ship*

## Overview
- I created this project throughout September and October 2025 with the goal of learning the basics of graphics, shaders, HLSL, and some interesting math along the way.

## Tech Stack
- Engine: Unity
- Language: C#, HLSL

## Project Setup
This repository does not contain any scripts from Unity, as any code of interest should be included below.

## What I learned
- This project was my first Unity project. I started it with the intention of making a two-week long basic pirate ship game, in which a pirate ship travels around shooting cannonballs at sea monsters and plundering the seas. However, my ambitions quickly grew out of proportion, as they are wont to do. I wanted to try and tackle a gripe I had with other top-down pixel art water: faking it. I wanted water that dynamically changed and could exert forces on the player. As you'll soon discover, this went through a couple iterations.
  
  - Attempt one: My first try was making a grid of cells. If one cell grew in magnitude, the cells around it would also increase. All cells had a dampening factor and there was a "wave" equation that would pass through all cells, repeating over time. However, this wave propogation never quite worked, and trying to visualize a thousand cells with no knowledge of anything GPU failed. I also kept all the xy cell positions in a 1D array, which wound up being horribly complicated. Even despite naive attempts at pre-optimizing, my first attempt quickly grew too laggy and I realized I needed to up my game.
    
  - Attemp two: I stuck with the same system, but tried something else: instead of the cells belonging to world position, they followed the player around. If the player moved, the values inside the cells moved dynamically too. Again, conceptually I don't think this was a terrible idea, I simply did not know enough to achieve it satisfyingly. If I were to redo either of these, I'd use hashsets, dictionaries, 2D lists/arrays, and all things GPU (compute shaders my beloved).

![OldSplash](https://github.com/user-attachments/assets/19beae01-da6c-458d-9e9b-4ef95cd8a223)
*Figure 2: First attempt at Creating Splashes*

- Before I move onto the final attempt, I figured I would include some of the code used in both of these iterations. This code only renders a border around the player, and forgets all other data. It's used in the gif above. There are numerous issues, but the most glaring is setting a texture2D on the CPU.

```csharp
Vector2 getForce(int indexedPos, float xRatio, float yRatio)
{
    //Calculates the negative direction of the slope using first order forward finite differences
    Vector2 unitForce = new Vector2(0f, 0f);
    unitForce.x = ((heightCurrent[indexedPos + 1] - heightCurrent[indexedPos]) / (CellSize)) * (1 - yRatio)
        + ((heightCurrent[indexedPos + 1 + GridWidth] - heightCurrent[indexedPos + GridWidth]) / (CellSize)) * (yRatio);
    unitForce.y = ((heightCurrent[indexedPos + GridWidth] - heightCurrent[indexedPos]) / (CellSize)) * (1 - xRatio)
        + ((heightCurrent[indexedPos + 1 + GridWidth] - heightCurrent[indexedPos + 1]) / (CellSize)) * (xRatio);
    return -unitForce.normalized;
    ;
}

float findShipHeight(int coord, float xRatio, float yRatio)
{
    //finds the wave amplitude at the ship and scales the force applied from a wave. Yes, it's disgusting...
    return (Mathf.Lerp((Mathf.Lerp(heightCurrent[coord], heightCurrent[coord + 1], xRatio)), (Mathf.Lerp(heightCurrent[coord + GridWidth], heightCurrent[coord + GridWidth + 1], xRatio)), yRatio));
}

private void CalculateHeightMap()
{
    Rigidbody2D rb = ship.GetComponent<Rigidbody2D>();
    Vector2 location = ship.transform.position / CellSize;
    Pos p = new Pos(Mathf.FloorToInt((location.x + GridWidth) / 2), Mathf.FloorToInt((location.y + GridHeight) / 2));
    float pFracX = location.x - p.x;
    float pFracY = location.y - p.y; //use the fractions to bilinearly interpolate the force applied to the ship. NOT used for waves.
    int indexedPos = Mathf.Clamp(toArrayCoord(p), 0, (GridWidth * GridHeight)); //clamp to ensure the index is within the bounds of the array. "Buffer Zone" will cover out of bounds errors.
                                                                                // Debug.Log(indexedPos);
    for (int i = indexedPos - (renderRadius * (GridWidth + 1)); i <= (indexedPos + renderRadius * (GridWidth + 1)); i++)
    {
        //this first line is the most confusing. It generates new heightMap values from the old.
        //Essentially, it takes the four sides adjacent and generates an average with a weighted dampening amount associated with the previous height.
        if (i % GridWidth == 0 || (i + 1) % GridWidth == 0 || i < GridWidth || i > ((GridHeight - 1) * GridWidth))
        {
            continue;
        }
        if ((i % GridWidth) > ((indexedPos % GridWidth) - renderRadius) && (i % GridWidth) < ((indexedPos % GridWidth) + renderRadius))//prevents the code from modifying the sides
        {
            heightNew[i] = Mathf.Clamp01(((2 - WaveDampening) * heightCurrent[i])
                - ((1 - WaveDampening) * heightPrev[i])
                + WaveSpeed * WaveSpeed * Time.fixedDeltaTime * Time.fixedDeltaTime
                * (heightCurrent[i - 1] + heightCurrent[i + 1] + heightCurrent[i + GridWidth] + heightCurrent[i - GridWidth] - (4 * heightCurrent[i])));
            if (heightNew[i] < 0.0001)
            {
                heightNew[i] = 0;
            }
            byte gray = (byte)(64 + (heightCurrent[i] * 191));
            pixels[i] = new Color32(255, gray, 0, 255);
        }
    }
    Water.SetPixels32(pixels);
    Water.Apply(false, false);
    float[] temp = heightPrev;
    heightPrev = heightCurrent;
    heightCurrent = heightNew;
    heightNew = temp;

    rb.AddForce((getForce(indexedPos, pFracX, pFracY)) * WaveIntensity * findShipHeight(indexedPos, pFracX, pFracY), ForceMode2D.Force);
}
```

- Attempt three: This time, I completely started from scratch. I began work on this iteration *after* the Texas Game Jam, which boosted my confidence and motivation. I switched from true 2D to 2.5D, in which the world was composed of 3D models but the camera was top down and the resolution scaled dramatically down. After switching to designing in 3D, I dedicated this project to be an intro to graphics... and I found myself quickly overwhelmed. I fell into a trap: relying on tutorials. Since my previous attempts were unsuccessful, I tried using more resources. This included YouTube, StackOverflow, Reddit, and AI. Most of the sources I found were way beyond my scope. I didn't fully absorb tutorials, and was soon walking out of finished scripts or sub graphs feeling even *less* sure than before. This led to complications quite quickly; before long I had problems with underlying systems that I didn't know how to change. I stuck with what I had, though, because it wasn't terrible. Not exactly presentable, but not an eyesore.

      Due to the fundamental problems, I decided to wrap up the project. I ended it by writing a few more features that were entirely my own. The first was a buouyancy system for the pirate ship. This turned out to be mathematically challenging, as you cannot extract height from an x, z coordinate directly while using Gerstner waves. Gerstner waves displace vertices in all three axes, which means that to get a precise point there has to be a degree of approximation. The other system I wrote was a splashing function. This adds waves that propogate outward as though from a splash. They can interfere with the basic Gerstner waves, both amplifying and dampening. I attached this HLSL script to my ship, so that as it traversed the seas a dynamic wake would be created from the ship's stern. I wrote in HLSL and made a custom node function in shader graph, as well as some scripting in C#. Walking out of this project, this was the feature I was most proud of.
  
  - **How it's done**

    Gerstner Wave Equations:
    
    $\ x = \alpha - \sum_{M}^{m = 1}\frac{k_{x,m}}{k_m}\frac{a_m}{\tanh(k_mh)}sin(\theta_m) $
    
    $\ y = \sum_{M}^{m = 1}a_mcos(\theta_m) $
    
    $\ z = \beta - \sum_{M}^{m = 1}\frac{k_{z,m}}{k_m}\frac{a_m}{\tanh(k_mh)}sin(\theta_m) $

    where

    $\theta = k_{x,m}\alpha + k_{z,m}\beta - \omega_mt-\phi_m $
    
    $\omega_m = \sqrt{gk_m\tanh(k_mh)} $

    m = Wave Component
    
    $\alpha$ = Vertex Position.x
    
    $\beta$ = Vertex Position.z
    
    $k_m$ = Wave Direction
    
    $a_m$ = Amplitude
    
    h = Depth

    ![Trochoidal_wave](https://github.com/user-attachments/assets/0c171173-b8d6-4a05-99e4-0bbe8cf903f0)<br>
    *Figure 3: Trochoidal (Gerstner) Waves*

    These equations were plugged into shader graph, and make decently convincing waves. To spruce up our ocean, we add multiple Gerstner waves, with different variables. Playing with them is quite fun, but it never truly makes a convincing sea; for that, we would need many, many waves. There are two solutions to this. One: we switch to fast Fourier transform, or two: add normal maps and textures. I went with the second option out of ease, but I intend to learn Fourier transforms for Unity at some point (A project idea is decoding the sound waves of various instruments to create basic synths!). The waves are now decently convincing, but the ship still isn't. The next thing I added was a buouyancy feature.
 
    <img width="2431" height="1137" alt="Screenshot 2025-12-24 201354" src="https://github.com/user-attachments/assets/d6460410-b81d-434a-8fee-c8bc8647ad6f" /><br>
    *Figure 4: Floating Pirate Ship*

    Four points were placed on the ship's hull in a rectangle. These points are fixed with the ship. Since the can rotate while moving with the A and D keys, I needed the points to rotate about the center of the ship. This took some minor trig equations. This is tracked by:
    
    ```csharp
    Vector3 angle = -transform.parent.parent.rotation.eulerAngles;
    Vector3 basePos = new Vector3(transform.parent.position.x + radius * Mathf.Cos(Mathf.Deg2Rad * (angle.y + localAngle)), 10, transform.parent.position.z + radius * Mathf.Sin(Mathf.Deg2Rad * (angle.y + localAngle)));
    ```

    Now that these points accurately follow the ship, I added a second set of points on the exact same x and z coordinate; these points differ in the fact that their y position follows precisely where the water level is, while the y coordinate of the first set of points follows the ship. The next function I added returned the water height at a given position. For this to work, I had to approximate the height as described earlier.

    Multiple methods were written to perform the same Gerstner wave calculations as written in shader graph. After these were complete, they returned the *change* in each coordinate from each wave. Even though an exact height couldn't be determined, a change in what the height originally was could be (As well as x and z positions). After Deltas was written, a second function called FindAB was written. In the equations from earlier, you might notice $\alpha$ and $\beta$ represent Vertex Positions. In an actual equation, these locations have an unknown offset. If we can calculate the change in x and z, we can apply this change to $\alpha$ and $\beta$ to find our y coordinate at an exact position.
    
    We start with a guess:

    $\alpha = currentPos.x$ and $\beta = currentPos.z$

    Then, we calculate the deltas. If we did this once, however, our approximation is still wrong. It will swing too far in the opposite direction. To counteract this, we lerp *most* of the way there, but not all of the way. Now we're close, but still a little off. We can recalculate the deltas and lerp again. With each recalculation, we swing around the accurate point like a dampening sin wave. With the right relax value (lerp input) this process is shortened. Currently, my relxas is set to 0.7 with 3 iterations. Note that the code below is my own, but the approximation solution was *not* my own.

    Now that $\alpha$ and $\beta$ are written, we can plug in them into our deltas function from earlier. This finds the y displacement at precisely the x and z coordinate inputted. Now, we can return currentPos.y + dy.

    ```csharp
    float fixed_displacement(Vector3 Position, Vector3 Dir1, Vector3 Dir2, Vector3 Dir3, Vector3 Dir4, Vector4 TimeScale, float Gravity, float Phase, float Depth, float amp1, float amp2, float amp3, float amp4, float time)
    {
      findAB(Position, Dir1, Dir2, Dir3, Dir4, TimeScale * time, Gravity, Depth, Phase,
             amp1, amp2, amp3, amp4, out float a, out float b);//returns alpha and beta, the modified x and z constants that return the correct dy

      deltas(new Vector3(a, 0, b), Dir1, Dir2, Dir3, Dir4, TimeScale * time, Gravity, Depth, Phase,
            amp1, amp2, amp3, amp4, out _, out float dy, out _); //returns the change in height, y

      float waveHeight = dy + Position.y;
      return waveHeight;
    }
  ```

- Due to the amount of control I wanted over the water's appearance, there are a ridiculous amount of parameters to each function. This is a great example of how I could have improved code. There are two specific things I've learned that could clean this up drastically--scriptable objects, and class instancing.

  To finish the buouyancy script, the heights of the ship's buouys and the water's buouys were compared, and a force was added. Similarly, the heights of the buouys relative to each other was compared to rotate the ship. Now that forces were dynamically added, depending on where the ship steered it's speed would be amplified or dampened.

<img width="2385" height="849" alt="Screenshot 2025-12-24 201632" src="https://github.com/user-attachments/assets/c1252340-91db-4905-8ffb-f8b122fa103b" /><br>
*Figure 5: Pirate Ship with Buouys*
  

- The next system that was added was a splashing system. An advantage or Gerstner waves was that it was *really* easy to further displace vertices; I only needed to add the splash function to the preexisting Gerstner function. To do this, I created a custom node in the HLSL script that could create a splash and used generic wave propogation equations straight out of my mechanics class. 

  $\y(r, t) = A\sin(kr - \omega t) $

  where

  $\r = \sqrt{(worldPos.x - waveOrigin.x)^2 + (worldPos.z - waveOrigin.z)^2} $

	This wave equation was then multiplied by a a fadeOut and fadeIn equation, both of which simply go from 0 to 1 in a short period of time. Finally, the equation was passed into a lerp function, where the added height from the splash would go from y(r, t) to 0 in a set amount of time. The only thing I would change about this in the future would be changing the dampening of the splash to be a sigmoid instead of a linear function. Additionally, this function is **not** accurate to, for example, a cannonball splash. For correct physics, the vertices would need to follow Navier-Stokes equations upon the initial impact. The effect is still convincing when covered by particles and pixelated.

	Now onto coding the problem. This was my first time using HLSL. Naturally, there was a two hour period of diagnosing bugs as shader graph tried to access the wrong file. I am still not entirely sure why, but I chalked it up to poor internet and moved on. One of my future project goals is to build a small DS-like console out of a Raspberry Pi and similar components. When I do, I would like to import my Unity projects. Regardless of whether that comes to fruition, I designed this project with the ability to be run on a lightweight device. Because of this, I decided to limit the spawned waves with fixed arrays. This also made transferring data to materials easy.

  ```csharp
  #ifndef SPLASH_NODE
  #define SPLASH_NODE

  float4 _WaveOrigins[16];
  float _Amplitudes[16];
  float _WaveSpeeds[16];
  float _LifeSpans[16];
  float _TimePassed[16];
  int _WaveCount;

  void Splash_float(float3 worldPos, float waveLength, float fadeSpeed, float fallOffDist, float timePassed, out float3 displacement)
  {
    float y_value = 0;
    
    for (int i = 0; i < _WaveCount; i++)
    {
        if (_Amplitudes[i] == 0 || _LifeSpans[i] == 0) continue;

        float dist = distance(worldPos.xz, _WaveOrigins[i].xz);
        float maxRadius = _WaveSpeeds[i] * (timePassed - _TimePassed[i]);
        //dist = 0;
	
        if (dist < maxRadius)
        {
            float basic = _Amplitudes[i] * sin((dist / waveLength * 6.283) - maxRadius);
			
            float fallOff = saturate(fallOffDist / (dist + 1) * fallOffDist / (dist + 1));
			
            float fadeIn = saturate((timePassed - _TimePassed[i]) * fadeSpeed);
			
            float die = saturate((timePassed - _TimePassed[i]) / _LifeSpans[i]);
			
            y_value += lerp(basic * fallOff * fadeIn, 0, die);
            //y_value += basic * fallOff * fadeIn;
        }
    }
    displacement = float3(0, y_value, 0);
  }

  #endif // SPLASH_NODE
  ```

  To transfer data from script to HLSL, I designed a class named SplashInfo. It currently has three functions: AddWave, UpdateShader, and DeleteWaves. What's nice is that SplashInfo contains modifiable lists, but when it transfers data to the water material, it only sends an array (currently of size 16). There is one instance of SplashInfo that exists, so other scripts can easily access it and summon splashes. Currently, the camera performs a raycast when the user left-clicks. If the raycast hits water, then a splash is spawned at that location. Additionally, the boat constantly cycles through splashes to create a wake. Additionally, in the same MonoBehavior as SplashInfo (but outside the class!) there is a function that can change the water intensity. It is a multiplier to the Gerstner waves' amplitudes. This can be used for dynamic weather. 

  ![ShadedSplashed](https://github.com/user-attachments/assets/b1254a95-af0c-4fc1-b5ab-e9f175ea940e)<br>
*Figure 6: Shaded Splash Created from Raycast*

![WireFrameSplash](https://github.com/user-attachments/assets/ab8e9a9b-9b30-4c51-b9a0-765dbaef43a5)<br>
*Figure 7: Wireframe Splash Created from Raycast*
  
  That concludes my graphics intro! In the end, I made no shortage of mistakes; but learned from all of them. Though I was never satisfied with the water from this project, I feel confident that the next time I create water it will exceed my original vision!
    
## Future Improvements
- I plan to add some of the features of this simulation to my procedural planet generator, so I can make dynamic gravity with planets' water. When I do that, however, those features will be changed fairly significantly. I would also like to note that the procedural planet project was an answer to the problems from this project; it's a modular project so any underlying problems *don't* effect every other system directly, whereas with the water system here changing something in a fundamental script would introduce problems in later scripts. If you would like to reach out, please email me at mray0256@gmail.com. Thanks!

## Sources & Inspirations
- Ship Buouyancy: [Tom Weiland on YouTube](https://www.youtube.com/watch?v=eL_zHQEju8s)
- Pirate Game Inspiration: [V.A. on youTube](https://www.youtube.com/watch?v=uwU9ZaQ9PhY)
- Gerstner Waves: [Zicore on YouTube](https://www.youtube.com/watch?v=Awd1hRpLSoI)
- Pirate Ship Model: [PolyRem on YouTube](https://www.youtube.com/watch?v=MoqXP7tpL1o&t=500s)
- Simulating Waves Paper: [Jerry Tessendorf](https://jtessen.people.clemson.edu/reports/papers_files/coursenotes2002.pdf)
- Technical Art Presentation: [Sea of Thieves devs at GDC](https://www.youtube.com/watch?v=y9BOz2dFZzs&t=1662s)
