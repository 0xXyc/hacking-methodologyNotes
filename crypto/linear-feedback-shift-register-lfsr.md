---
description: 09/11/2023
---

# Linear Feedback Shift Register (LFSR)

## Introduction

A shift register <mark style="color:yellow;">where the input bit is the output of a linear function of two or more of its previous states (taps)</mark>.

The <mark style="color:yellow;">initial input of an LSFR is called a fill or a seed</mark>.&#x20;

This happens by shifting the bits one position to the left and <mark style="color:yellow;">replacing the leaving bit with XOR of the the leaving bit and bits at special locations in the register</mark> called <mark style="color:yellow;">taps</mark>.

This constitutes one step of the feedback shift register.

{% embed url="https://www.sciencedirect.com/topics/mathematics/linear-feedback-shift-register" %}
Insane explanation of LFSR
{% endembed %}

{% embed url="https://www.youtube.com/watch?v=Ks1pw1X22y4" %}
Computerphile Explanation
{% endembed %}

## Random Numbers

```python
state = 0b1001
for i in range(20):
    print("{:04b}".format(state))
    newbit = (state ^ (state >>1)) & 1
    state = (state >>1) | (newbit <<3 )
```

Output:

```
1001
1100
0110
1011
0101
1010
1101
1110
1111
0111
0011
0001
1000
0100
0010
1001
1100
0110
1011
0101
```

Creating a super number:

```python
state = (1 << 127) | 1
while True:
    print(state & 1, end = '')
    newbit = (state ^ (state >> 1) ^ (state >> 2) ^ (state >> 7))
    state = (state >> 1) | (newbit << 3)
```

Output:

```
10001011111111111110111111111111111011111111111111101111111111111110111111111101111111111111111111111111111111111111111111111111111111111111110111111111111111011111111111111101111111111111110111111111111111011111111111111101111111111111110111111111111111011111111111111101111111111111110111111111111111011111111111111101111111111111110111111111111111011111111111111101111111111111110111111111111111011111111111111101111111111111110111111111111111011111111111111101111111111111110111111111111111011111111111111101111111111111110111111111111111011111111111111101111111111111110111111111111111011111111111111101111111111111110111111111111111011111111111111101111111111111110111111111111111011111111111111101111111111111110111111111
```
