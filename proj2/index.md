# Project 2: MeshEdit 

**Contributors**: Micah Yong (micahtyong@berkeley.edu) and Kevin Li (kevintli@berkeley.edu)

## Overview


## Task 1: Bezier Curves with 1D de Casteljau Subdivision

De Casteljau's algorithm is a strategy for evaluating Bezier curves. The algorithm uses recursive subdivision and successive linear interpolation to compute numerically accurate Bezier curves. To implement his algorithm, 
- We defined a linear interpolation helper function, `Vector2D lerp(Vector2D p1, Vector2D p2, float t)`, which takes in two points and returns a control point between `p1` and `p2`. 
- In `evaluate_step`, we use `lerp` for all successive pairs of points (i.e., point 0 and point 1, point 1 and point 2, … point n - 1 and point n). Then, we return a vector of `n - 1` points. 
- In `bezierCurve.cpp`, we call `evaluate_step` each time we press `e` in the GUI. For a starting control vector of `n` points, we press `e` `n-1` times to retrieve the final point––which is guaranteed to lie on the Bezier curve for parameter `t`. 


Here are screenshots showing each step / level of the evaluation from the original control points down to the final evaluated point.


| Level      | Screenshot |
| ----------- | ----------- |
| 1   | ![t11](/images/task1-1.png)       |
| 2   | ![t12](/images/task1-2.png)        |
| 3   | ![t12](/images/task1-3.png)        |
| 4   | ![t12](/images/task1-4.png)        |
| 5   | ![t12](/images/task1-5.png)        |
| 6   | ![t12](/images/task1-6.png)        |
| 7   | ![t12](/images/task1-7.png)        |

Here is a screenshot screenshot of a slightly different Bezier curve. We achieved this different Bezier curve by moving the center control point down and adjusting the neighboring points. We also increased `t` to be close to `1`. 

![t12](/images/task1-vary.png) 

## Task 2: Bezier Surfaces with separable 1D via de Casteljau

As explained in [Lecture 7]​​(https://cs184.eecs.berkeley.edu/sp19/lecture/7-90/geometry-and-splines), here is how de Casteljau algorithm extends to Bezier surfaces:
1. We can use the 1D implementation of De Casteljau’s Algorithm as a subroutine to evaluate `n` Bezier successive curves independently. This step should return a list of points, where each point corresponds to a point along its corresponding Bezier curve evaluated at `t = u`.
2. We can use the 1D implementation of De Casteljau’s Algorithm once again on the list of points from Step 1. The result is a point that is guaranteed to be on the Bezier surface across the given `n` Bezier curves evaluated at `t = v`. 

Here is the requested screenshot of `bez/teapot.bez`, evaluated our implementation. 

![t12](/images/task2-withwireframe.png) 

We also provide a screenshot without the wireframes. 

![t12](/images/task2-without.png) 

## Task 3: Area-weighted vertex normals

To implement area-weighted vertex normals, we followed the [simple scheme described in lecture](https://cs184.eecs.berkeley.edu/sp22/lecture/6-36/rasterization-pipeline). Concretely,
1. We collected the positions of neighboring vertices with respect to the current vertex by traversing half edges.
2. Then, we computed the surface normal of each face, following the geometric derivation outlined in [this article about surface normals](https://www.khronos.org/opengl/wiki/Calculating_a_Surface_Normal). Then, we weight the surface normal by its respective area by calling the `unit()` function on the result. 
3. Finally, we returned the final normal vector for the vertex by summing over the weighted normals of each face, then calling the `unit()` function on the result. 


Finally, here is a diagram that compares the output of `dae/teapot.dae` with and without shading. 

| Flat Shading       | Phong shading |
| ----------- | ----------- |
| ![t11](/images/task3-flat.png)   | ![t11](/images/task3-phong.png)       |