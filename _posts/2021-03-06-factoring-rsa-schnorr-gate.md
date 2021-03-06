---
title: "RSA암호의 종말? 과연 그 진위는? a.k.a Schnorr-gate"
categories: 
   - Cryptography
---

며칠 전(3월 3일) [eprint][1]<sup>[1](#footnote_1)</sup>에 [RSA 암호가 깨졌다는 내용의 논문][schnorr_paper]이 공개되었다.

이 논문의 저자는 [Schnorr 전자서명][3]로 잘 알려진 암호학자 Schnorr였고,
초록에는 _This destroys the RSA cryptosystem (이 공격은 RSA암호시스템을 파괴할 것이다)_ 라고 당당히 밝히고 있었기에
논문이 업로드된지 하루도 되지 않아 [암호커뮤니티 (crypto stackexchange)를 비롯하여][4] [reddit][5]과 같은 곳에서는 엄청난 파장을 불러일으키며 논문의 진위에 대한 논란이 확장되었다.


## RSA암호와 소인수분해 문제
참고로 RSA암호는 현대 인터넷 통신의  보안을 위하여  필수적인 기술로써,
RSA암호의 안전성은 현대의 컴퓨터로는 사이즈가 큰 수 (현재 안전다하고 여겨지는 정도는 1024bit 이상의 수이다.)의 소인수분해 문제를 푸는 것이 어렵다는 사실 (기술적인 용어로는 *계산복잡도 (computational complexity)*라고 한다)에 기반하고 있다. 
현재까지 알려진 가장 빠른 소인수분해 알고리즘은 [Number Field Sieve][6] 알고리즘이며,
최신 기록은 2020년 2월에 발표된 것으로 [RSA-250 challenge (250자리 십진수의 소인수분해)를 Intel Xeon Gold 6130 CPU로 2700 core-years 시간만에 계산][7]할 수 있는 것으로 알려져 있다.

불과 1년 전 학계 최고 수준의 기록으로도 250자리수 소인수분해를 계산하는데 무려 2700년 정도가 걸리는 것을 생각하면,
RSA암호를 파괴할 수 있다는 Schnorr의 주장이 학계 및 산업계에 얼마나 큰 파장을 불러일으킬까 하는 것을 추측하기는 어렵지 않을 것이다.

## RSA암호의 종말? 그 진위는?
기존의 소인수분해 알고리즘 [Quadratic Sieve][8]나 [Number Field Sieve][6]등은 크게 Sieving 단계와 Linear algebra단계로 나누어진다.
일반적으로 Number Field Sieve 알고리즘 등에서 가장 시간을 소모하는 부분이 바로 Sieving 단계이며,
2020년의 기록 또한 총 2700년의 계산 시간 중 2450년 정도의 시간이 Sieving 단계에 소요되었다.

Schnorr의 알고리즘은 주장대로라면 이 Sieving 단계의 계산을 매우 빠른 시간 안에 계산할 수 있게 된다.
(Schnorr의 논문에는 계산복잡도가 명시적으로 드러나 있지 않기 때문에 수치적으로 기존 알고리즘 대비 얼마나 빨라진다고 주장하고 있는지는 분명하지 않다.)
하지만, 이미 cryto stackexchange 등에서 드러난 바와 같이 Schnorr의 논문은 몇가지 치명적인 계산 오류를 포함하고 있다.
(아래 내용은 [crypto exchange][4]의 Daniel Shiu 답변 및 [Keegan Ryan의 twitter][9]을 참조)

* factor relation이라고 불리는 관계식을 찾기 위해서는 lattice
<img src="https://latex.codecogs.com/gif.latex?R_{n,f}" title="R_{n,f}" />
의 short vector를 찾아야 한다.
그리고 이 short vector의 크기는 lattice의 determinant 값에 크게 의존한다.
하지만 논문에서는 이 determinant 계산에 오류가 있다.
(lattice
<img src="https://latex.codecogs.com/gif.latex?R_{n,f}" title="R_{n,f}" />
의 determinant를
<img src="https://latex.codecogs.com/gif.latex?N^{n&plus;1}\frac{n(n&plus;1)}{2}lnN" title="N^{n+1}\frac{n(n+1)}{2}lnN" />
로 계산하고 있지만, 실제로는
<img src="https://latex.codecogs.com/gif.latex?N^{n&plus;1}n!lnN" title="N^{n+1}n!lnN" />
이어야 한다.)

* determinant 값의 큰 오류로 인해서 factor relation을 얻기 위한 계산량에 엄청난 차이가 발생하게 된다.
논문의 3쪽에서
<img src="https://latex.codecogs.com/gif.latex?N&space;\approx&space;2^{800}" title="N \approx 2^{800}" />
경우,
잘못된 determninat값으로 인하여 short vector의 크기를 0.917 정도로 estimate하였으나,
실제로는 10^243 정도의 크기를 갖는다.
결과적으로 논문에서는 하나의 factor relation을 얻을 확률을 거의 1로 예측하였지만
실제로 이 확률은 10^(-202) 정도로 매우 작아지며,
따라서 96개의 factor relation을 얻기 위해서는 96번의 lattice reduction이 아니고 거의 2^200번의 reduction이 필요하게 된다.


lattice 암호 분야의 전문가인 Leo Ducas는 Schnorr의 알고리즘을 실제로 구현하여 [github에 공개][10]하였으며,
Schnorr의 알고리즘으로는 주장하고 있는 만큼 충분한 factor relation을 얻을 수 없다는 것을 보여주었다.

## 향후 향방은?
결과적으로 Schnorr의 현재 알고리즘은 몇가지 계산적 오류를 포함하고 있으며,
이는 RSA를 파괴하기에 충분하지 못할 것으로 보인다.
조만간 Schnorr 본인이 이같은 오류에 대하여 수정하거나 별도의 해명이 있을 수도 있지만,
만약 그렇지 않다면 이 사건은 현재로서는 해프닝으로 끝날 것으로 보인다.

다만 Schnorr의 아이디어는 연구자들의 흥미를 자극하기에 충분했으며,
이번 사건을 계기로 소인수분해 알고리즘을 비롯한 cryptanalysis (암호분석)분야의 연구에 새로운 동기부여가 될 수 있을 것으로 기대한다.

_written by Taechan Kim_


<a name="footnote_1"><font size="2">1</font></a>: <font size = "2">암호와 관련된 preprint를 공유하는 논문 사이트로 타 분야의 [arxiv][2]와 비슷한 사이트임.</font>



[1]: https://eprint.iacr.org/
[2]: https://arxiv.org/
[schnorr_paper]: https://eprint.iacr.org/2021/232.pdf
[3]: https://en.wikipedia.org/wiki/Schnorr_signature
[4]: https://crypto.stackexchange.com/questions/88582/does-schnorrs-2021-factoring-method-show-that-the-rsa-cryptosystem-is-not-secur
[5]: https://www.reddit.com/r/math/comments/lwf0t5/fast_factoring_integers_by_svp_algorithms/
[6]: https://en.wikipedia.org/wiki/General_number_field_sieve
[7]: https://listserv.nodak.edu/cgi-bin/wa.exe?A2=NMBRTHRY;dc42ccd1.2002
[8]: https://en.wikipedia.org/wiki/Quadratic_sieve#cite_note-1
[9]: https://twitter.com/inf_0_/status/1367376959055962112
[10]: https://github.com/lducas/SchnorrGate
