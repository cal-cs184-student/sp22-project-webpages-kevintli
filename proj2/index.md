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