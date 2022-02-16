Q1: Our strategy is reminiscent of rejection sampling. To rasterize a triangle, 
1. Derive a min_x, max_x, min_y, and max_y from the coordinates of the triangle’s vertices. If any of these values exceed the bounds of the frame buffer (e.g., max_x greater than the frame buffer’s width), clamp the point to the edge of the frame buffer (e.g., max_x becomes the framebuffer’s width). 
2. For each pixel in our bounding box, perform the point-in-triangle test, outlined in steps 3a - 3c. Sample the center of each pixel (i.e., x + ½, y + ½).
3. Point-in-triangle test: For each edge of the triangle,
	- Arrange the edge’s start and endpoints in counterclockwise orientation.
	- Perform the line test as described in the lecture to determine if the point is on or inside the edge.
	- If all three edges pass our line test, the point must be inside the triangle.  
	- If a point passes the point-in-triangle test, fill the pixel with the inputted color value. Otherwise, reject the point (i.e., do nothing).
