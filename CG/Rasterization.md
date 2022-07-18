# Rasterization  
The main idea of rasterization is to project a triangle into view space and rasterize it into fragments in the color and depth buffers. 
In this chapter, we assume that vertices of the triangle are projected int the view space, after they undergo various transformations.  

## Primitive Rasterization  
For the rasterization process, We commonly use triangles as input primitives, mainly because it is the simplest polygon and simplifies the rasterizaiton process. Rasterization process has two main goals.  

1. pixel coverage determination.  
2. parameter interpolation.  

First, Given a pixel of the color buffer, we determine whether the pixel belongs to a given triangle or not.
Once the pixel is covered by the triangle, we also need to compute its color or other parameters such as its depth value for the depth buffer.  
For the coverage problems, There are many ways. 
- One is to check whether the center of a pixel is inside of a triangle. $\to$ based on a point sample
- Another is to measure an area coverage ratio of a pixel against the triangle. $\to$ based on area computation

While the area based computation is more correct, the sample based approach is more efficient, 
and thus is commonly adopted for rasterization process. Usually, we use mixing the advantages of the two methods.   

## ScanLine based triangle Rasterization
The those days, the memory was very expensive, and thus having the full resolution of color and depth buffers is not preferred. 
Instead, these scaneline based and approaches maintain a scanline and incrementally update the scanline to raster the whole triangle. 
Specially, we rasterize an input triangle from top to bottom.
Once we meet a vertex of the triangle, we setup the scanline information.
For the Next scanline, we incrementally update those starting and end coordinates by utilizing slope information of two edges starting form the vertex. 
It was identified to show poor scalability to handle scenes with many triangles, since this technique relies on expensive sorting operations and is not friendly for parallelization.

## Rasterization with Edge Equations
In this section, we discuss a rasterization technique for triangles based on edge equations. 
we will see that this approach is simply and friendly for parallelization, to achieve a high performance and thus handle a scence with many triangles.   
Let's us first compute an edge equation given two vertices, $\dot{v_0}$, and $\dot{v_1}$, of a triangle.
Our goal is to construct an edge equation, $\bar{e}$, whose normal vector heads towards the inside of the triangles.   
Overall, coefficients of the edge equation is given by the cross product between those two vertces.   
$\bar{e} = \dot{v_0} \times \dot{v_1}$  
$= [x_0\; y_0\; 1] \times [x_1\; y_1\; 1]^t$  
$= [(y_0 - y_1) (x_1 - x_0) (x_0y_0 - x_1y_0)]$  
$= [A\; B\; C]$  




