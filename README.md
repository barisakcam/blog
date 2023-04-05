# Surface Rendering

### Introduction

The purpose of this program is to render Bezier surfaces with different triangulation sample rates. The program uses an input file containing 1-5 light sources and up to 36 Bezier surfaces. Each surface is triangulated with an amount of 10 samples per edge by default. In such a case, a surface contains 10x10 vertices and 9x9x2 triangles. The program allows modification of this sample rate using "W" and "S" buttons in a range from 2 to 80. The rotation on horizontal axis of the final object is possible with "R" and "F" buttons. User can increase the size of the object on x-y axises without modifying z values by using "E" and "D" buttons. I also configured "Q" and "A" buttons to switch between wireframe and standard versions of the surface.

### Implementation

Before starting coding, I had to configure my environment to run OpenGL programs. I am using a Windows 11 machine with Ubuntu WSL2. Since I was previously using WSL2 for development, I decided to keep using it. Although I have a RTX3050Ti GPU, I could not manage to use OpenGL 4.6. After a day spent searching a reliable method to use OpenGL on WSL2, I managed to set up Mesa implementation of OpenGL 4.2. This most probably causes major performance limitations but it did not cause problem for this project.

I started my implementation from an example OpenGL code provided in class. The first thing I did was to set up view and projection matrices correctly and draw a single triangle. After dealing with that and also parsing the input, I started to design how to triangulate and send the surfaces to the GPU. Since each surface is identical except for the control points, I decided to use "glDrawElementsInstanced" call which draws the same element multiple times. Also since coordinates of vertices were not defined at CPU side, I only provided indices I generated from a simple triangulation algorithm I created, in a single vertex buffer object. Triangulation algorithm starts creating triangle indices starting from the top left corner of the surfaces when we look from the positive z axis. Since I used "glDrawElementsInstanced", focusing on a single surface was enough for any work done at CPU side.

![Triangulation method](/docs/assets/triangulationv2.png)

Lastly, uniform variables such as light positions and control points are managed at CPU side. Also a simple FPS counter to print current sample rate and FPS is implemented. The important parts of the program are implemented in shaders. The vertex shader takes various uniform variables such as control points and calculates vertex position with respect to "gl_VertexID". There are two ways to do Bezier calculations. First one is to use matrix multiplications like in Hermes curves and the other one is Bernstein polynomials. Although the second one is simpler and most probably faster, I used the first approach to make normal calculations simpler.

![Bezier formula](/docs/assets/bezierformulea.png)

![Normal calculation](/docs/assets/derivative.png)

After calculating coordinates for vertices, the scaling operations are applied to them to make sure that the final object fits in [-0.5, 0.5] interval. Also coordinate multiplication from user input ("E" and "D" buttons) is applied here. After these operations, each surface instance reside at the center of the coordinate system. Necessary transformations to reposition these surfaces are done using "gl_InstanceID" value. After that comes the normal calculations. There are 9 versions of Bezier surface equation in vertex shader, implemented as different functions. These consist of functions for Q(s, t), dQ(s, t)/ds and dQ(s, t)/ds, 3 functions for x, y and z coordinates for each of them. Thererefore, normals can be easily obtained by calculating the cross product of partial derivatives.

At fragment shader Lambertian diffuse shading and Phong shading are calculated. There is nothing special except that I used "gl_FrontFacing" value to reverse vertex normals when back faces of triangles are visible. Thanks to that, back faces of triangles are also coloured.

### Testing

For testing, I was planning to make a FPS-samples comparison but my program is currently not affected by the sample amount. The issue might be that I am using Mesa implementation on Ubuntu WSL2 or the problem is just too simple to cause framerate changes. Here are results for the example inputs:

![input1.txt](/docs/assets/input1.png) ![input1.txt wireframe mode](/docs/assets/input1w.png)
 
![input2.txt](/docs/assets/input2.png) ![input2.txt wireframe mode](/docs/assets/input2w.png)

![input3.txt](/docs/assets/input3.png) ![input3.txt wireframe mode](/docs/assets/input3w.png)

A strange bug I discovered during testing is that the triangulation gets distorted when sample count is 66 or 74. It only happens at these numbers independent of the input given in my environment. I suspect my driver issues might be causing that so I decided to not focus on it but it is a good example that OpenGL works in mysterious ways.

