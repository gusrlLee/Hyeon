# SLIC Superpixels Compared to State-of-the-art Superpixel Methods.

---
Title : SLIC Superpixels Compared to State-of-the-art Superpixel Methods.
Author : Radhakrishna Achanta, Appu Shaji, Kevin Smith, Aurelien Lucchi, Pascal Fua, and Sabine Su ̈sstrunk

Date : 2022-12-31
link : https://ieeexplore.ieee.org/document/6205760

---

## Summary
 본 논문은 SegTC에서 사용한 SLIC algorithm에 관한 논문이다. 논문에서는 논문에서 발표한 아이디어와 기존에 존재하는 superpixel에 관한 내용과 성능 분석을 통해 이야기가 시작된다. 간단하게 요약하자면 기존에 존재하는 알고리즘 보다 더 나은 Performance와 quality를 가지고 있어 State-of-the-art라고 한다. 또한 쉬운 알고리즘을 사용하고 있어 메모리 측면과 속도 측면에서 superpixel을 더 compactness하게 생성할 수 있다고 한다. 본 논문의 SLIC의 전체적인 이름은 _Simple linear iterative clustering_ 이다. _k-means_ 를 superpixel generation에 관하여 채택했고 기존과 다른 2가지의 차이점이 존재한다고 한다. 
- The number of distance calculations in the optimization is dramatically reduced by limiting the search space to a region proportional to the superpixel size.
- A weighted distance measure combines color and spatial proximity, while simultaneously providing control over the size and compactness of the superpixels.


## Contents
### Algorithm
- 기존 _SLIC Algorithm_ 은 4개의 step으로 이루어져 있다. 
- 그리고 CIELAB이라는 color space[^1]를 사용한다. 

#### 1. initialization step
- $k$ initial cluster centers $C_i = [l_i \; a_i \; b_i \; x_i \; y_i]^T$ are sampled on a regular grid spaced S pixels apart.
- To produce!roughly equally sized superpixels, the grid interval is $S = \sqrt{N/k}$.
- The centers are moved to seed locations corresponding to the lowest gradient position in a 3 × 3 neighborhood.
- This is done to avoid centering a superpixel on an edge, and to reduce the chance of seeding a superpixel with a noisy pixel.

#### 2. assignment step
- each pixel $i$ is associated with the nearest cluster center whose search region overlaps its location,
- This is the key to speeding up our algorithm because limiting the size of the search region significantly reduces the number of distance calculations.
- This is only possible through the introduction of a distance measure $D$
- $D$ 는 뒤에 이어서 설명. 
- Since the expected spatial extent of a superpixel is a region of approximate size S × S, the search for similar pixels is done in a region 2S × 2S around the superpixel center.


#### 3. update step
- The L2 norm is used to compute a residual error E between the new cluster center locations and previous cluster center locations.
- Once each pixel has been associated to the nearest cluster center, an update step adjusts the cluster centers to be the mean $\left[ l \; a \; b \; x \; y \right]^T$ vector of all the pixels belonging to the cluster.
- The $L_2$ norm is used to compute a residual error $E$ between the new cluster center locations and previous cluster center locations.

#### 4. post-processing step
- a post-processing step enforces connectivity by re-assigning disjoint pixels to nearby superpixels.
- To correct for this, such pixels are assigned the label of the nearest cluster center using a connected components algorithm.

### Distance Measure
- $D$ computes the distance between a pixel i and cluster center $C_k$.
- $$D = \sqrt{d_c^2 + \left( \frac{d_s}{S}\right)^2 m^2}$$
- m also allows us to weigh the relative importance between color similarity and spatial proximity.
- $d_c = \sqrt{\left( l_j - l_i \right)^2}$
- $d_s = \sqrt{\left( x_i - x_j \right)^2 + \left( y_i - y_j \right)^2 + \left( z_i - z_j \right)^2}$

[^1]: 해당하는 것은 CIE x,y,z를 변형한 공간이며, 각 의미하는 바는 CIE L*a*b 색 공간에서 _L_ 값은 밝기를 나타낸다. _L_ = 0 이면 검은색이며, _L_ = 100 이면 흰색을 나타낸다. _a_  빨강과 초록 중 어느쪽으로 치우쳤는지를 나타낸다.