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

## Task 3: Transforms (10 pts)

![task 3](/images/Task-3.png)

Cubeman is born to life! Instead of being a boring red statue, Cubeman has become a Gymbro. In this photo, we’ve rotated his arms and legs and transformed their relative positions to make Gymbro do jumping jacks. We also scaled up the relative widths of his arms and legs to show that Gymbro has strong biceps and sturdy quadriceps. He will be grinding in the RSF later today. 

## Task 4: Barycentric Coordinates (10 pts)

The 2D Barycentric coordinate system is a system in which the location of a point is specified by the proportional distances between the vertices of a triangle. For texture mapping, we want to derive texture coordinates on a 2D plane, so we use our rasterized triangle coordinates and Barycentric coordinates to interpolate textures across the triangle. 

Below is an example of Barycentric coordinates in action to interpolate the spectrum of RGB values in a triangle. Here, the triangle’s vertices represent the primary colors, red (upper left), green (upper right), and blue (bottom right). Each point within the triangle is weighted by their distance from the triangle’s vertices. These weights (i.e., `alpha`, `beta`, `gamma`) directly correspond to their RGB value in the color spectrum. 

![task 4](/images/Task-4-2.png)

Here is the requested screenshot for `svg/basic/test7.svg` with default viewing parameters and sample rate 1: 

![task 4](/images/Task-4.png)

## Task 5: "Pixel sampling" for texture mapping (15 pts)

Pixel sampling is the process of sampling from an image signal. In the case of pixel sampling for texture mapping, our goal is to sample color values from a texture file which will be rendered in screen space. For every discrete (x, y) coordinate in screen space, there is a corresponding (u, v) coordinate in the texture from which we can sample.

Our rasterizer handles textures in the form of triangles, similar to the procedure described in Tasks 1 and 2. However, for each (x, y) coordinate, the sampling process is a bit more involved — this is because we are sampling from a texture whose coordinates do not necessarily align with those of the screen itself (the texture may be “warped” relative to screen space, or be a different size than the canvas). To handle this, we use the following steps:

1. Map the coordinate to its corresponding (u, v) location in texture space using barycentric interpolation. Unlike the (x, y) coordinate, the (u, v) coordinate is continuous.
Sample a value from the texture at (u, v) using one of two methods: nearest-pixel sampling or bilinear sampling.

- Nearest-pixel sampling involves simply using the value at the discrete coordinate nearest to (u, v) in texture space. While this approach works, it can lead to aliasing in regions where the texture frequency is too high relative to our sampling rate — we provide an example of this later. 

- To help mitigate this issue, we could instead use bilinear sampling, which samples the four nearest discrete coordinates in texture space, then uses a bilinear interpolation of their color values. Intuitively, this results in smoother color values for our final rendered image. Theoretically, this can be viewed as applying a certain filter to the texture before sampling, which reduces the maximum frequency of the signal and thus reduces aliasing.
