# Project 1: Rasterizer 

## Task 1: Drawing Single-Color Triangles (20 pts)

Our strategy is reminiscent of [rejection sampling](https://en.wikipedia.org/wiki/Rejection_sampling). To rasterize a triangle, 

1. Derive a `min_x`, `max_x`, `min_y`, and `max_y` from the coordinates of the triangle’s vertices. If any of these values exceed the bounds of the frame buffer (e.g., `max_x` greater than the frame buffer’s width), clamp the point to the edge of the frame buffer (e.g., `max_x` becomes the framebuffer’s width). 
2. For each pixel in our bounding box, perform the **point-in-triangle** test, outlined in steps 3a - 3c. Sample the center of each pixel (i.e., x + ½, y + ½).
3. Point-in-triangle test: For each edge of the triangle,
	- a. Arrange the edge’s start and endpoints in counterclockwise orientation.
	- b. Perform the line test as described in the lecture to determine if the point is on or inside the edge.
	- c. If all three edges pass our line test, the point must be inside the triangle.  
	- d. If a point passes the point-in-triangle test, fill the pixel with the inputted color value. Otherwise, reject the point (i.e., do nothing).

Once we finish performing step 3 for each pixel in our bounding box, the triangle rasterization process is complete. The result is a triangle, shaded with the input `color`. 

Steps 1 and 2 create a bounding box for the input triangle, so we don’t have to sample every pixel in the frame buffer. As a result, our algorithm performs the same as one that checks each sample within the bounding box of the triangle. An example from `basic/test4.svg` is shown below: 


![Triangles from Task 1](/images/Task-1.png)
