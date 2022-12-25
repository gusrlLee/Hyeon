# rg-etc1
---
Title : Fast, high quality ETC1 (Ericsson Texture Compression) block packer/unpacker
Author : Rich Geldreich

Date : 2022-12-25
link : https://code.google.com/archive/p/rg-etc1/
code link : https://github.com/richgel999/rg-etc1

---

## Summary 
해당하는 방식은 Rich Geldreich가 작성한 ETC1의 개선 방안을 Google에서 공유한 코드이다. 해당 방식은 2x-28x faster하다고 밝히고 있고 많이 사용되는 code이다. Betsy GPU에서도 이 code를 reference로 참고했다고 밝히고 있다. 해당 방식은 간단하게 보자면 기존 ETC1 algorithm에서 flip/diff combination result로 방식을 하여 3가지의 mode로 encoding을 진행하는 형식이다. 먼저 base color에 대해 average를 고르고 그에 대한 encoding을 진행한 후 table을 이용하여 quality에 따른 optimization step을 통해 produce한다. 

## Contents

### Algorithm
- rg_etc1's compressor has three quality modes: low (but fast), medium (decent speed for single threaded compression on a Core i7 CPU), and high (intended for multithreaded compression).
- These quality modes are intended to be roughly comparable in quality to Ericsson's fast/medium/slow non-perceptual modes.
- rg_etc1 contains a simple, near-optimal solid block compressor that uses several small lookup tables. $\to$ 기존 보다 9%의 개선된 result를 return
- 기존 방식과 동일하게 lookup table이 존재. 
- For non-solid color blocks, rg_etc1 currently uses this algorithm: The outer loop iterates through each flip and differential possibility.
	- non-flipped/differential colors
	- non- flipped/absolute colors
	- flipped/differential colors
	- flipped/absolute colors
- Error Metric : RMSE 
- _본 Algorithm의 과정_
	- the compressor first computes the average subblock color
	- it radix sorts the input subblock colors along the ETC1 intensity (1,1,1) axis.
	- It then scans through a subset of the 3D lattice centered around the quantized,
		- In low quality mode only a single lattice point is checked
		- In medium and high quality modes progressively larger lattice regions are scanned depending on the best error found so far.
	- There are two intensity table/pixel selector evaluation functions used to compute the error resulting from each trial quantized subblock color.
		- The first, faster function (used in low and medium quality modes) enforces a total ordering of pixel selectors along the ETC1 intensity axis (1,1,1).
		- The second, used only in high quality mode, picks the best intensity table/selectors using plain RGB squared distance as an error metric.
- Each time a better solution is found within the lattice scan loop, the compressor attempts to refine the current solution's quantized subblock color using a simple linear optimization based off the current selectors
- this optimization step produces a new subblock "base" color which results in less error.
