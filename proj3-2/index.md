# Project 3: Pathtracer (Part 1)

**Contributors**: Micah Yong (micahtyong@berkeley.edu) and Kevin Li (kevintli@berkeley.edu)

## Overview

In this project, we extend our results from our ray tracer from project 3-1 by making two primary contributions.
1. First, we develop support for glass materials and mirror-like surfaces. We made such contributions in the MirrorBSDF class. This task covered topics and techniques such as reflection, refraction, and Schlick’s approximation. 
2. Second, we develop support for microfacet metal-like materials like gold, copper, and nickel. We made such contributions in the MicrofacetBSDF class. This task covered topics and techniques such as Fresnel terms, normal distribution functions, and importance sampling according to the Beckmann distribution. 

The most important part of this project was understanding and correctly implementing the complicated procedures/equations described in lecture. We approached the project by first carefully reading the spec, focusing on the required implementation items and mistakes to watch out for, then writing the code while frequently referring back to the spec and lecture slides. We encountered several tricky bugs, particularly with Part 2. To fix these, we found it helpful to pair program in order to sanity check our assumptions, and also take breaks and come back to the code with a fresh pair of eyes.

## Part 1: Mirror BRDF

### Summary 

In this problem, we added support for mirror and glass materials by implementing reflection and refraction. To do this, we added three new BSDF and sampling functions:

- **Mirror**: Given an incoming light ray, simply reflect it in the opposite direction (x, y change signs but z stays the same).
- **Refraction**: Given an incoming light ray, we compute the direction of refraction as described in Lecture 14: Material Modeling, as well as the BSDF value based on the index of refractions of the two materials.
- **Glass**: This is a combination of reflection and refraction. Instead of trying to model the exact Fresnel equations, we apply Schlick’s approximation, which computes a reflection coefficient R according to the index of refraction and then decides whether to reflect or refract each ray with a coin flip (with reflection probability R).

### Results

## Part 2: Microfacet BRDF

### Summary

In part 2, we implemented the microfacet BRDF model. In computer graphics, the microfacet BRDF models metallic materials that primarily reflect rather than refract (e.g., copper, gold, nickel-iron alloys). To implement this model, we helped fill in the MicrofacetBSDF class, which inherits from the BSDF class.  

In `MicrofacetBSDF::f` (task 1), we followed the derivation of the microfacet evaluation model derived in lecture, which makes use of the fresnel term, shadow-masking term, and normal distribution function (NDF). For the NDF term, we passed in the half vector between `wi` and `wo`. We employed `(0, 0, 1)` as our surface normal. 

In `MicrofacetBSDF::D` (task 2), we implemented the normal distribution function using Beckmann’s distribution. We vary our value for roughness, `alpha`, in the results section below. 

In `MicrofacetBSDF::F` (task 3), we implemented the Fresnel term evaluation function using the air-conductor approximation presented in the spec. We vary our indices of refraction, `eta` and `k`, in the results section below to create a nickel-iron variant of the bunny. 

In `MicrofacetBSDF::sample_f` (task 4), we implemented importance sampling according to the shape of the Beckmann distribution. Here, it was imperative to check that the resulting `pdf` was positive and the sampled incident angle, `wi`, and outgoing angle, `wo`, were pointing in the positive direction. Without the aforementioned checks, some black dot artifacts would appear in the images. 

### Results

First, we will present a sequence of four renders of `CBdragon_microfacet_au.dae` with varying levels of `alpha`. Since `alpha` represents the roughness of the macro surface, you will notice that smaller values of `alpha` yield glossier dragons, while larger values of `alpha` yield dragons with a matte-like surface (more light diffusion). We rendered the following images using 128 samples per pixel, 8 threads, 5 light bounces, and 1 sample per light. 

| Alpha      | Resulting Dragon |
| ----------- | ----------- |
| `0.5`      | ![p5](microfacet_dragon_0p5.png)       |
| `0.25`   | ![p5](microfacet_dragon_0p25.png)        |
| `0.05`   | ![p5](microfacet_dragon_0p05.png)        |
| `0.005`   | ![p5](microfacet_dragon_0p005.png)        |

Second, we will present a side by side comparison of `CBbunny_microfacet_cu.dae`. The left image uses the default cosine hemisphere sampling while the right image uses our contributed importance sampling. In the left image, you will notice several rendering artifacts on and around the bunny. The reason for this observation is that it takes much longer for cosine hemisphere sampling to converge compared to importance sampling (e.g. 64 samples per pixel was likely not enough). The right image uses importance sampling that follows the Beckmann distribution––the same distribution we used for our NDF term. Thus, we achieve an accurately-depicted bunny with only 64 samples per pixel. We rendered the following images using 64 samples per pixel, 8 threads, 5 light bounces, and 1 sample per light. 

| With cosine hemisphere sampling      | With importance sampling following the Beckmann distribution |
| ----------- | ----------- |
| ![p5](microfacet_bunny_HS.png)      | ![p5](microfacet_bunny_IS.png)       |

Finally, we will present a new version of `CBbunny_microfacet_cu.dae` using a nickel-iron alloy derived [here](https://refractiveindex.info/?shelf=other&book=Ni-Fe&page=Tikuisis_gold150nm). Our newly derived values for `eta` and `k` give the bunny a sandpaper-like silver look (we used `alpha = 0.05`). The resulting  material is reminiscent of a roughed up permalloy. We rendered the following images using 64 samples per pixel, 8 threads, 5 light bounces, and 1 sample per light (same as above). Both images below use importance sampling. 

| Copper      | Nickel-Iron |
| ----------- | ----------- |
| ![p5](microfacet_bunny_IS.png)      | ![p5](microfacet_bunny_nife.png)       |


## Note on Collaboration

Kevin Li and Micah Yong collaborated on the path tracing project together in a similar fashion to project 3-1. 
- How we collaborated: Kevin led task 1: MirrorBSDF, while Micah led task 2 on MicrofacetBSF. In both parts, one person was a lead (driver) and one person was a watcher (passenger). Leads were responsible for performing most of the initial research, coding implementation, and pull request. Passengers were responsible for assisting in pair-programming sessions, providing supplemental research and insights for the problem, and carefully reviewing the lead’s code. 
- How it went: We found that this system works very well for completing most tasks, as ownership and responsibilities were clearly delineated from the onset. However, for highly complex tasks or debugging sessions, the “roles” faded away and we needed to be highly collaborative throughout the entire process. This was especially the case for Project 3.2.2, where we faced issues debugging the black dot artifacts. When the driver (Micah) bounced off ideas with his passenger (Kevin), it was much easier to debug compared to when the driver attempted to debug on his own. 
- What we learned: With regards to collaboration, we learned how to communicate our thoughts clearly to one another and give constructive feedback during code reviews. This collaboration gives us a taste of what working with colleagues on challenging problems would be like in the real world. Furthermore, we learned to code more carefully and thoughtfully by taking breaks, sleeping well before a coding session, and eating before a coding session. 

