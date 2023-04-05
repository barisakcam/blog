# Surface Rendering

### Introduction

The purpose of this program is to render Bezier surfaces with different triangulation sample rates. The program uses an input file containing 1-5 light sources and up to 36 Bezier surfaces. Each surface is triangulated with an amount of 10 samples each edge by default. In such a case, a surface contains 10x10 vertices and 9x9x2 triangles. The program allows modification of this sample rate using "W" and "S" buttons in a range from 2 to 80. The rotation on horizontal axis of the final object is possible with "R" and "F" buttons. User can increase the size of the object on x-y axises without modifying z values by using "E" and "D" buttons. I also configured "Q" and "A" buttons to switch between wireframe and standard versions of the surface.

### Implementation
