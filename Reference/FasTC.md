# FasTC
---
Title : FasTC: Accelerated Fixed-Rate Texture Encoding 
Author : Pavel Krajcevski, Adam Lake, Dinesh Manocha
University of North Carolina at Chapel Hill, Intel Corporation, 

Date : 2022-12-26
link : http://gamma.cs.unc.edu/FasTC

---

## Summary 
 본 논문은 QuickETC2에 reference paper로 되어 있어 읽어본 논문이다. 해당 논문은 fixed-rate compression format 안에서 low dynamic range image를 encoding하기 위한 algorithm을 제시한다. 그래서 기존의 동일한 quality를 유지하고 더 빠른 속도로 compression을 진행할 수 있다고 한다. Block Truncatioon Coding을 기반으로 하고 OpenGL의 BPTC format을 사용했다고 한다. 이 논문의 핵심은 endpoint에 대한 approximation을 compute하기 위하여 generalize한 cluster fit을 사용하고 axis-aligned bounding box 를 사용함으로써  texel block의 적절한 partitioning을 estimate하는 것이다. 
 
## Contents 

### Introduction.
- rendering from a compressed representation of the image has become a key tech- nique for both reducing memory footprint and increasing texture access bandwidth.
- These requirements make development of fast, high-quality texture compression methods important.
- FasTC, for generat- ing texture encodings that provide orders of magnitude improve- ments in speed over existing compression methods while maintain- ing comparable visual quality.
- Our approach is applicable to low dynamic range formats that support partitioning by using a coarse approximation scheme to estimate the best partition.
	- Due to the quantization artifacts in the compressed data, this coarse approxi- mation gives a substantial increase in speed while avoiding a severe penalty in quality.
- we use a generalized _cluster-fit_ to find the best encoding for the partition.
	- The benefit of this approach is that we perform linear regression in the quantized space rather than in the continuous or even discrete RGB space.
	- 추후, K-means를 언급함.
- Optionally, we allow the user to refine the data using simulated anneal- ing which provides the ability to set the speed versus quality ratio within a single algorithm.

### Texture Encoding
- The main consideration for a texture compression algorithm is to efficiently produce a stream of bits, or encoding, that when decom- pressed recreates the original image as accurately as possible.
- We split the problem into three parts: 
	- _partition selection_ 
	- _end-point estimation_
	- _endpoint refinement_.
- The goal of a compression algorithm is to compute the best points that lie on this lattice and per-texel indices that together reconstruct the original texel values.
	- Simon Brown’s _libsquish_ uses a method known as a cluster-fit.
	- the 16 texel values pi are first ordered along their prin- cipal axis
	- then each 4-clustering that preserves this ordering is used to solve the following least-squares problem.
	- $$\min_{a,b} \left\vert (\alpha_i \boldsymbol{a} + \beta_i \boldsymbol{b}) - \boldsymbol{p}_i\right\vert$$
- A CPU-based real-time algorithm was first introduced by J.M.P. Van Waveren that computes the diagonal of the axis-aligned bound- ing box (AABB) of the texels in color space.
- The goal of a compression method is to minimize the total error of the com- pressed texels, i.e.
	- $$\min_{\boldsymbol{p}_a, \boldsymbol{p}_b} \Phi (\boldsymbol{p}_a, \boldsymbol{p}_b)$$
- $\Phi(\boldsymbol{p}_a, \boldsymbol{p}_b)$ : _endpoint compression error_
$$\Phi(\boldsymbol{p}_a, \boldsymbol{p}_b) = \left[ \sum^n_{i=0} \sum_{k\in\left\{ r, g, b \right\}} \left\vert \frac{p^k_a(2^\boldsymbol{I} - d_i - 1) + p^k_b d_i}{2^\boldsymbol{I} - 1} - p^k_i\right\vert\right]$$
- This minimization problem is a special case of the quadratic integer programming problem, and it can be reduced to computing the shortest vector on a lattice(SVP).

> 기존 ETC2 Texture Compression의 K-means clustering 기법을 사용하는 이유가 이 논문의 영향이 존재하는 것 같다.


### System Overview (Figuare 2 참고)
- The first step of the algorithm is to check whether or not all of the texels are uniform or transparent.
- Next, for each subset in the partition, we perform another uniformity check and approximate the best endpoints to use
- an optional refinement step that searches for better solutions using simulated annealing

### Choosing a Partitioning.
- The quality of a shape with respect to the block’s texels is deter- mined by the interrelationships of the texels in each subset and the input parameters to the compression algorithm.
- Mathematically, the collinearity of the texels in RGB space would be a good measurement of their ability to be approximated in this way.
- However, this method for shape estimation has the disadvantage that we must compute eigenvalues for each of the hundreds of possible shapes in our compression format for each block.
- Due to the discrete nature of endpoint optimization, any approxi- mation that relies on the continuity of the domain may introduce inaccuracies.
- Instead of performing principal component analysis, we propose an approximation to each subset to be the amount of error it would create if it were encoded using the real-time CPU based method introduced by Waveren, which we will refer to as bounding-box estimation.
- It also provides an efficient check for uniformity by measuring the length of the diagonal.

### Endpoint Estimation.
- Brown’s cluster-fit algo- rithm searches over all 4-point clusterings that preserve the points’ ordering along the principal axis.
- The core of Brown’s algorithm is to assign each texel of a cluster to an interpolation point along the line segment.
- present the generalized cluster fit:
- $$\lVert c_{d_i} - p_i \rVert \leq \lVert c_k - p_i \rVert$$
- approximatethesolution to the endpoint optimization problem as the pair $(p_a, p_b)$ that best approximates the overdetermined system of $n$ equations $\alpha_i p_a + \beta_i p_b = p_i$ where:
- $$(\alpha_i, \beta_i) = \left( \frac{2^I - d_i - 1}{2^I - 1}, \frac{d_i}{2^I - 1}\right)$$

### Endpoint Refinement
- After estimating the endpoints for a given block of texels, we have introduced errors due to quantization.
- Nearby lattice points to these endpoints can provide better approximations to the endpoint optimization problem.
- Most compressors include a localized search around the estimated endpoints to improve their approximation.
- The size of this search space proves to be another limiting factor in the speed of compression algorithms.
- we use simulated annealing to do a local search for the best endpoints in each subset.
- Simu- lated annealing is particularly well suited for this problem due to the size and discrete nature of the search space.
- For each step of simulated anneal- ing, we choose neighboring lattice points $q'_a,q'_b$ for each of the two endpoints.
- If the approximation to the endpoint optimization problem $\Phi(q'_a , q'_b )$ is better than the initial approximation, then we restart the annealing process with the improved approximation.
- if the approximation is close enough to the one we cal- culated in the previous annealing step, we continue the annealing with this approximation.
- close enough를 측정하는 방식은 아래와 같다.
	- $$Accept(\epsilon, \epsilon', \gamma) = e^{\frac{\epsilon - \epsilon'}{10 \gamma}}$$
- if this function is too restrictive, then there is a high chance that the annealing process will get stuck in a local minimum.
- Conversely, if the function is too permissive, the annealing might go in directions that have a catastrophically large amount of error.

### Multi-Core Parallelization.
- The most common ap- proach is to spawn as many threads as the host machine has cores and divide the number of blocks evenly across all threads.
- the algorithm that we have presented uses uniformity checks and other tests to attempt to resolve the encoding of a block as quickly as possible.
- in this way will thus not have every thread finish its work at the same time.
- In order to fully utilize the multiple CPU cores, we use a worker queue for processing textures.
- but for each thread we only process as many blocks as will fit in L1 cache.