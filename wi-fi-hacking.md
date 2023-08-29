---
description: 802.11 Security
---

# 📻 WiFu

## Introduction to the 802.11 Standard

* This is the current standard for wireless technology

It can be composed into different sections:

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

## Wireless Encryption

* Since Wi-Fi transmits data over radio waves, it is subject to eavesdropping
* That means encryption needs to be in use to protect the data while in flight
* <mark style="color:yellow;">Wired Equivalent Privacy (WEP)</mark> was created when 802.11 was released in the 90's
  * WEP can be cracked in under a minute!
* <mark style="color:yellow;">Wi-Fi Protected Access (WPA)</mark> came out in 2003 to supersed WEP
* <mark style="color:yellow;">WPS (Wi-Fi Protected Setup)</mark> was released in 2006 to standardize PIN-based authentication
  * 8-digit PIN codes for authentication
* <mark style="color:yellow;">WPA3</mark> announced in 2018

## Wireless Pentesting Overview

{% embed url="https://academy.tcm-sec.com/courses/1152300/lectures/24793689" %}

### WPA Exploit Walkthrough

{% embed url="https://academy.tcm-sec.com/courses/1152300/lectures/24793690" %}
