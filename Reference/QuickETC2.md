# QuickETC2: Fast ETC2 Texture Compression using Luma Differences.
---
Title : QuickETC2: Fast ETC2 Texture Compression using Luma Differences.
Author : JAE-HO NAH
LG Electronics, South Korea.

Date : 2022-12-25
link : https://dl.acm.org/doi/abs/10.1145/3414685.3417787

---

## Summary
해당 논문은 기존의 ETC2의 performance를 증가시키는 방법을 연구한 논문이다. 해당 방식은 heuristics한 방식을 사용하여 빠른 연산으로 최고 학회에 등재된 논문이다. 기존 ETC2의 texture Compression을 유지한 상태로 해당하는 연산을 100배 진행하여 기존 안드로이드 정규 texture compression format으로 사용 가능한 방식으로 채택되어 있다. 기존 방법에서는 RGB Data를 luminance 값으로 변경하여 그에 따른 값들을 이용하여 ratio를 설정한 후 그에 따른 compression을 진행한다. 하지만 performance를 위해 P mode와는 error check를 하지 않고 기존 ETC1과 T/H mode만 error를 비교하여 속도에 초점을 맞춘 방식이라고 생각하면 된다. 

## Contents

### Introduction.
- Compressed textures are indispensable in most 3D graphics applications to reduce memory traffic and increase performance.
- we introduce two new compression techniques, namedQuickETC2. The first technique is an early compressionmode decision scheme.
- by exploiting the luma difference of the block to reduce unnecessary compression overhead.
- The second technique is a fast luma-based T- and H-mode compression method
- we replace the 3D RGB space with the 1D luma space and quickly find the two groups that have the minimum luma differences.
- 본 논문의 연구의 초점.
	- To reduce the ETC2 overhead, we present an early compressionmode decision scheme that prevents unnecessary duplicated tests.
	- To support the full ETC2 mode (T, H, and planar) without significantly increased costs, we present a new fast T-/H-mode compression algorithm.

### EARLY COMPRESSION-MODE DECISION.
- if we can determine proper compression mode(s) in advance according to the luma difference of a block, we can avoid duplicated compression
- 본 논문의 방식
	- We first calculate the luma values of each pixel in a block
	- find the min/max luma values in the block.
	- 그리고 4단계로 지정.
		- very-low contrast
		- low-contrast
		- mid-contrast
		- high-contrast
	- For very-low contrast blocks, we use the planar mode
	- In the case of low-contrast blocks, we check whether a block can be smoothly expressed by the base colors at the corners of the block.
	- We compress mid-contrast blocks in the ETC1 mode.
	- High-contrast blocks may create block artifacts, so we perform both the ETC1 and T-/H-mode compression.
		- at the late encode selector stage, we compare the errors in the ETC1 and T/H modes and select the block with a lower error value.
	- SIMD instruction인 SSE/AVX2를 사용하여 CPU에서 가속 연산을 진행.

### Our approach.
- 본 논문에서의 new fast T-/H-mode에 대한 compression algorithm을 소개.
- First, we replace the 3D RGB space on pixel clustering with the 1D luma space
- Second, we select a proper mode in advance,
- Third, we reduce the number of distance candidates of a block
- Fourth, we implement the error calculation part using SSE/AVX2 intrinsics to exploit SIMD parallelism
- Finally, we only use one base-color pair and do not allow additional iterations with multiple base-color pairs,
- 각 단계별 더 자세한 내용은 아래처럼 수행되어짐. (Fig.4 참고)
	- First, we sort the pairs of a luma value and a pixel index in an input block, in ascending order of luma values.
	- Second, to find two proper clusters, we calculate the minimum value of the 15 summed luma differences in the line
	- Third, based on the two clusters generated from the above lumabased clustering, we set a proper compression mode of the block before error calculations.
	- Fourth, we calculate two base colors from the two clusters. For the calculation, we apply different strategies for the two following cases
	- Fifth, to reduce the number of error-calculation iterations using the distance table in the ETC2 format
	- we find the best candidate with the minimum error by changing the distance value.
