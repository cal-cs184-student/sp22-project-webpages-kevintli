# Project 1: Rasterizer 

**Contributors**: Micah Yong (micahtyong@berkeley.edu) and Kevin Li (kevintli@berkeley.edu)

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

Q1: Supersampling is an anti-aliasing technique that approximates a 1-pixel box filter. It does this by sampling `sample_rate` points within each pixel in screen space, and averaging together their color values. As we increase our sampling rate, we reduce the effects of aliasing like jagged edges and other rendering artifacts. 

We made the following changes to the rasterization process: 
- Resize our sample buffer and frame buffer to width * height * sample_rate.
- Now, whenever we access the sample buffer or frame buffer, multiply the index by `sqrt(sample_rate)`.
- In `rasterize_triangle`, translate the coordinates of our triangle’s input vertices into supersampled coordinates by multiplying each value by `sqrt(sample_rate)`. 
- In `resolve_to_framebuffer`, we take sample_rate samples from the sample buffer for each corresponding pixel in the frame buffer, and compute the average RGB values to get our average color. We then fill the corresponding entry in the frame buffer. 


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

The 2D Barycentric coordinate system is a system in which the location of a point is specified by the proportional distances between the vertices of a triangle. If the vertices of the triangle are $$P_1, P_2, P_3$$, then we can represent any point $$P$$ within the triangle using barycentric coordinates $$(\alpha, \beta, \gamma)$$, where $$P = \alpha P_1 + \beta P_2 + \gamma P_3$$. The barycentric coordinate values are restricted such that $$\alpha, \beta, \gamma \geq 0$$ and $$\alpha + \beta + \gamma = 1$$.

For texture mapping, we want to derive texture coordinates on a 2D plane. Given our triangle vertex coordinates in screen space and their corresponding coordinates in texture space, we can convert any screen space coordinates within the triangle to barycentric coordinates, then interpolate and get the desired coordinates in texture space.

Below is an example of Barycentric coordinates in action to interpolate the spectrum of RGB values in a triangle. Here, the triangle’s vertices represent the primary colors, red (upper left), green (upper right), and blue (bottom right). Each point within the triangle is weighted by their distance from the triangle’s vertices. These weights (i.e., alpha, beta, gamma) directly correspond to their RGB value in the color spectrum. 


![task 4](/images/Task-4-2.png)

Here is the requested screenshot for `svg/basic/test7.svg` with default viewing parameters and sample rate 1: 

![task 4](/images/Task-4.png)

## Task 5: "Pixel sampling" for texture mapping (15 pts)

Pixel sampling is the process of sampling from an image signal. In the case of pixel sampling for texture mapping, our goal is to sample color values from a texture file which will be rendered in screen space. For every discrete (x, y) coordinate in screen space, there is a corresponding (u, v) coordinate in the texture from which we can sample.

Our rasterizer handles textures in the form of triangles, similar to the procedure described in Tasks 1 and 2. However, for each (x, y) coordinate, the sampling process is a bit more involved — this is because we are sampling from a texture whose coordinates do not necessarily align with those of the screen itself (the texture may be “warped” relative to screen space, or be a different size than the canvas). To handle this, we use the following steps:

1. Map the coordinate to its corresponding (u, v) location in texture space using barycentric interpolation. Unlike the (x, y) coordinate, the (u, v) coordinate is continuous.

2. Sample a value from the texture at (u, v) using one of two methods: nearest-pixel sampling or bilinear sampling.

Nearest-pixel sampling involves simply using the value at the discrete coordinate nearest to (u, v) in texture space. While this approach works, it can lead to aliasing in regions where the texture frequency is too high relative to our sampling rate — we provide an example of this later. 

To help mitigate this issue, we could instead use bilinear sampling, which samples the four nearest discrete coordinates in texture space, then uses a bilinear interpolation of their color values. Intuitively, this results in smoother color values for our final rendered image. Theoretically, this can be viewed as applying a certain filter to the texture before sampling, which reduces the maximum frequency of the signal and thus reduces aliasing.

| Nearest-pixel sampling (1 sample/pixel)      | Bilinear-sampling (1 sample/pixel) |
| ----------- | ----------- |
| ![5-1-1](/images/Task-5-1-1.png)      | ![5-1-1](/images/Task-5-1-2.png)       |

| Nearest-pixel sampling (16 samples/pixel)      | Bilinear-sampling (16 samples/pixel)       |
| ----------- | ----------- |
| ![5-1-1](/images/Task-5-1-3.png)   | ![5-1-4](/images/Task-5-1-1.png)        |

The figure above shows a comparison of `svg/texmap/test5.svg` (the Berkeley seal) rendered using nearest-pixel vs. bilinear-sampling, with 1 sample/pixel and 16 samples/pixel each. We see that bilinear sampling results in a more “blurred” image, with significantly less aliasing than the nearest-pixel sampled image. In fact, while supersampling 16 samples/pixel does help reduce aliasing slightly, switching from nearest-pixel to bilinear sampling produces a much larger improvement (even bilinear sampling with just 1 sample/pixel).

