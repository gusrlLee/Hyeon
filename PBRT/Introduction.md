# 1. Introduction.
### 1.1 Literate Programming
- 해당하는 책의 표현 방법에 대해 설명.

---
### 1.2 Photorealistic Rendering and the Ray-Tracing Algorithm.
- Ray Tracing Algorithm is based on following the _path of a ray_ of light through a scene as it interacts with and bounces off objects in an environment.
- Almost Ray Tracing systems simulate at least the following objects and phenomena:
	- Cameras 
	- Ray-object intersections
	- Light Sources
	- Visibility
	- Surface scattering
	- Indirect light transport 
	- Ray Propagation.

#### 1.2.1 Cameras.
- One of the simplest devices for taking photographs is called the _pinhole camera_
- _pinhole camera_ consist of a light-tight box with piece of photographic paper that is affixed to the other end of the box 
- we will refer to the region of space that can potentially be imaged onto the film as the _viewing volume_.
	- 사진에서 두 개의 pyramid가 존재한다고 언급.
- When the film (or image) plane is in front of the pinhole, the pinhole is frequently referred to as the _eye_.
	- 기존 real world에서 말고 pinhole이 eye가 되고 eye 앞에 Film이 있다는 가정으로 simulation $\to$ 더 편리
- Chapter 6에서 더 자세하게 함.

#### 1.2.2 Ray-Object Intersections.
- To find the intersection, we must test the ray for intersection against all objects in the scene and select the one that the ray intersects first.
- Given a ray , we first start by writing it in _parametric form_:
$$ r(t) = o + td,$$
- $o$ is ray's origin, $d$ is its direction vector, and  $t$ is a parameter whose legal range is $(0, \infty)$.
- 위의 식을 이용하여 ray equation을 구할 수 있음 $\to$ 본 책에서는 sphere을 예시로 함.
	- 해당하는 sphere의 equation을 이용하여 $r(t)$  를 대입. 해당하는 $t$에 대한 연립방정식을 풀어 근의 공식을 이용. $\to \; \sqrt{b^2 - 4ac}$ 를 이용하여 intersection point 판별.
- intersection point만 가지고는 정확한 Ray Tracer의 역할을 하지 못함. $\to$ 추가적인 정보 필요.
	- First, a representation of the material at the point must be determined and passed along to later stages of the ray-tracing algorithm.
	- Second, additional geometric information about the intersection point will also be required in order to shade the point.

#### 1.2.3 Light Distribution.
- we need to know how much light is _arriving_ at this point.
- This involves both the _geometric_ and _radiometric_ distribution of light in the scene.
- 윤성의 교수님의 "Rendering" 이라는 책에서 irradiance라는 chapter에서 공부한 내용.
- differential power per area $dE$ (the _differential irradiance_)
$$dE = \frac{\varPhi \cos \theta}{4 \pi r^2}$$

#### 1.2.4 Visibility.
- Each light contributes illumination to the point being shaded only if the path from the point to the light’s position is unobstructed.
- Fortunately, in a ray tracer it is easy to determine if the light is visible from the point being shaded.
- _shadow rays_.
	- a new ray whose origin is at the surface point and whose direction points toward the light.
- If there is no blocking object between the light and the surface, the light’s contribution is included.

#### 1.2.5 Surface Scattering.
- _bidirectional reflectance distribution function_ (BRDF).
- This function tells us how much energy is reflected from an incoming direction $\omega_1$ to an outgoing direction $\omega_0$.
- BRDF 
```cpp
for each light: 
	if light is not blocked: 
		incident_light = light.L(point) 
		amount_reflected = surface.BRDF(hit_point, camera_vector, light_vector) 
		L += amount_reflected * incident_light
```

#### 1.2.6 Indirect Light Transport 
- Turner Whitted's original ray tracing emphasized its _recursive_ nature, which was the key that made it possible to include indirect specular reflection and transmission in rendered images.
- _light transport equation_ (also often known as the _rendering equation_)
