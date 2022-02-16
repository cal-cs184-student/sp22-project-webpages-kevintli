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


## Task 2: Antialiasing by Supersampling (20 pts)

Q1: Supersampling is an anti-aliasing technique that approximates a 1-pixel box filter. As we increase our sampling rate, we reduce the effects of aliasing like jagged edges and other rendering artifacts. 

We made the following changes to the rasterization process: 
Resize our sample buffer and frame buffer to `width * height * sample_rate`.
Now, whenever we access the sample buffer or frame buffer, multiply the index by `sqrt(sample_rate)`.
In `rasterize_triangle`, translate the coordinates of our triangle’s input vertices into supersampled coordinates by multiplying each value by `sqrt(sample_rate)`. 
In `resolve_to_framebuffer`, we take NxN samples from each pixel and compute the average RGB values to get our average color. We then fill the corresponding entry in the frame buffer. 

Q2: Side-by-side examples from `basic/test4.svg` are shown below. 
- The leftmost image has a sampling rate of 1. This image contains the most “jaggies” and no smoothing since a pixel’s color is either present or not present. This decision depends on whether the center point is in the triangle. 
- The middle image has a sampling rate of 4. This image contains some rugged edges and some smoothing since a pixel’s color varies based on how many of the 4x4=16 samples are inside the triangle. 
- The rightmost image has a sampling rate of 16. This image contains almost no “jaggies'' and appears to be the most smooth since a pixel’s color varies based on how many of the 16x16=256 samples are inside the triangle. This calculation is much more dense and accurate for each pixel. 

|    1x1    | 4x4 | 16x16     |
| ![task 2-1](/images/Task-2-1.png)      | ![task 2-4](/images/Task-2-4.png)        | ![task 2-1](/images/Task-2-16.png)    |