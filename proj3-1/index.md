# Project 3: Pathtracer (Part 1)

**Contributors**: Micah Yong (micahtyong@berkeley.edu) and Kevin Li (kevintli@berkeley.edu)

## Overview

In this project, we contribute a physically-based renderer using a path tracing algorithm. Concretely, our contributions take on 5 main parts.
1. In part 1, we perform ray generation and primitive scene intersection. By the end of this part, we were able to generate camera rays, sample rays over a unit pixel, and compute intersections between rays and two primitives: the triangle and sphere. 
2. In part 2, we constructed a bounding volume hierarchy for an input scene using the recursive algorithms outlined in lecture. By the end of this part, we were able to generate a bounding volume hierarchy for a given set of primitives and intersect a ray with the BVH efficiently (i.e., in log time rather than linear time). In this part, it was crucial to keep track of pointers and valid intersection intervals between a ray and bounding box for part 2.2. 
3. In part 3, we implemented direct illumination for both zero-bounce rays and one-bounce rays. To implement one-bounce rays, we constructed two sampling algorithms: Direct lighting with uniform hemisphere sampling and direct lighting with importance sampling. We needed to carefully compute normalization factors and ensure that we operated in the correct coordinate system (either world space or local object space). 
4. In part 4, we implemented global illumination for at-least-one-bounce rays. We used an unbiased method for terminating rays called Russian Roulette. By the end of both parts 3 and 4, we were able to render photorealistic images with complex light scenes. 
5. In part 5, we sped up our pixel-sampling algorithm from part 1 by terminating sampling algorithms that have likely converged already via adaptive sampling. By the end of this part, we could render more noise-free images more efficiently. One bug we faced was regarding floating point errors, where the measure of importance was always equating to zero. To solve this problem, we ensured that all integers were converted to floating point before computing `I`. 


## Part 1: Ray generation and scene intersection

The overarching goals for part 1 were two-fold: 
1. Between tasks 1 and 2, our goal was to implement a ray generation sampling mechanism that allows us to estimate the integral of radiance over a pixel in a computationally feasible way (i.e., suggesting that computing the exact integral of radiance over a pixel is oftentimes infeasible). In task 2, we achieved this mechanism by sampling `num_samples` rays uniformly in a pixel (i.e., between (x, y) and (x + 1, y + 1)), then averaging the resulting radiance estimates. To generate any ray, we relied on our implementation of task 1, which generates a ray from the camera’s perspective passing through an ideal sensor plane (i.e., z-coordinate always set to `-1` in camera space). 
2. Between tasks 3 and 4, our goal was to compute or check intersections with two primitive shapes, the triangle and sphere. Both implementations relied on the same fundamental principle: set the ray equation equal to the primitive’s, solve for `t` (time), then compute the resulting surface normal for measuring surface radiance. If our selected `t >= 0` and within the camera’s visible boundaries (i.e., `t <= max_t && t >= min_t`), then we can conclude that there exists some real point along the input ray that intersects with the primitive. 

