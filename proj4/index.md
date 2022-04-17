# Project 4: Cloth Sim

**Contributors**: Micah Yong (micahtyong@berkeley.edu) and Kevin Li (kevintli@berkeley.edu)

## Overview

In this project, we implemented a real-time cloth simulation using a mass and spring-based system. We implemented the exact physical constraints on each point mass and spring, then applied numerical integration to simulate the way the cloth moves over time. We implemented collisions with other objects as well as handling for self-collisions. We learned advanced techniques such as the 10% rule to define deformation constraints and Verlet integration to estimate cloth movement efficiently. 

## Part 1: Masses and Springs

To communicate our results for part 1, we portray the following screenshots of `scene/pinned2.json` to show the structure of our masses and springs. 

| Experiment description      | Result |
| ----------- | ----------- |
| Without any shearing constraints (you might have to zoom in)      | ![p5](images/task-1-no-shear.png)       |
| With only shearing constraints      | ![p5](images/task-1-shear.png)       |
| With all constraints applied     | ![p5](images/task-1-all.png)       |

## Part 2: Simulation via Numerical Integration 

In the following experiment, we describe how individual parameters affect the end result of the cloth simulation. 

**Varying the spring constant, `ks`**
 
- With a very low `ks` (e.g., 10 N/m), the cloth is very loose with little spring force to maintain its original square layout. The top section appears to be stretched out. 
- With a very high `ks` (e.g., 99999 N/m), the cloth is held tightly by the two pins with ample spring force to maintain its original square layout. The top section appears to be much tighter than the first experiment.

| Low `ks`: `10 N/m`      | High `ks`: `99999 N/m` |
| ----------- | ----------- |
| ![p5](images/task2-low-spring.png)      | ![p5](images/task2-high-spring.png)       |

**Varying the density constant, `density`**

- With a very low `density` (e.g., 0.1 g/cm^2), the cloth is very tight and light, making it much easier for a small amount of spring force (e.g., 1000 N/m) to maintain its original square layout. The cloth’s behavior reminded me of a thin handkerchief. 
- With a very high `density` (e.g., 10000 g/cm^2), the cloth is very stretched and heavy, making it much more difficult for the same amount of spring force (e.g., the default 1000 N/m) to maintain its original shape. For example, the top section appears to be stretched out. The cloth’s behavior reminded me of thick rope. 

| Low `density`: `0.1 g/cm^2`      | High `density`: `10000 g/cm^2` |
| ----------- | ----------- |
| ![p5](images/task2-low-density.png)      | ![p5](images/task2-high-density.png)       |

**Varying the damping constant**

- With a very low `damping` (e.g.,`0%`), the cloth infinitely remains in motion. In particular, the cloth swings back and forth due to the spring force and gravity, with its effects remaining the same overtime. Without knowing the parameters of this simulation, one might think that there’s a constant wind force in the environment. 
- With a very high `damping` (e.g.,`1%`), the cloth is incredibly stiff. In particular, the cloth falls in slow motion and reaches a stand-still as soon as the cloth reaches its resting position the first time (i.e., without the need to swing back and forth to slow down). The cloth’s behavior reminded me of a thick sheet of foam.

| Low `damping`: `0%` (after 1 second)      | High `damping`: `1%` (after 1 second) |
| ----------- | ----------- |
| ![p5](images/task2-low-damping.png)      | ![p5](images/task2-high-damping.png)       |

Here is what our shaded cloth from `scene/pinned4.json` looks like in its final resting state:

![p5](images/p2-pinned4.png)

## Part 3: Handling collisions with other objects

**sphere.json**

The following table portrays our results from a cloth falling onto a sphere. We vary the spring force constant, `ks`, and write down our observations. 

| Experiment description      | "Final" resting state | Observations |
| ----------- | ----------- | ----------- |
| Set `ks = 5000` (baseline) | ![p5](images/task3-default-sphere.png) | The cloth rests on the sphere as expected. We will use this image as a baseline for the following two experiments. |
| Set `ks = 500` | ![p5](images/task3-lower-ks.png) | The cloth appears to be much more loose (more drape-y) when resting on the sphere, compared to the baseline experiment of `ks = 5000`. |
| Set `ks = 50000` | ![p5](images/task3-higher-ks.png) | The cloth appears to be much more stiff when resting on the sphere, compared to the baseline experiment of `ks = 5000`. |

The true final resting state of all of our experiments is merely the sphere itself, since the cloth falls off after a long period of time.

**plane.json**

The following table portrays our results from a cloth falling onto a plane. We chose to use a pink-ish white color to celebrate the fading cherry blossom season. We also show the same resting result with the default normal color. 

