# SegTC: Fast Texture Compression using Image Segmentation.
---
Title : SegTC: Fast Texture Compression using Image Segementation.  
Author : P. Krajcevski and D. Manocha  
The University of North Carolina at Chapel Hill  
  
Date : 2022-12-28  
link : https://diglib.eg.org/handle/10.2312/hpg.20141095.071-077  

---
## Summary
 본 논문은 기존 FasTC에서 Image Segementation 기법을 추가하여 image의 quality를 유지함과 동시에 performance를 개선한 연구이다. 해당하는 논문은 FasTC에 partition하는 부분을 image segmentation기법을 추가하여 조금 더 기존 색상과 가깝게 color를 선택하는 방식을 채택했다. 본 논문에서는 Segementation을 위해 SLTC 기법을 사용했으며 Superpixel이라는 개념이 등장한다. Background 지식을 쌓고 이전 논문인 FasTC를 읽고 난 뒤 읽는 것을 추천 한다. 

## Contents
### Introduction.
- It is important to design techniques that can result in faster iteration times to support the hardware resources.
- Compressed textures with small memory footprints have many advantages such as cache coherency during rendering, allowing more textures to be stored in GPU memory, and reduced memory bandwidth for data access.
- we need improved algorithms for fast and high quality texture compression.
- The per-block task of an encoding algorithm becomes twofold: (본 논문의 방향성.)
	- Select a partitioning out of a predefined set.
	- Choose parameters for each subset of the partitioning.
- ==Our method first computes a segmentation of the image into _superpixels_ to identify homogeneous areas based on a given metric.==
- ==Next, we use a vantage point tree to find the near- est matching partition based on Hamming distance.==
- We observe up to a 6x performance increase on existing single-core compression implementations $\to$ 본 논문의 성과.

### Segmentation Images for Texture Compression.
#### - Overview
- First, we segment the texture into superpixels
- The second stage of our approach computes compression parameters for each subset using the cluster-fit algorithm.
- this subset of la- bels is used as the target for a nearest-neighbor search of the available P-shapes.

#### Segementation.
- In this section, we describe the segmentation algorithm that underlies the P-shape selection method.
- The goal of image segmentation is to apply a labeling to each pixel such that pixels sharing a common label all share a common property or visual characteristic.
- segmenting the image into large contiguous regions of pixels known as superpixels.
- We find that partitioning blocks by superpixel boundaries is suitable for texture compression.
- We use a superpixel segmentation method known as SLIC, or **Simple Linear Iterative Clustering**.
- Once the clustering is computed, any pixels that are not contiguously connected to their cluster centers then ’push’ the superpixel border to connect the components.
- error metric of SLIC $\to$ L2 Norm distance (CIELAB space)
$$d(p_1, p_2) = \alpha \lVert (x_1, y_1) - (x_2, y_2)\rVert_2 + \beta \lVert (L_1, a_1, b_1) - (L_2, a_2, b_2)\rVert_2$$
- Different error metrics may provide better compression values for specific textures because of the high variability of information types stored in textures.

#### P-Shape Selection.
- Once the target partitioning has been selected from the seg- mented image, we proceed by finding the best P-shape defined by our compression format that matches it.
- In order to properly compare one partitioning to another, we must define an error metric between partitionings.
- we can ac- celerate the search by using a data structure to perform effi- cient nearest neighbor lookups using the metric.

#### Block partitioning metric
- Partitionings are represented by a per-pixel labeling, but each label’s compression parameters are computed independently.
- Two partitionings with labels $p$ and $q$ are identical if:
	- $$p_i = p_j \iff q_i = q_j\; \forall \; i, j$$
- we must first find a relabeling $R$ from unique labels in $p$ to unique labels in q such that the Hamming distance $R(x)$
- we compute the optimal relabeling as a bi- partite matching problem which can be done in polynomial time.

#### Vantage Point Trees
- In order to perform nearest neighbor lookups we use a vantage-point tree or VP-Tree.
- We use the VP-Tree because of its ability to handle high dimensional queries along with its $O(logN)$ query complexity.
- The VP-tree is a binary tree where each node is a P-shape $p$, and the left child $p_l$ contains all of the P-shapes within a radius $r$ of $p$ while the right child $p_r$ contains all of the P-shapes outside of this radius.