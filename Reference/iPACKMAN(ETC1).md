# iPACKMAN(ETC1)
---
Title : iPACKMAN: High-Quality, Low-Complexity Texture Compression for Mobile Phones
Author : Jacob Ström and Tomas Akenine-Möller
Ericsson Research, Lund University

Date : 2022-12-22
link : https://diglib.eg.org/handle/10.2312/EGGH.EGGH05.063-070

---
## Summary
해당 논문은 기본적인 PACKMAN에서 품질을 개선 시킨 ETC1의 논문이다. 기본적으로 PACKMAN의 구조를 두개를 사용한 방법이라고 생각하면 이해하기 편하다. 총 Bit는 64bit로 이루어져 있으며 2개의 Base color, 2개의 code book index, Diff bit, flip bit, index table 으로 이루어져 있다. 
기존의 PACKMAN의 제한적인 표현을 개선 시킨 방식이라고 생각하면 된다. 특징으로 가지는 것은 Mobile Device를 Target으로 하는 Texture compression format이다.

## Contents

### 1. Introduction
- Our new texture compression scheme was originally tar- geted for mobile phones, but is in no way limited to those platforms, and can thus be used for PC graphics cards and game consoles as well.
- The basic idea of iPACKMAN is to use larger blocks, 4 x 4 pixels instead of 2 x 4 for PACKMAN.
- it gives greater opportu- nities to obtain better image quality (or compression rate)
	- spatial redundancy in a larger area can be exploited.

### 2. Review of PACKMAN Texture Compression.
- 본 논문에서는 간단하게 PACKMAN texture compression technique에 관하여 설명을 진행한다. 
- The texture image is split into 2 x 4 blocks, where each block is represented by 32 bits. $\to$ _Base Color_  
- A table consists of only four different numbers, and so each pixel index needs two bits for choosing which constant to use. $\to$ 추후 연구인 ETC2에서 T/H, P-mode 로 인해 변경 됨.
- PACKMAN hardware decompression.
	- The 12-bit base color is expanded from 4 bits per color component to 8 bits.
	- The 4-bit table codeword is used to pick a specific table of four numbers from the codebook of tables.
	- The 2 pixel index bits associated with the pixel are used to choose a modifier value from the table
	- The final step computes the final decompressed color by adding the modifier value to the expanded base color.

### 3. $i$PACKMAN Texture compression.
- where the chrominance shifts slowly over the blocks, chrominance banding can be visible, $to$ 해당하는 단점을 확인할 수 있었기 때문에 ETC1에서는 이런 점을 개선하려고 노력.
- One way to combat these chrominance banding artifacts is to improve the color representation for slowly changing areas.
- 본 논문에서는 RGB444에 Data에 한계점을 지적하며 어떻게 하면 더 많은 색상을 표현할 수 있을까 라는 의문을 제시 $\to$ 20장의 test image를 이용하여 average를 측정 해본 결과 [-4:4] range에 88%의 Data들이 모여있는 것을 확인.(Figure 2 참고)
- 따라서 RGB555로 표현을 하고 difference를 두어 dRdGdB를 333Bit로 표현 $\to$ 3bit => [-4:4]
- an overwhelming majority of the blocks can be coded differentially using three bits per color component.
- one bit must be preserved that determines whether we use differential coding in the 4 x 4 block or not.
- 하지만 이렇게 1bit를 설정하는 것은 image quality가 하락하는 것을 확인할 수 있었음. $\to$ 하지만 0.2dB의 감소이기 때문에 본 논문에서는 무시한다고 함.
- We use this bit to indicate whether the subblocks are vertically oriented (two 2 x 4 blocks side by side) or if they are horizontally oriented (two 4 x 2 blocks on top of each other). ==(논문의 핵심)==

### 4. $i$PACKMAN Decompression.
- Decompression (Figure 3 참고) $\to$ 그림에 직관적으로 설명이 잘 되어 있음.
	- 먼저 해당하는 Diffbit를 참고하여 RGB Data의 Format을 확인.
	- Diff bit에 따른 RGB Data의 연산을 진행하고 Quantization 되어 있는 RGB를 888bit로 convert
	- 이에 converted RGB Data를 가지고 Pixel indices를 따라 code Table bit를 참고하여 decompression 진행.
- PACKMAN과 동일하게 $i$PACKMAN도 code book을 가지고 있음. $\to$ 쉬운 Decoding.

### 5. $i$PACKMAN compression.
- 2개의 subblock은 independent 함.
- 본 논문에서는 Compression을 빠르게 하기 위하여 여러 실험을 수행.
	- The fastest of them starts out by quantizing the aver- age colors of the subblocks to 555 bits, and computes the difference between these.
	- It then tries all tables and modifier values for each subblock, and uses the parameters that gives smallest error compared to the original image. Finally, the same proce- dure is carried out with the block flipped,
	- due to the fact that the luminance is later mod- ified, it is not certain that the 555 color closest to the base color is the best quantization.
	- Therefore, in our second com- pression approach, all color pairs within ±1 quantization steps are searched. $\to$ very slowly.

### 6. Error Metric
- 해당 본 논문은 아래와 같은 최종적인 Error Metric를 사용.
- $$e^2_{percept}(u,v) = w^2_r(u_r - v_r)^2 + w^2_g(u_g - v_g)^2 + w^2_b(u_b - v_b)^2$$
- 위의 식을 보면 단순한 weight를 이용하여 distance를 구하는 방식으로 이루어짐.
- 사람의 눈은 각 color마다 다른 weight를 가지고 있기 때문에 여기서도 동일하게 distance에 weight를 부여.