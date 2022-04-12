# Mesh reconstruction with photogrammetry

**Contributors**: Kevin Li, Micah Yong, Ethan Lee, Annie Wang

## Summary

Creating realistic 3D animations is a challenging problem for filmmaking, game development, and a variety of other graphics applications. Our goal is to create a simpler, cheaper workflow for creating such animations by leveraging recent state-of-the-art techniques in 3D reconstruction and keyframe animation. 

## Problem Description

3D animation is complex and requires a large amount of manual effort to create textures for, rig, and animate a 3D model. This results in a fairly high barrier to entry. We envision a simpler process which would allow anyone to create their own animations from a series of camera images. 

Users would go through the following steps:
1. Position their subject in a series of positions, representing keyframes of the animation they would like to create
2. Take pictures from several camera angles for each of these positions
3. Upload these pictures and have our software generate 3D meshes for each frame 
4. Incorporate keyframe animations, lighting, and a variety of other effects 

Recently, there have been many exciting developments in computer graphics that enable faster and more accurate 3D reconstruction from a series of 2D camera images. For example, open-source photogrammetry frameworks like AliceVision [1] provide an easy way to reconstruct arbitrary scenes or objects. We plan to leverage these frameworks in order to get 3D meshes that we can then use for the effects mentioned in Step 4.

We anticipate several challenges in getting this pipeline to work end-to-end:
- Integrating AliceVision into our codebase, which will likely involve doing preprocessing on the user’s input images before reconstruction, and postprocessing to ensure that the 3D meshes are in a format compatible with our animation/raytracing code
- Extending the keyframe interpolation techniques discussed in class to 3D meshes
- Optimizing our raytracing/lighting effects code to work with very complex meshes (or finding ways to reduce the size of the meshes before animating and adding effects, e.g. by downsampling)

## Goals and Deliverables

### What we plan to deliver
Our baseline goal is to generate an animated 3D model for one example (e.g., a person in motion). We will achieve this goal with the following workflow: 
1. First, we must find an image dataset that captures a dynamic object with different camera angles. If we cannot find a dataset that achieves this goal, we will take photos ourselves.
2. Second, we will use AliceVision [1] to reconstruct a 3D point cloud, then a 3D triangle mesh from the images. 
3. We will repeat steps 1 and 2 for three to five keyframes. For example, if we’re simulating a moving person, our results will appear similar to the [top row of this diagram from lecture](​​https://cs184.eecs.berkeley.edu/sp22/lecture/16-42/intro-to-animation) on keyframe animation. With AliceVision, we can generate these keyframes to be physically realistic rather than cutely drawn. 
4. We will implement a keyframe animation technique that extends to 3D meshes as keyframes using keyframe interpolation. We will perform our keyframe animation technique to the three to five meshes that we’ve generated. We may look for external resources for guidance, such as [this summary](​​https://web.arch.virginia.edu/arch545/handouts/keyframing.html). 

To best communicate our results, we will show images of each 3D mesh that we generate. Then, we will show a resulting GIF/video demo that shows the 3D meshes interpolated for our one example. 

We will evaluate our results using the following metrics (non-exhaustive, open for improvement): 
- We will measure how long it takes to generate the dynamic model (i.e., total runtime for steps 1 through 4). 
- We will roughly measure the accuracy of our output by comparing the video result to a similar video or scene from the real world. 
- We will try to estimate the cost to reproduce a similar dynamic model using expensive laser scanners

With our analysis, we hope to answer the following questions:
- If we have widespread access to cheap cameras, is it cheaper to generate 3D point clouds using those cameras rather than laser scanners? How much cheaper? 
- Is it possible to create dynamic motion models using keyframe animation of 3D meshes? Does it make more sense to first generate a 3D mesh, then generate the subsequent keyframes by modifying that mesh?

### What we hope to deliver
If the baseline project goes well and we implement steps 1 through 4 ahead of schedule, then we will test the robustness and generalizability of our algorithm by applying it to several more examples. Some examples may include: 
- A moving car. In particular, it may be expensive to construct a dynamic model of a car that has driften on a highway.
- A suspension bridge. In particular, it may be expensive to construct a dynamic model of a bridge lifting up.
- A person performing a jumping jack (e.g., for a movie or video game). 


## Schedule 

**Week 1** (4/13 - 4/19)
- Set up AliceVision and verify that we can get 3D reconstruction working on a few example objects

**Week 2** (4/20 - 4/26)
- Implement our basic keyframe animation algorithm
- Integrate raytracing/material effects code from Project 3
- Use example meshes outputted by AliceVision to test keyframe animation and various materials and lighting effects

**Week 3** (4/27 - 5/2)
- Turn our existing pipeline into a web or mobile app with a basic UI for uploading images and customizing animations
- Optimize code if needed

**Week 4** (5/3 - 5/10)
- Finalize web/mobile app
- Final presentation on Thurs, 5/5
- Peer review due Mon, 5/10
- Final ACM paper due on Tues, 5/11


## Resources
1. [AliceVision](https://github.com/alicevision/meshroom) is an open-source 3D reconstruction software based on a photogrammetric computer vision frameworks. 
2. [3D Keyframe Animation Summary](https://web.arch.virginia.edu/arch545/handouts/keyframing.html) 
