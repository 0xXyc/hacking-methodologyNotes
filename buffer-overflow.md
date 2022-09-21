---
description: >-
  A Buffer Overflow (BOF) occurs when the amount of data in the buffer exceeds
  its storage capacity. Once this happens, the extra data overflows into other
  memory location and RCE can occur.
cover: >-
  https://www.imperva.com/learn/wp-content/uploads/sites/13/2018/01/buffer-overflow.png
coverY: 0
---

# 🚰 Buffer Overflow

## Visual Depiction

<figure><img src=".gitbook/assets/Capture (4).PNG" alt=""><figcaption></figcaption></figure>

The stack is overflowed and is replaced with malicious shellcode.

## MY Walkthrough

{% content-ref url="buffer-overflow/overflow1.md" %}
[overflow1.md](buffer-overflow/overflow1.md)
{% endcontent-ref %}

{% content-ref url="buffer-overflow/overflow2.md" %}
[overflow2.md](buffer-overflow/overflow2.md)
{% endcontent-ref %}

{% content-ref url="buffer-overflow/overflow3.md" %}
[overflow3.md](buffer-overflow/overflow3.md)
{% endcontent-ref %}

## Attacking the Stack part of memory

* We need to overflow buffer space → write over buffer space down to EIP → Control the stack → spawn reverse shell
* Remember, the buffer overflow methodology consists of several steps:
  1. Spiking
     1. Method to find the vulnerable part of the program.
  2. Fuzzing
     1. Sending a bunch of characters/messages to a program to break it.
  3. Finding the offset
     1. At what point did we break it? This is called the offset.
  4. Overwriting the EIP
     1. The offset will then be utilized to overwrite the EIP.
  5. Finding bad characters
     1. We need to find out which characters are compatible with the shellcode and which characters are not.
  6. Finding the correct module
     1. Looking for a DLL or something similar within a program that does not contain memory protections.
     2. Mona Modules + Immunity Debugger
  7. Generate shellcode that leads to RCE

<figure><img src=".gitbook/assets/Capture (5).PNG" alt=""><figcaption></figcaption></figure>

## Buffer Overflow Methodology Example

#### Step-By-Step Example

### Spiking:

[https://www.youtube.com/watch?v=3x2KT4cRP9o\&list=PLLKT\_\_MCUeix3O0DPbmuaRuR\_4Hxo4m3G\&index=2](https://www.youtube.com/watch?v=3x2KT4cRP9o\&list=PLLKT\_\_MCUeix3O0DPbmuaRuR\_4Hxo4m3G\&index=2)

### Fuzzing:

[https://www.youtube.com/watch?v=FCIfWTAtPr0\&list=PLLKT\_\_MCUeix3O0DPbmuaRuR\_4Hxo4m3G\&index=3](https://www.youtube.com/watch?v=FCIfWTAtPr0\&list=PLLKT\_\_MCUeix3O0DPbmuaRuR\_4Hxo4m3G\&index=3)

### Finding the Offset:

[https://www.youtube.com/watch?v=GqwyonqLYdQ\&list=PLLKT\_\_MCUeix3O0DPbmuaRuR\_4Hxo4m3G\&index=4](https://www.youtube.com/watch?v=GqwyonqLYdQ\&list=PLLKT\_\_MCUeix3O0DPbmuaRuR\_4Hxo4m3G\&index=4)

### Overwriting the EIP:

[https://www.youtube.com/watch?v=Wh9wRKBzajo\&list=PLLKT\_\_MCUeix3O0DPbmuaRuR\_4Hxo4m3G\&index=5](https://www.youtube.com/watch?v=Wh9wRKBzajo\&list=PLLKT\_\_MCUeix3O0DPbmuaRuR\_4Hxo4m3G\&index=5)

### Finding Bad Characters:

[https://www.youtube.com/watch?v=uIFYNVqpZ0k\&list=PLLKT\_\_MCUeix3O0DPbmuaRuR\_4Hxo4m3G\&index=6](https://www.youtube.com/watch?v=uIFYNVqpZ0k\&list=PLLKT\_\_MCUeix3O0DPbmuaRuR\_4Hxo4m3G\&index=6)

### Finding the Right Module:

[https://www.youtube.com/watch?v=k9D9RuFT02I\&list=PLLKT\_\_MCUeix3O0DPbmuaRuR\_4Hxo4m3G\&index=7](https://www.youtube.com/watch?v=k9D9RuFT02I\&list=PLLKT\_\_MCUeix3O0DPbmuaRuR\_4Hxo4m3G\&index=7)

### Generating Shellcode and Gaining RCE:

[https://www.youtube.com/watch?v=qSjxR8tfokg\&list=PLLKT\_\_MCUeix3O0DPbmuaRuR\_4Hxo4m3G\&index=8](https://www.youtube.com/watch?v=qSjxR8tfokg\&list=PLLKT\_\_MCUeix3O0DPbmuaRuR\_4Hxo4m3G\&index=8)

### Additional Resources for Buffer Overflows:

[TryHackMe | Buffer Overflow Prep](https://tryhackme.com/room/bufferoverflowprep)

[TryHackMe | Brainpan 1](https://tryhackme.com/room/brainpan)
