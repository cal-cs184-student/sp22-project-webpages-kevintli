# Project 3: Pathtracer (Part 1)

**Contributors**: Micah Yong (micahtyong@berkeley.edu) and Kevin Li (kevintli@berkeley.edu)

## Overview

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
