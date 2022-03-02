# Project 2: MeshEdit 

**Contributors**: Micah Yong (micahtyong@berkeley.edu) and Kevin Li (kevintli@berkeley.edu)

<br>

## Link to Writeup
https://cal-cs184-student.github.io/sp22-project-webpages-kevintli/proj2/index.html

<br>

---
<br>

## Overview

In this project, we built Bezier curves and surfaces using de Casteljau algorithm, manipulated triangle meshes represented by half-edge data structure, and implemented loop subdivision. These features are a critical part of representing realistic shapes and textures in graphics applications.

The most interesting thing we gained from this project was an understanding for why certain concepts or data structures presented in lecture were designed the way they were. For example:
- Implementing the de Casteljau algorithm in Parts 1 and 2 helped us appreciate the elegance and simplicity of the recursive definition of Bezier curves. Although evaluating the Bernstein polynomials directly is possible, it would have been more complex and we would have to be very careful to make sure our mathematical expressions were correct.

- Working with HalfedgeMeshes in Parts 3-6 helped us understand why exactly the graphics community decided to define the data structure in this way. In lecture, the choice of vertices, edges, faces, and halfedges (as well as the pointers that each one had) seemed arbitrary, and although we were told that it was “similar to a linked list”, we had no intuition for why that would be useful — in fact, it seemed unnecessarily difficult to traverse a mesh using this data structure. But when we implemented edge flips and splits in Parts 4-5, it was clear why this “linked list”-like structure was critical, because it allowed us to very easily reassign pointers when inserting new mesh elements.

<br>

---
<br>

## Task 1: Bezier Curves with 1D de Casteljau Subdivision

De Casteljau's algorithm is a strategy for evaluating Bezier curves. The algorithm uses recursive subdivision and successive linear interpolation to compute the coordinates of points along a Bezier curve. To implement this algorithm, 
- We defined a linear interpolation helper function, `Vector2D lerp(Vector2D p1, Vector2D p2, float t)`, which takes in two points and returns a control point between `p1` and `p2`. 
- In `evaluate_step`, we use `lerp` for all successive pairs of points (i.e., point 0 and point 1, point 1 and point 2, … point n - 1 and point n). Then, we return a vector of the `n - 1` interpolated points. 
- In `bezierCurve.cpp`, we call `evaluate_step` each time we press `e` in the GUI. For a starting control vector of `n` points, we press `e` `n-1` times to retrieve the final point––which is guaranteed to lie on the Bezier curve for parameter `t`. 


Here are screenshots showing each step / level of the evaluation from the original control points down to the final evaluated point.


| Level      | Screenshot |
| ----------- | ----------- |
| 1   | <img src="images/task1-1.png" alt="t11" width="500"> |
| 2   | <img src="images/task1-2.png" alt="t12" width="500"> |
| 3   | <img src="images/task1-3.png" alt="t13" width="500"> |
| 4   | <img src="images/task1-4.png" alt="t14" width="500"> |
| 5   | <img src="images/task1-5.png" alt="t15" width="500"> |
| 6   | <img src="images/task1-6.png" alt="t16" width="500"> |
| 7   | <img src="images/task1-7.png" alt="t17" width="500"> |

Here is a screenshot screenshot of a slightly different Bezier curve. We achieved this different Bezier curve by moving the center control point down and adjusting the neighboring points. We also increased `t` to be close to `1`. 

![t12](images/task1-vary.png) 

<br>

---
<br>

## Task 2: Bezier Surfaces with separable 1D via de Casteljau

