# Texture Compression Techniques
---
Title : TEXTURE COMPRESSION TECHNIQUES
Author : T. Paltashev, I. Perminov
Advanced Micro Devices, Sunnyvale, CA, United States of America
University ITMO, St. Petersburg, Russian Federation

Data : 2022-12-21
link : http://sv-journal.org/2014-1/06/en/index.php?lang=en

---

## Summary 

해당 reference는 기본적인 Texture Compression에 대한 Format들을 전반적인 이해도를 키우기 위해 대략적인 설명을 해주는 Reference이다. 본 논문을 읽으면 S3TC, ETC, PVRTC, ASTC 등 여러 Format들에 대한 이해도를 얻을 수 있다. 해당 정리 본은 본인의 연구에 초점이 맞추어져 있기 때문에 전반적인 Texture에 관한 이야기와 Standard한 Texture Compression Format을 중점적으로 정리했다. 더 자세한 내용은 link를 통해 알 수 있다. 

## Contents.

### Specifics of Texture Compression.
#### Texture Compression Scheme : 
- Random Access
- fixed-rate compression.
- lossy Compression. 
- High Decompression Speed:
	- Low cost hardware implementation of the decoder.
	- No indirect and chaining memory access.
- High Compression Ratio
- blocks of small size 
- Acceptable visual quailty

### Important Previous Works
- Palette compression is a special case of VQ (Vector Quantization) with the vector being a single pixel.
- Block compression, an alternative approach, employs compression of fixed size blocks.
- Block Truncating Coding(BTC)
	- 해당하는 기술은 mean과 variance를 이용하여 compression을 진행.
	- 각 texel에 대하여 mean값을 기준으로 두어 Quantization을 진행. (Figure 4 참고)
	- 4x4 의 Fixed Tile size를 사용.
- Color Cell Compression (CCC).
	- 위의 BTC는 gray scalar에 대한 texture를 target.
	- BTC와 동일한 방식으로 진행하지만 luminance 값을 이용하여 compression을 진행.
	- $Luma = 0.3 * R + 0.59 * G + 0.11 * B$
	- CCC results in a low quality image, since only two unique colors are used within a block.

### BC1 Block of S3TC Family
- S3TC는 Desktop을 target으로 하는 Texture Compression Techniques.
- 그 기술중 제일 Standard한 BC1 Block을 기준으로 정리.
- 뒤의 BC2~7은 BC1의 extend한 형태이기 때문에 자세한건 링크 참고.
- the BC1 block consists of two base colors c0 and c1 and an index table (bitmap)
- a two-bit entry.
- The presence of four different colors significantly increases the image quality in comparison to CCC.
- The base colors are stored in RGB565 format
- 기본적인 알고리즘은 CCC Algorithm과 매우 유사.
- For the first type, there are two ways to define c2 and c3.
- $$c_2 = \frac{2}{3}c_0 + \frac{1}{3}c_1$$
- $$c_3 = \frac{1}{3}c_0 + \frac{2}{3}c_1$$
- 더 좋은 quality를 위해서 아래와 같이 linear interpolation을 더 많은 step을 기준.
- $$c_2 = \frac{5}{8}c_0 + \frac{3}{8}c_1$$
- $$c_3 = \frac{3}{8}c_0 + \frac{5}{8}c_1$$

### ETC Family 
- The ETC (Ericsson Texture Compression) format was originally developed for use in mobile devices.
- The main idea of ETC compression is based on a well-known fact about color perception
	- 사람의 눈은 chrominance보다 luminance에 더 민감하기 때문에.

#### PACKMAN
- As mobile devices are strictly limited in memory sizes and bus widths, it was considered to use 2x4 tiles for PACKMAN.
- the compressed block occupies only 32 bits. Thus, the compression level is 4bpp, which is equal to that of BC1.
- Only one RGB444 base color is stored in a block. Other colors are obtained by changing texel’s luminance. While basic idea is far from BC1,
	- decoding 작업도 매우 비슷함. (Figure 21 참고.)
		- 해당하는 index table을 이용하여 Base color를 이용.
		- Luminance index에 대해 codebook을 참고하여 index table의 bit에 대한 값을 이용하여 해당하는 texel에 대해 연산을 진행. (Table 3 참고)
- The table values were created by starting with random values and then optimizing them by minimizing the error for a test image.

#### ETC1
- A single low precision (RGB444) base color was the main quality limiting factor in PACKMAN. This issue was addressed in ETC1.
- it consists two sub-blocks. 
- bit를 setting함으로써 2가지의 mode가 존재.
	- vertically
	- horizontally
- bit에 대한 구성은 총 64bit로 구성. (Figuare 22 참고.)
- The size of an ETC1 tile is 4x4, and it consists two sub-blocks, as in a PACKMAN block.
- Differential block type, where a base color for the first sub-block is stored with RGB555 precision, and 3-bit differences are stored for the second base color.
- A block, where both base colors are stored directly in RGB444 format, is called Individual.
- ETC1도 PACKMAN과 동일하게 Code book이 존재.

#### ETC2
- The differential mode and the possibility to rotate sub-blocks have significantly improved the quality.
- only one base color is available in each sub-block and tiles with high chrominance variation are compressed with high errors.
- smooth gradients can also be problematical for ETC1.
- ETC2 addresses these problems by introducing extra block modes.
	- when the sum of the base color and the offset overflows the valid 5-bit range [0, 31].
	- ETC2 format에 관해 3가지 모드가 존재하는 데 아래와 같은 mode가 존재.
	- T-mode : payload = 59-bits
	- H-mode : payload = 58-bits
	- P-mode : payload = 57-bits
	- 더 정확한 내용은 ETC2 paper를 읽어보며 다룰 예정.
- ETC2 bit들을 보면 Table을 표시하는 bitr가 존재하는데 Look-Up-Table(LUT)가 존재함.
	- 해당하는 bit를 이용하여 distance에 연산에서 사용.
- image quality를 올리기 위한 방식인 T/H-mode 
- image의 smooth gradient를 위한 P-mode
	- $$C(x, y) = \frac{x(C_H - C_0)}{4} + \frac{y(C_V - C_0)}{4} + C_0$$
- ETC2도 동일하게 codebook이 존재.