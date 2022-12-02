---
description: Root shells all day
---

# 😆 Hardware Hacking

## Excellent References

### Flashback Team

{% embed url="https://www.youtube.com/@FlashbackTeam" %}

### Stacksmashing

{% embed url="https://www.youtube.com/@stacksmashing" %}

## Common Acronyms

### UART- <mark style="color:yellow;">Universal Asynchronous Receiver/Transmitter</mark>

{% embed url="https://www.youtube.com/watch?v=01mw0oTHwxg" %}

* A computer hardware device that provides <mark style="color:yellow;">serial communication for debugging</mark>
* Transmission speeds and data formats are configurable
* This device looks different on all devices, can be hidden, removed, or modified&#x20;
* Four pins, however <mark style="color:yellow;">we ONLY need to worry about three</mark> of them
  * <mark style="color:yellow;">RX</mark>, <mark style="color:yellow;">TX</mark>, and <mark style="color:yellow;">GND</mark> (VCC is not important to us)
  * <mark style="color:red;">NOTE</mark>: They are often configured in different orders from board to board, so troubleshooting with a multimeter is required prior to testing!

<figure><img src=".gitbook/assets/image (20) (3).png" alt=""><figcaption><p>Pins exposed (no need to solder) </p></figcaption></figure>

<figure><img src=".gitbook/assets/image (12) (4).png" alt=""><figcaption><p>Exposed but Soldering or "Paper Clip Bridge" method required here</p></figcaption></figure>

<figure><img src=".gitbook/assets/image (17) (4).png" alt=""><figcaption><p>Heavily modified/hidden UART</p></figcaption></figure>

<figure><img src=".gitbook/assets/image (8).png" alt=""><figcaption><p>"Paper Clip Bridge"</p></figcaption></figure>

## GPIO- <mark style="color:yellow;">General Purpose Input/Output</mark>

A standard interface used to <mark style="color:yellow;">connect microcontrollers to other devices</mark>

* Classified as an <mark style="color:yellow;">uncommitted digital signal pin</mark>
* Found on integrated circuits and electronic circuit boards
* Used as an input, output, or both
* No explicit predefined purpose and are left on the board <mark style="color:yellow;">unused by default</mark>

<figure><img src=".gitbook/assets/image (8) (8).png" alt=""><figcaption><p>GPIO embedded on a Raspberry Pi</p></figcaption></figure>

## JTAG- Joint Test Action Group
