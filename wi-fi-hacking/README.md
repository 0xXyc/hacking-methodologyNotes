---
description: 802.11 Security
---

# 📻 Wi-Fi Hacking

## Introduction to the 802.11 Standard

* This is the current standard for wireless technology

It can be composed into different sections, or bands:

* _802.11_: The original WLAN standard
* _802.11a_: Up to 54 Mbit/s on 5 GHz
* _802.11b_: 5.5 Mbit/s and 11 Mbit/s on 2.4 GHz
* _802.11g_: Up to 54 Mbit/s on 2.4 GHz, backwards compatible with 802.11b
* _802.11h_: Regulatory requirement to limit power and transmission in the 5GHz band
* _802.11i_: Provides enhanced security
* _802.11n_: Provides higher throughput with Multiple Input/Multiple Output (MIMO), aka Wi-Fi 4
* _802.11ac_: Very High Throughput < 6 GHz, aka Wi-Fi 5
* _802.11ad_: Multi-Gigabit in the 60GHz band, known as WiGig
* _802.11ax_: High Efficiency (HE) Wireless LAN, aka Wi-Fi 6

**The main way to remember this or for quick reference would be:**

B, A, G, N, AC, AX.

## Wireless Encryption

* Since Wi-Fi transmits data over radio waves, it is subject to eavesdropping
* That means encryption needs to be in use to protect the data while in flight
* <mark style="color:yellow;">Wired Equivalent Privacy (WEP)</mark> was created when 802.11 was released in the 90's
  * WEP can be cracked in under a minute!
* <mark style="color:yellow;">Wi-Fi Protected Access (WPA)</mark> came out in 2003 to supersed WEP
* <mark style="color:yellow;">WPS (Wi-Fi Protected Setup)</mark> was released in 2006 to standardize PIN-based authentication
  * 8-digit PIN codes for authentication
* <mark style="color:yellow;">WPA3</mark> announced in 2018

### Wireless Operating Modes

<mark style="color:yellow;">Managed</mark>: Client/operational mode -> default mode/non-promiscuous

<mark style="color:yellow;">Monitor</mark>: Privileged operational mode, promiscuous mode -> Ability to see what's going on around you

To find out what mode your wireless card is currently in, you can run `iwconfig`.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption><p>Viewing operating modes via <code>iwconfig</code></p></figcaption></figure>

### [Aircrack-ng](http://aircrack-ng.org/) Suite of Wireless Penetration Testing Tools

We will be spending quite a bit of time learning the <mark style="color:green;">aircrack-ng</mark> suite. This is an absolutely OG toolset and will be imperative for learning going forward.&#x20;

#### Tools to be familiar with in the suite

1. <mark style="color:yellow;">`airmon-ng`</mark> -> View network interfaces and their current operating modes as well as place them/assign them a separate monitor mode interface (e.g. wlan34mon)
2. <mark style="color:yellow;">`airodump-ng`</mark> -> Begin capturing wireless packets and data on a specified wireless network interface that's been placed in monitor mode
3. <mark style="color:yellow;">`aireplay-ng`</mark> -> Grants the ability to perform deauthentication attacks as well as replaying intercepted information to fool security systems
4. <mark style="color:yellow;">`aircrack-ng`</mark> -> This is a tool that will be ran to actively attack (crack) the harvested/intercepted wireless data (e.g. commonly used in conjunction with a lengthy series of IV's or with a handshake (obtained via newly joining client or deauthed client and interception of handshake after deauth packet is sent) and a dictionary wordlist.

## WiFi Hacking Blog Structure

Going forward, I'm going to break up each section of 802.11 security algorithms accordingly. There, I will address their vulnerabilities and attack paths.

### Security algorithms to be mindful of

* WEP64 & WEP128 -> Inherently vulnerable and broken in nature
  * &#x20;Enough Initialization Vectors (IV's) will lead to a successful crack
* WPA
* WPA2
* WPA3
  * All three forms of WPA are rather secure but one solid sample (A.K.A. captured or deauthenticated handshake) alongside with a solid dictionary wordlist will lead to a successful crack
  * Upon failure, other options will have to be weighed out

## Wireless Pentesting Overview

{% embed url="https://academy.tcm-sec.com/courses/1152300/lectures/24793689" %}

### WPA Exploit Walkthrough

{% embed url="https://academy.tcm-sec.com/courses/1152300/lectures/24793690" %}