| Resting position, custom wireframe      | Resting position, default coloring |
| ----------- | ----------- |
| ![p5](images/task3-custom-plane.png)      |   ![p5](images/task3-default-plane.png) |


## Part 4: Self-Collisions

### Basic self-collision example with cloth
| | | |
| -- | -- | -- |
| ![p5](images/p4-anim-1.png) | ![p5](images/p4-anim-2.png)  |  ![p5](images/p4-anim-3.png)  | 
| ![p5](images/p4-anim-4.png) | ![p5](images/p4-anim-5.png)  |

### Results of varying density and `ks`
With lower density, the cloth remains fairly smooth during the self-collision, and quickly flattens itself out after making contact with the ground. With higher density, the cloth crumples a lot more upon landing, and appears to be in a series of clumps even as it slowly flattens out.

| Lower density (1 g/cm^2) | Original density (15 g/cm^2) | Higher density (100 g/cm^2) |
| -- | -- | -- |
| ![p5](images/p4-low-density.png) | ![p5](images/p4-orig-density.png) | ![p5](images/p4-high-density.png) |

With a lower `ks`, the cloth tends to take on a more wrinkled shape and easily collapses on itself. With a higher `ks`, it instead appears more “bouncy” during self-collisions, and retains its original flat shape for the most part.


| Lower `ks` (1 N/m) | Original `ks` (5000 N/m) | Higher `ks` (100000 N/M) |
| -- | -- | -- |
| ![p5](images/p4-low-ks.png) | ![p5](images/p4-orig-density.png) | ![p5](images/p4-high-ks.png) |


<br>

## Part 5: Shaders

Shader programs are programs that implement useful calculations for the graphics pipeline (e.g. computing positions, normal vectors, and colors), and can be executed in parallel on GPU in order to speed up rendering significantly.

In this project, we used the OpenGL Shading Language (GLSL) to implement our shaders. We worked with two types of shaders:
- **Vertex shaders**, which modify the position and normal vectors of vertices that are later used by the fragment shader
- **Fragment shaders**, which compute the color that should be used for each fragment of the output image (which is similar to a pixel).

<br>

### Blinn-Phong shader
The Blinn-Phong shading model takes into account three components of lighting:
- **Diffuse lighting**, which depends only on the angle, distance, and characteristics of the lighting source, not on the viewing angle
- **Specular lighting**, which depends on the above and additionally the viewing angle
- **Ambient lighting**, which is a constant value added to every part of the scene

| Ambient only | Diffuse only |
| -- | -- |
| ![p5](images/p5-phong-ambient.png) | ![p5](images/p5-phong-diffuse.png)  |

| Specular only | Full Blinn-Phong model |
| -- | -- |
| ![p5](images/p5-phong-specular.png) | ![p5](images/p5-phong-full.png)  |

<br>

### Texture mapping shader
Below is an example of texture mapping with a custom texture file:

| Cloth and sphere (before falling) | Cloth and sphere (after falling) |
| -- | -- |
| ![p5](images/p5-texture-before.png) | ![p5](images/p5-texture-after.png)  |

<br>

### Bump and displacement mapping

**Bump mapping**
| Cloth and sphere (before falling) | Cloth and sphere (after falling) |
| -- | -- |
| ![p5](images/p5-bump-before.png) | ![p5](images/p5-bump-after.png)  |

**Displacement mapping**
| Cloth and sphere (before falling) | Cloth and sphere (after falling) |
| -- | -- |
| ![p5](images/p5-disp-before.png) | ![p5](images/p5-disp-after.png)  |

**Comparison of bump and displacement mapping**
<br>
Bump mapping modifies the normal vectors of vertices to create the *illusion* of texture, while displacement mapping additionally modifies the position of vertices to actually reflect the geometry of this new texture. Because of this, we can see that displacement mapping, unlike bump mapping, changes the actual shape of the sphere in the comparison below:

| | Bump | Displacement |
| -- | -- | -- |
| `-o 16 -a 16` | ![p5](images/p5-bump-sphere-16.png)  |  ![p5](images/p5-disp-sphere-16.png)  | 
| `-o 128 -a 128` | ![p5](images/p5-bump-sphere-128.png)  | ![p5](images/p5-disp-sphere-128.png) |

<br>

### Mirror Shader

| Cloth and sphere (before falling) | Cloth and sphere (after falling) |
| -- | -- |
| ![p5](images/p5-mirror-before.png) | ![p5](images/p5-mirror-after.png)  |

And here are the results of the mirror shader on the top surface of the cloth:

![p5](images/p5-cloth-top.png)