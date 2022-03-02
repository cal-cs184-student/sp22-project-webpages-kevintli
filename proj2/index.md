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

## Task 4: Edge Flip 

Implementing the edge flip operation requires reassigning pointers for the various mesh elements of the two involved triangles. We found the following [diagram from CMU’s computer graphics course](https://cmu-graphics.github.io/Scotty3D/meshedit/local/edge_flip_diagram.png) extremely helpful as a guide of all the elements that may potentially be affected:

![t11](/images/task4-cmu.png)

One observation we made was that the outer halfedges (`h6`, `h7`, `h8`, `h9` in the diagram above) were not affected by the edge flip – all of their pointers would remain the same. Each edge’s halfedge would remain the same as well. Thus, the only elements we had to consider were the four vertices, the two faces, and the six inner halfedges.

As mentioned in the writeup, vertices and faces only have a `halfedge` pointer that may need to be reassigned, whereas halfedges have five pointers: `next`, `twin`, `vert`, `edge`, and `face`. Below are the elements we considered, each labeled with which pointers needed to be reassigned:

| Vertices and faces       | Half edges |
| ----------- | ----------- |
| ![t11](/images/task4-vertices.png)   | ![t11](/images/task4-halfedges.png)       |

(As a side note, it is possible that the halfedge pointers for vertices/faces remain correct after the edge flip, and that no change is needed. However, since the choice of halfedge is arbitrary for these elements, we wanted to be safe and set them all regardless. For example, f0’s halfedge were set to h1 before the flip, then this would no longer be correct after the flip — just in case, we set it to h0 instead.)

Here’s an example of some edges being flipped on the teapot:

| Before flip      | After flip |
| ----------- | ----------- |
| ![t11](/images/task4-beforeflip.png)   | ![t11](/images/task4-afterflip.png)       |

## Task 5: Edge Split

The edge split operation is similar to the edge flip operation in the sense that we need to update the pointers of our mesh elements — however, there is some extra work involved due to the creation of new vertices, edges, and halfedges. Taking inspiration from the diagram from Task 4, we drew a new diagram for what the edge split operation would do (newly created mesh elements are labeled in green):

![t11](/images/task5-split-diagram.png)

Since there are a lot more elements involved, we decided to just reassign all the pointers for all elements according to the diagram above, in order to avoid the possibility of bugs. Additionally, we observed that the resulting diagram is made of four symmetric triangles (not the shapes themselves, but the mesh elements). So to keep our code more concise, we wrote a helper function to assign the pointers of a single triangle, using the upper left triangle in the diagram as reference, then simply reused it for all the other triangles.

Here is an example of `dae/teapot.dae` before and after some edge splits. The region circled in red was split multiple times, while the region circled in purple was split only once:

| Before split      | After split |
| ----------- | ----------- |
| ![t11](/images/task5-before.png)   | ![t11](/images/task5-after.png)       |

And here is the same before/after diagram but with a combination of edge splits and edge flips:

| Before splits/flips      | After splits/flips |
| ----------- | ----------- |
| ![t11](/images/task5-before.png)   | ![t11](/images/task5-after-wflip.png)       |

## Task 6: 