As explained in [Lecture 7](https://cs184.eecs.berkeley.edu/sp19/lecture/7-90/geometry-and-splines), here is how de Casteljau algorithm extends to Bezier surfaces:
1. We can use the 1D implementation of De Casteljau’s Algorithm as a subroutine to evaluate `n` Bezier successive curves independently. This step should return a list of points, where each point corresponds to a point along its corresponding Bezier curve evaluated at `t = u`.
2. We can use the 1D implementation of De Casteljau’s Algorithm once again on the list of points from Step 1. The result is a point that is guaranteed to be on the Bezier surface across the given `n` Bezier curves evaluated at `t = v`. 

Here is the requested screenshot of `bez/teapot.bez`, evaluated using our implementation. 

![t12](images/task2-withwireframe.png) 

We also provide a screenshot without the wireframes. 

![t13](images/task2-without.png) 

<br>

---
<br>

## Task 3: Area-weighted vertex normals

To implement area-weighted vertex normals, we followed the [simple scheme described in lecture](https://cs184.eecs.berkeley.edu/sp22/lecture/6-36/rasterization-pipeline). Concretely,
1. We collected the positions of neighboring vertices with respect to the current vertex by traversing half edges.
2. Then, we computed the surface normal of each face, following the geometric derivation outlined in [this article about surface normals](https://www.khronos.org/opengl/wiki/Calculating_a_Surface_Normal). By the definition of the cross product, the resulting surface normal is weighted by the area of the respective triangle face.
3. Finally, we returned the normal vector for the vertex by summing over the weighted normals of each face, then calling the `unit()` function on the result. 


Here is a diagram that compares the output of `dae/teapot.dae` with and without area-weighted vertex normals for shading.

| Flat Shading       | Phong shading |
| ----------- | ----------- |
| ![31](images/task3-flat.png)   | ![32](images/task3-phong.png)       |


<br>

---
<br>

## Task 4: Edge Flip 

Implementing the edge flip operation requires reassigning pointers for the various mesh elements of the two involved triangles. We found the following [diagram from CMU’s computer graphics course](https://cmu-graphics.github.io/Scotty3D/meshedit/local/edge_flip_diagram.png) extremely helpful as a guide of all the elements that may potentially be affected:

![t41](images/task4-cmu.png)

One observation we made was that the outer halfedges (`h6`, `h7`, `h8`, `h9` in the diagram above) were not affected by the edge flip – all of their pointers would remain the same. Each edge’s halfedge would remain the same as well. Thus, the only elements we had to consider were the four vertices, the two faces, and the six inner halfedges.

As mentioned in the writeup, vertices and faces only have a `halfedge` pointer that may need to be reassigned, whereas halfedges have five pointers: `next`, `twin`, `vert`, `edge`, and `face`. Below are the elements we considered, each labeled with which pointers needed to be reassigned:

| Vertices and faces       | Half edges |
| ----------- | ----------- |
| ![t42](images/task4-vertices.png)   | ![t43](images/task4-halfedges.png)       |

(As a side note, it is possible that the halfedge pointers for vertices/faces remain correct after the edge flip, and that no change is needed. However, since the choice of halfedge is arbitrary for these elements, we wanted to be safe and set them all regardless. For example, if f0’s halfedge had been set to h1 before the flip, then this would no longer be correct after the flip — just in case, we should set it to h0 instead.)

Here’s an example of some edges being flipped on the teapot:

| Before flip      | After flip |
| ----------- | ----------- |
| ![t44](images/task4-beforeflip.png)   | ![t45](images/task4-afterflip.png)       |


<br>

---
<br>

## Task 5: Edge Split

The edge split operation is similar to the edge flip operation in the sense that we need to update the pointers of our mesh elements — however, there is some extra work involved due to the creation of new vertices, edges, and halfedges. Taking inspiration from the diagram from Task 4, we drew a new diagram for what the edge split operation would do (newly created mesh elements are labeled in green):

![t51](images/task5-split-diagram.png)

Since there are a lot more elements involved, we decided to just reassign all the pointers for all elements according to the diagram above, in order to avoid the possibility of bugs. Additionally, we observed that the resulting diagram is made up of four symmetric triangles (the shapes themselves are not necessarily symmetric, but the mesh elements are). So to keep our code more concise, we wrote a helper function to assign the pointers for a single triangle, using the upper left triangle in the diagram as reference, then simply reused it for all the other triangles.

Here is an example of `dae/teapot.dae` before and after some edge splits. The region circled in red was split multiple times, while the region circled in purple was split only once:

| Before split      | After split |
| ----------- | ----------- |
| ![t52](images/task5-before.png)   | ![t53](images/task5-after.png)       |

And here is the same before/after diagram but with a combination of edge splits and edge flips:

| Before splits/flips      | After splits/flips |
| ----------- | ----------- |
| ![t54](images/task5-before.png)   | ![t55](images/task5-after-wflip.png)       |



<br>

---
<br>

## Task 6: Loop Subdivision for Mesh Upsampling

To implement loop subdivision, we roughly followed the pseudo code described in both the spec and in the skeleton for `upsample(..)`. 
1. To start, we compute all updated positions for old vertices, then store those positions in the vertex’s `newPosition` attribute. To compute updated positions, we followed the Loop Subdivision rule discussed in the spec: `(1 - n * u) * original_position + u * original_neighbor_position_sum`.
	* It’s imperative to use floats during the Loop Subdivision rule, so we added floating points to every expression. For example, for the n=3 case, instead of setting `u` to `3 / 16`, it was important to use `3.0 / 16` to avoid `u` being incorrectly set to zero. 
2. Second, we compute the positions for new vertices (i.e., for when we split edges), then store these positions in the edge’s `newPosition` attribute. To compute new positions, we followed the closed formula discussed in the spec: `3/8 * (A + B) + 1/8 * (C + D)`.
	* Similar to (1), it’s imperative to use floats during our calculations, so we added floating points to every expression. For example, instead of multiplying `(A + B)` by `3 / 8`, it was important to use `3.0 / 8` to avoid `(A + B)` being incorrectly set to zero. 
3. Next, we perform edge splits over every edge in the original mesh input. Every edge split returns a new vertex. For each returned vertex, we set its new position to the value stored in the edge’s `newPosition` attribute.
	* To avoid an infinite for loop, we precomputed the number of edges, `N`, in the original mesh (e.g., using `.nEdges()`) before entering the loop. We set a counter, `k`, to track our location in the iterator. Our for loop used two updates like so: `for (EdgeIter e = mesh.edgesBegin(); k < N; k++, e++)`.
4. Then, we flipped any *new* edges that connected an old and new vertex, as described in the spec. 
5. Finally, we copied the precomputed vertex positions (i.e., stored in `v->newPosition`) into each vertex’s actual position (i.e., `v->position`). 

**Sharp corners and edges**

We loaded `dae/cube.dae`, then paid attention to how corners and edges changed as we performed loop subdivision. 

For sharp corners, we noticed that certain corners would move closer to the center of the cube at each iteration. This movement was most drastic after the first iteration of loop subdivision, then gradually moved less at each iteration after that. For edges, one call to upsample would subdivide each edge into more edges at each iteration. For example, after the first iteration, a given edge would split to become two edges. After the second iteration, those two edges split into four edges. After the third iteration, those four edges would split into eight edges (i.e., we observed a branching factor of 2). Combined with the original observation with sharp corners, the original edges appeared to “curve” into a more spherical shape. 

To reduce this effect for sharp corners, we can pre-split edges surrounding a given corner. This has the following effects:
- Recall that the formula to update the position of an old vertex is given by `(1 - n * u) * original_position + u * original_neighbor_position_sum`, where `u = 3 / (8 * n)` (for `n != 3`) and `n` is the vertex’s degree. Logically, if we increase `n` by `1`, then `u` decreases by a factor of `8`. In the formula to update a vertex position, we see that the `1 - n * u` term increases and the `u` term decreases. In other words, the `original_position` will carry more weight, while the sum of neighboring positions will influence the original position less. As a result, the sharp corner vertex will move less towards the center of the cube. 
- Additionally, when we pre-split an edge that is incident to the corner vertex, a new vertex is created that is on the midpoint of the edge. This means that the new neighboring vertex that is used in the weighted average is *closer* to the original position of the corner vertex, again causing it to move less towards the center than it otherwise would have.

For example, we loaded up `dae/cube.dae`, identified a corner vertex, then split all `n` edges exiting that vertex. We repeated this operation again multiple times, resulting in the following mesh:

![t60](images/task6-split-side-view.png)

Then, as we performed loop subdivision, we observed that the corner vertex moved much less at each iteration:

| Without pre-splitting      | With pre-splitting |
| ----------- | ----------- |
| ![t61](images/task6-before.png)   | ![t61](images/task6-before.png)
| ![t62](images/task6-no-split.png)   | ![t63](images/task6-with-splits.png)       |
| ![t64](images/task6-no-split-2.png)   | ![t65](images/task6-with-splits-2.png)       

**Asymmetry of cube**

During loop subdivision, the new positions of vertices are calculated using a weighted average of old vertex positions. Without any preprocessing, the cube becomes asymmetric because there are likely to be vertices whose neighboring vertices are not “balanced.” 

For example, in the diagram below, the vertex circled in red does not contribute to the average for the yellow starred vertex, causing its new position to be biased towards the other directions:

![t66](images/task6-asymmetric-vertex-diagram.png)

To fix this, we can pre-process the cube with edge flips/splits to make sure that each vertex’s neighbors are symmetric in position, like in the picture below:

![t67](images/task6-cube-symmetric-initial.png)

Here is a comparison of the cube shape after repeated subdivisions, with vs. without preprocessing. (The last frame of each GIF switches to a side view of the cube to make the differences clearer.)

| Asymmetric loop subdivision (no preprocessing)      | Symmetric loop subdivision (with preprocessing) |
| ----------- | ----------- |
| ![t68](images/task6-1.png) | ![t69](images/task6-cube-symmetric-initial.png)       |
| ![t6a](images/task6-2.png) | ![t6b](images/cube-symmetric-2.png)       |
| ![t6c](images/task6-3.png) | ![t6d](images/cube-symmetric-3.png)       |
| ![t6e](images/task6-4.png) | ![t6f](images/cube-symmetric-4.png)       |
| ![t6g](images/task6-5.png) | ![t6h](images/cube-symmetric-5.png)       |
| ![t6i](images/task6-6.png) | ![t6j](images/cube-symmetric-6.png)       |
| ![t6i](images/task6-q3-nopreprocess-res.png) | ![t6j](images/cube-symmetric-side.png)       |