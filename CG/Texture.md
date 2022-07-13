# Texture  
we have many modeling techinques by using more triangles, lights, and materials.
But using additional resoures (such as trangle and lights) come with side effects such as memory overheads and additional running time. 
In order to eliminate this effect, Many approximation technique have been developed.
Also, Texture mapping has been one of main approximation techniques.  
One thing that we need to understand is that while textures are originally designed for representing complex shapes of geometry, they can be uitilized for various other purpose. 
In Computer architectures, texture is simplest a 2D array. 
So, we can be readily pre-computed and used at runtime. 
The Z-buffering learned last time used for visibility determination can be considered as a type of a texture  
## Texture Mapping   
Texture is a 2D buffer representing each element and color.
Commonly a Texture refer to a 2D image.
Texture Mapping indicates a mapping from a part of the texture to a part of a model.
Texture coordinates are used like this:(u, v), to locate a particular location of a 2D texture.  
We link the texture location to a particular location or a vertex of a mesh by using 2D or higher geometry coordinates. 
Once we compute the texture coordinate, we compute the 2D indices of the corresponding texel, and use the color of the texel for illumination or other purpose.  

- Perspective-correct interpolation  
A naive interpolation of various vertex attributed in the image space does not provide the expected results that are supposed to be computed by the __object-space interpolation__.
So, the perspective-correct interpolation is developed to correct result even in the image-space interpolation.  
    - _what is the object-space interpolation?_  
    object-space interpolation is a space where texture is mapped focusing on objects. 
    So if the object moves in world space, the texture will also move with the object vertices.

## Oversampling of Texture  
This phenomenon occurs due to various transformations (e.g., modeling and projective transformations) and the angle of the triangle against the view direction.  
Since the pixel in the image space does not match with that in the texture space. we have two cases: oversampling and undersampling. 

- **Oversampling**  
Oversampling refers to the case where the sampling resolution in the texture space is smaller than the available resolution of the texture. 
The Oversampling issue occurs when we magnify the geometry.  

Fotunately, etc. A simple method to solve this issue is to identify the nearest neighbor texel center and use its color for the quadrilateral.
During the rasteriation, we compute colors or other attributes based on center positions of pixels using smapling point location.
But nearest neighbor approach is quite fast, its visual-quality is poor especially along the boundary of texels.
In order words, when two sampling locations are very close, but are in different texels, they get different colors, resulting in visual gaps in the image space.  
Another approach is to use linear interpolation. Given the same smapling location, we identify four nearby texels. 
We than perform the linear interpolation along the U and V texture directions, thus named as bi-linear interpolation.  
We can define it as follows(Bi-linear interpolation).  
___
$\alpha = \frac{x - x_0}{x_1 - x_0}, \; \beta = \frac{y - y_0}{y_1 - y_0}$  
$c = (1 - \beta )((1 - \alpha )c_0 + \alpha c_1) + \beta ((1 - \alpha )c_2+\alpha c_3)$  
___  

The effect of using the bi-linear interpolation can see that boundary texels were smoothened. 
Nonetheless, we can also see that edge information inherent in the original texture was filtered out too.
So, We must consider carefully.


    