The benefits of bilinear sampling are more evident the higher the frequency of the texture is relative to the sampling rate. Practically, this might happen when the texture is very fine-grained compared to the number of pixels that will be rendered on the screen — in other words, if the texture goes through many changes in color values within the area of a single pixel in screen space. When this is the case, bilinear sampling is especially useful because it incorporates more information from the texture by using a weighted average of nearby texture samples, whereas nearest-pixel sampling is limited to the value at a single (u, v) coordinate and is thus more prone to aliasing.

This means, for example, there would typically be a large improvement when using bilinear sampling to render a very zoomed-out texture, because its frequency would be very high relative to the screen space. On the other hand, if we were rendering a lower-frequency texture (such as the solid color region of the parrot svg, shown below), we could get away with using nearest-pixel sampling without much or any reduction in performance.

| Nearest-pixel sampling (1 sample/pixel)      | Bilinear-sampling (1 sample/pixel) |
| ----------- | ----------- |
| ![5-1-1](/images/Task-5-2-2.png)      | ![5-1-1](/images/Task-5-2-1.png)       |

## Task 6: "Level sampling" with mipmaps for texture mapping (25 pts)

Another antialiasing technique we can use for texture mapping is **level sampling**. The motivation for level sampling comes from the fact that aliasing occurs whenever the max signal frequency is greater than half the sampling rate (Nyquist frequency). Ideally, we could avoid such aliasing by sampling each pixel from a signal whose frequency matches the screen sampling rate.

In practice, level sampling approximates this by using a data structure called a **mipmap**, which stores progressively downsampled versions of the texture in powers of two. Then for each (x, y) coordinate, we can simply sample from the mipmap level whose resolution most closely matches the “screen sampling rate” in that region. Lecture 5 provides the exact equations needed to compute the appropriate mipmap level for a given coordinate (shown below) — essentially, the greater the distance traveled in texture space for a given distance in screen space, the higher the mipmap level (lower frequency signal) we should use.

![6-diagram-1](/images/Task-6-Aid.png)

To implement this, we first use barycentric interpolation to find the (u, v) coordinates as well as the changes in texture space per pixel in screen space (`du/dx`, `dv/dx`, `du/dy`, and `dv/dy`). Then, we use the equations from Lecture 5 to compute the appropriate mipmap level as a continuous value. Finally, since the actual mipmap levels are discrete, we must choose one of three techniques for level sampling:
- **Level zero**: always use level zero of the mipmap (the full-resolution texture).
- **Nearest level**: choose the nearest valid integer mipmap level to use.
- **Linear interpolation**: sample once from each of the two nearest mipmap levels, then use a weighted average of their values.

This can be done *in combination with* the other sampling methods too (pixel sampling and supersampling)!

The three sampling techniques each have their own tradeoffs:

|       | Supersampling | Pixel Sampling      | Level Sampling |
| ----------- | ----------- | ----------- | ----------- |
| **Runtime**      | Requires `sample_rate` times more sampling operations, followed by a simple averaging operation. (When sample_rate > 4, this is the most expensive)       | Requires 4x more sampling operations, followed by 3 lerps.      | Requires level calculation and just 1-2 sampling operations (depending on whether we use linear interpolation or nearest/zero level sampling).       |
| **Memory Usage**   | **Higher**. Requires additional sample buffer of size `sample_rate * screen_width * screen_height`. If `(screen resolution * supersampling rate)` is higher than texture resolution, this will generally be more memory-intensive than level sampling.
       | **Lowest**. No additional memory required, since we simply use the existing buffer and texture file.   | **Higher**. Requires mipmaps: let T be the `texture_width * texture_height`. Then, this requires additional buffers of size `T/4 + T/16 + T/64 + … = O(T)`. If texture resolution is higher than screen resolution, this will generally be more memory-intensive than supersampling.        |
| **Anti-aliasing Power**   | Generally very strong antialiasing, since it approximates a 1-pixel box filter. However, this is only useful for textures with sufficient frequency. When doing texture magnification, for example, there isn’t enough resolution in the texture to begin with, so it only helps a bit or not at all — using bilinear pixel sampling would make more sense.        | Slightly effective at antialiasing. However, it’s particularly useful for the texture magnification case, since pixel sampling with bilinear interpolation allows us to produce smoother-looking images when the texture resolution itself is not high enough.   | Very strong antialiasing, since it chooses the best available texture resolution based on the screen sampling rate for each pixel.        |


To show how "Level sampling" operates in combination with "Pixel sampling" from Task 5, we have four versions of Yoho National Park. 

| L_ZERO and P_NEAREST      | L_ZERO and P_LINEAR |
| ----------- | ----------- |
| ![6-1-1](/images/Task-6-Lzero-Pnearest.png)      | ![6-1-1](/images/Task-6-Lzero-Pbilinear.png)       |

| L_NEAREST and P_NEAREST      | L_NEAREST and P_LINEAR       |
| ----------- | ----------- |
| ![6-1-1](/images/Task-6-Lnearest-Pnearest.png)   | ![6-1-1](/images/Task-6-Lnearest-Pbilinear.png)        |