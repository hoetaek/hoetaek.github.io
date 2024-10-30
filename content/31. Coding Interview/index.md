---
title: Coding Interview
---
# 시간 복잡도 계산하는 법
Practical Time Complexity Limits

1.	Constant Time - O(1):
	- No practical limit. This complexity is generally manageable regardless of input size.
2.	Logarithmic Time - O(\log n):
	- Very efficient. Can handle very large inputs
	- 최대 $10^{18}$(1,000,000,000,000,000,000) 그리고 더 크게도 가능함
3.	Linear Time - O(n):
	- Efficient for large inputs. 
	- 최대 10<sup>6</sup> (1,000,000)
4.	Linearithmic Time - O(n \log n):
	- Still efficient for large inputs
	- 최대 10<sup>5</sup> to 10<sup>6</sup>(1,000,000)
5.	Quadratic Time - O(n^2):
	- Practical for moderate-sized inputs
	- 최대 $10^4$ (10,000)
6.	Cubic Time - O(n^3):
	- Only practical for small inputs. Typically,  n  should be less than 500 to 1,000.
7.	Exponential Time - O(2^n):
	- Impractical for large  n . 
	- 최대 20 ~ 30
8.	Factorial Time - O(n!):
	- Extremely impractical for large  n . 
	- 최대 10 ~ 15

