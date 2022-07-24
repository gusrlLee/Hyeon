# RayTracing    
RayTracing and Casting techniques have been introduced late 70's and early 80's computer graphics field as rendering techniques for achieving high-quality images. Let's more detail.   
RayCasting shoots a ray from the camera origin to a pixel and compute the first intersection point between the ray and objects in the scene.
Ray casting then computes the color from the intersection point and ues it as the color of the pixel. 
It computes a direct illumination that has one bounce from the light to the eye.
Its result is same to those of the basic rasterization considering only the direct illumination.   
Ray tracing is an recursive version of the ray casting. 
In order words, once we have the intersection between the initial ray and objects, ray tracing generates another ray or rays to simulate the interaction between and the light and objects.
A ray can be considered as a photon traveling in a straight line, and by simulating many rays in a physically correct way, we can achieve physically correct images.
While the algorithm is very simple.   

## Basic Algorithm 
_Basic Algorithm RayTracing_   
- Trace rays from the eye into the scene. (backward ray tracing)
- identify the first intersection point and shade with it.
- Generate additional, secondary rays needed for shading.
    - Generate ray for reflections.
    - Generate ray for refraction and trannsparency.
    - Generate ray for shadows.   

We first generate a ray from the eye to the scene. While a photon travels from a light source, we typically perform ray tracing in backward from the eye. 
We than identify the fisrt intersection point between the ray and the scene. 
At this point, we simply assume that we can compute such intersection points.   
Reflection and refractions are handled in a similar manner by generating another secondary rays. 
The main quesition that we need to address here is how we can construct the secondary rays for supporting reflection and refraction.
The origin of the reflection ray, $R$, is set to the hit point of the prior ray, and the direction of $R$ is set as the reflection direction.
Most objects in practice do not support such perfect reflection. such as Water and glasses.   
Handling texture mapping efficiently is one of key components for many rasterization techniques running on GPUs. 
On the other hand, ray tracing generates various rays for such effects, and handling rays efficiently is one of key components of ray tracing.   

## Intersection Tests   
Performing intersection tests is one of main operations of ray tracing. 
Furthermore, they tend to become the main bottleneck of ray tracing and thus have been optimized for a few decasdes.   
Any points, $p(t)$, in a ray parameterized by a parameter $t$ can be represented as follows:    
$p(t) = o + t\vec{d}$   
where $o$ and $\vec{d}$ are the origin and direction of the ray, respectively.

## Bounding Volume Hierarchy(BVH)    
A navie approach of computing the first instersection point between a ray and those triangles is to linearly scan those triangles and test the ray-triangle intersection tests.
Many accelation techniques have been proposed to reduce the time spent on ray intersection tests.
In this section, we discuss an hierarchical acceleration technique that can improve the linear time complexity of the naive linear scan method.    
Two hierarchical techniques have been widely used for accelerating the performance of ray tracing.   
- kd-trees   
- bouding volume hierarchies (BVHs)    

kd-trees are constructed by partitioning the space of a scene and thus are classified as spatial partitioning trees.   
BVHs are constructed by partitioning underlying primitives and thus known as object partitioning trees.    

## Bounding Volumes    
We first discuss bounding volums. 
A BA is an object that encloses triangles. 
Also, the BV should be efficient for performing an intersection test between a ray and the BV.
BVs commonly used in pratice are sphere.   
- Axis-Aligned Bounding Box (AABB) 
- Oriented Bounding Box (OOB)
- K-DOPs (Discrete Oriented Polytopes)     

Sphere and AABBs are fast for checking intersection tests against a ray. 
Furthrrmore, constructing these BVs can be done quite quickly.
sphere and AABBs may be too lose BVs, especially when the underlying object is not aligned into such canonical directions or is elongated along a non-canonical direction.    
OBBs and k-DOPs tend to provide tighter bounding, but to require more complex ad thus slow intersection test.

## Construction   
Let's think about how we can construct a bounding volume hierarchy out of triangles.
A simple approach is a top-down construction method, where we partition the input triangles into sets in a recursive way, resulting in a binary tree. (AABBs, BVs)    
We first construct a root node with its AABB containing all the input triangles. 
We then partition those triangles into left and right child nodes.
To partition those triangles associated with a current node, a simple method is to use a 2D plane that partitions the longest edge of the current AABB of the node. 
Once we compute triangle sets for two child nodes, we recursively perform the process until each node has a fixed number of triangles.     
In the aforementioned method, we explaneed a simple partitioning method. 
More advanced techniques have been proposed including optimization techniques with Surface Area Heuristic (SAH). 
The SAH method estimates the probability that a BV intersects with random rays, and we can estimate the quality of a computed BVH.
It has been demonstrate that this kind of optimizations can be slower than the simple method, but can show shorter traversal time spent on performing ray-BVH intersection tests.    

#### Dynamic Models.   
It is important to build or update BVHs of models as they are changing.
This is one of main benefits of using BVHs for ray tracing, since it is easy to update the BVH of a model, as the model changes its positions or is animated.    
One of the most simple methods is to refit the existing BVH in a bottom-up manner, as the model is changing.
Each leaf node is associated with a few triangles.
As they change their positions, we recompute the min and max vlaues of the node and update the AABB of the node. 
We then merge those recomputed AABBs of two child nodes for their parent node by traversing the BVH in a bottom-up manner.
This process has the linear time complexity in terms of the number of triangle.
Nonetheless, this refitting approach can result in a poor quality, when the uderlying objects deform significantly.    

## Traversing a BVH    
Given a ray, we first perform an intersection test between the ray and the AABB of the root node.    
If there is no intersection, it guarantees that there are no intersections between the ray and triangles contained in the AABB.
As a result, we skip traversing its sub-tree.    
If there is an intersection, we traverse its sub-trees by accessing its two child nodes. 
Among two nodes, it is more desirable to access a node which is located closer to the ray origin, since we aim to identify the first intersection point along the ray starting from the ray origin.
Many types of BVHs do not provide a strict ordering between two child nodes given a ray.






