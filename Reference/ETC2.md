# ETC2: Texture Compression using Invalid Combinations
---
Title : ETC2: Texture Compression using Invalid Combinations
Author : Jacob Ström and Martin Pettersson
Ericsson Research

Date : 2022-12-23
link : https://dl.acm.org/doi/10.5555/1280094.1280102

---

## Summary 
해당 논문은 기존 ETC1에서 3개의 mode를 추가하여 개선 시킨 연구이다. 기존 bit의 구조에서 T/H-mode 그리고 P-mode를 추가하여 image quality를 개선시키는 방향으로 진행이 된다. Decoding 방식도 기존과 동일하게 단순하게 진행이 되고 연구의 주된 target이 ETC2이기 때문에 개선 방향을 잡는 좋은 논문이기도 하여 읽어보기로 했다. 추가적으로 이러한 ETC2 format의 compression time을 100배 감소시킨 방식을 연구한 QuickETC2를 연속적으로 읽어보며 연구의 방향을 잡아나갈 것이다. 


## Contents
### Introduction.
- Texture compression can be used to re- duce bandwidth usage.
- Our new texture compression system is built on the iPACKMAN/ETC system.
- Our major contribution is to show how three extra modes can be added by the use of invalid bit combi- nations, while keeping backwards compatibility and with- out increasing the bit rate.
- 기존 ETC1과 여러 방식에 대한 설명과 ETC1을 개선한 방안으로 설명한다는 요약.

### Invalid Bit Sequence ==(논문의 핵심)==
- One major contribution of this paper is to show how invalid bit combinations can be used to improve ETC.
- They can thus signal that the bits of the blocks should be interpreted in a new way, and not be decoded using the regular ETC logic. $\to$ bit 에 대한 overflowr가 발생하게 된다면 ETC1의 logic은 물리적으로 의미가 없어진다. 
- If the diffbit is not set, the 63 remaining bits are decoded using the regular individual mode from ETC.
- we first see whether the addition R1 = R0+dR underflows or overflows.
- If we are not overflowing, we are decoding the 63 bits the usualway, using the differential mode from ETC
- if the sum overflows, we decode the bits in an alternate way.
- We say that we have a payload of 55 bits in the auxiliary mode.
- Overflow가 발생하게 된다면 다른 방식으로 Compression을 진행. 여기서 overflow & underflow 방식에 따라 payload의 bit size가 달라짐.
- 만약 Decoding을 진행하게 될 때, R0 = 2, dR = -3이라면 underflow가 존재하게 된다. ETC2도 동일하게 overflow관련 table이 존재하는데 (Table 1 참고) , Table을 참고하여 4개의 bit만 추출한다. 
- In this way, we have increased the payload in our auxiliary mode from 55 bits to 59.
- overflow가 발생할 때 발생한 색상에 따라서 mode가 실행되어짐.
	- 해당하는 Red Data에서 Overflow가 발생하지 않고 Green에서 Overflow가 발생한다고 가정한다면 Red Color에서 5bit, diff bit 1 으로 설정하여 58bit의 payload를 가지게 된다.
- Analogously, it is possible to construct a third mode, where red and green does not overflow, but blue does, and this mode will have a payload of 57 bits.

## New Modes in ETC2
- T-mode 
	- LUT(Look Up Table)을 이용하여 해당하는 Distance들을 이용하여 가장 최적의 색상을 표현하는 distance에 대한 정보를 저장.
	- 3개의 paint color를 이용.
	- 해당하는 그래프를 그리게 된다면 "T"와 비슷하게 생겨 T-mode라는 이름이 붙음.
	- The first 24bits are the two base colors coded as RGB444. These colors are extended to RGB888 by copying the lower four bits to the upper four bits for each component.
	- 생성된 4개의 Paint Color를 이용하여 가장 색상을 잘 나타내는 색상을 찾음.

- H-mode 
	- Sometimes there are two groups of colors for which in- tensity modulation could be useful.
	- since the H-pattern is completely symmetri- cal, we can swap the base colors and obtain exactly the same result.

- P-mode 
	- it is desirable to find a representation that can cope well with smoothly varying chrominances, like the ones found in the second image
	- linear filter를 사용하여 근처에 있는 색상을 계산.
	- $$R(x, y) = \frac{x(R_H - R_0)}{4} + \frac{y(R_V - R_0)}{4} + R_0$$
		- $\to$ Hardware의 더 잘 계산할 수 있는 layout을 위함.
		- division by 4 instead of 3 gives the added advantage that smaller steps are possible, and smoother transitions can thus be represented.
	- A disadvantage is that it is not always possible to have maximum or minimum strength in some pixels in a block,
	- this mode does not have to cope with every type of block, since there are four other modes that will most likely be able to handle those well.

- _In total, ETC2 consists of the following five modes: Indi- vidual and differential, that were also part of ETC, the 59-bit T-mode and the 58-bit H-mode, which are modifications of previous work, and the 57-bit planar mode, which is new._