Now, we’ll dive into task 3, ray-triangle intersection, in more detail.
- We defined a helper function, `getMollerTrombore`, which computes `[t b1 b2]` efficiently with the [Moller Trumbore algorithm defined in lecture](https://cs184.eecs.berkeley.edu/sp22/lecture/9-22/ray-tracing). Here, `b1` and `b2` refer to the Barycentric coordinates of our intersection at time `t`. 
- We defined another helper function, `getBaryCoordinates`, which computes `[b1 b2 b3]`, the normalized Barycentric coordinates of our intersection at time `t`. 
- Then, we implemented `Triangle::has_intersection(..)`, which determines whether or not our ray intersects with the triangle based on `t` from `getMollerTrombore` and our Barycentric coordinates from `getBaryCoordinates`. Concretely, if `t < 0` or the intersection is not within the camera’s visible boundaries (i.e., `t <= max_t && t >= min_t`), then we return false. Furthermore, if any of our Barycentric coordinates aren’t within `[0, 1]`, the intersecting point is outside of the triangle’s plane and we return false. If both of these conditions pass, then the ray must have a valid intersection with the triangle. 
- Finally, we implemented `Triangle::intersect(..)`, which determines whether or not our ray can intersect with the triangle––if so, we can populate `isect` with metadata regarding the intersection. Concretely, we determine the surface normal at our intersection by interpolating the Barycentric coordinates with the vertex normals, then save the result. Furthermore, we save the time of intersection, `t`, the surface material (i.e., `BSDF`), and the triangle itself (i.e., `this`). Finally, we update `max_t = t`, since we don’t need to consider any intersections behind the triangle plane. 

Here are some screenshots after implementing part 1, which utilizes both `Triangle::intersect(..)` and `Sphere::intersect(..)`: 

![t12](images/task1-1.png)
![t12](images/task1-2.png)

## Part 2: Bounding volume hierarchy

To construct a bounding volume hierarchy (BVH) for a set of primitives, we implement a recursive algorithm as follows: 
1. First, we iterate through all primitives on the scene to construct the root bounding box. 
2. Then, if the number of primitives in the bounding box (computed via `std::distance(start, end)`) is greater than our target leaf size, we will determine the splitting point as the midpoint of the longest axis for the current bounding box. At this point, we can recursively call `construct_bvh(..)` for the left and right nodes based on our splitting point (i.e., all primitives to the left of our splitting point will enter the left node, all primitives to the right of our splitting point will enter the right node). We succinctly achieved the above assignments using `std::partition`. 
3. Otherwise, if the number of primitives in the bounding box is less than or equal than our target leaf size (or we’ve finished computing step 2), we can simply return the node constructed from step 1. 

Reported below are images rendered from a few large .dae files. Without BVH acceleration, each of the files below would’ve taken too long to render (shortest render time without BVH is 47 seconds).

![t12](images/task2-1.png)
![t12](images/task2-2.png)
![t12](images/task2-3.png)
![t12](images/task2-4.png)

To communicate the impact of BVH on rendering times, we perform A/B testing on a few moderately complex geometry scenes and compare rendering stats. 

| Geometry file      | Without BVH | With BVH | Difference |
| ----------- | ----------- | ----------- |----------- |
| cow.dae      | (47.6920 seconds, 469993 rays traced, 2386.484205 intersection tests per ray)      | (0.6089 seconds, 471133 rays traced, 23.980791 intersection tests per ray)      | (240x speedup, 100x fewer intersection tests per ray)      |
| maxplanch.dae   | (553.8451 seconds, 454831 rays traced, 24826.189824 intersection tests per ray)       | (1.1775 seconds, 475518 rays traced,  40.440957 intersection tests per ray)       | (470x speedup, 620x fewer intersection tests per ray) |

From the above experiment where we compare statistics on geometries rendered without and with BVH acceleration, we see that BVH results in a >240x speedup in rendering time for `cow.dae` and a >470x speedup in rendering time for `maxplanch.dae`. We achieve these results by pruning nodes whose bounding box does not intersect with the traced ray. Thus, we performed 100x fewer primitive intersection tests per ray for `cow.dae` and 620x fewer primitive intersection tests per ray for `maxplanch.dae`. In lecture, we derived that primitive intersection tests are computationally expensive (even with our applied optimizations). Through BVH acceleration, we can make significantly fewer primitive intersection tests and therefore cut runtime by orders of magnitude.

## Part 5: Adapative Sampling

To perform adaptive sampling, we modify our `Pathtracer::raytrace_pixel(..)` implementation as follows: 
1. We keep track of both a running sum and running squared sum, denoted by variable names `s1` and `s2`, respectively. 
2. At every iteration, we compute the sample’s illuminance by calling `radiance.illum()`, where `radiance` is our estimate from Task 3 and 4. Then, we update `s1` and `s2` and continue with our uniform sampling algorithm from Task 1.2. 
3. After every `samplesPerBatch` samples, we compute the mean, variance, and measure of pixel convergence ([source](https://cs184.eecs.berkeley.edu/sp22/docs/proj3-1-part-5)). If our measure of pixel convergence is less than or equal to `maxTolerance * mean`, we can halt our sampling algorithm, update the sample buffer with our average, update the count buffer with the number of samples we performed, then return. 

Depicted below is the bunny scene with `2048` samples per pixel. Here, we use 1 sample per light and 5 for max ray depth. Take special notice of the sample rate image (right), which shows how adaptive sampling changes depending on which part of the image we are rendering.


| Noise-free Rendered Image      | Sample Rate Image |
| ----------- | ----------- |
| ![t12](images/task5-bunny.png)      | ![t12](images/task5-bunny_rate.png)       |

### Note on collaboration

Kevin Li and Micah Yong collaborated on the path tracing project together. 
- How we collaborated: For every part, one person was a lead (driver) and one person was a watcher (passenger). Leads were responsible for performing most of the initial research, coding implementation, and pull request. Passengers were responsible for assisting in pair-programming sessions, providing supplemental research and insights for the problem, and carefully reviewing the lead’s code.  
- How it went: We found that this system works very well for completing most tasks, as ownership and responsibilities were clearly delineated from the onset. However, for highly complex tasks or debugging sessions, the “roles” faded away and we needed to be highly collaborative throughout the entire process. This was especially the case for tasks like Project 2.5 and Project 3.1.2. 
- What we learned: With regards to collaboration, we learned how to communicate our thoughts clearly to one another and give constructive feedback during code reviews. This collaboration gives us a taste of what working with colleagues on challenging problems would be like in the real world. 
