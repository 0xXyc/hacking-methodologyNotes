---
description: 05/14/2024
---

# WEP

## Introduction

WEP, or Wired Equivalent Privacy is an old wireless protection (introduced in 1997) and was the first attempt at securing wireless communications hence its shortcomings, broken cryptographic protocol, and inherent vulnerabilities.&#x20;

As a result, it became deprecated in 2004. But, it still makes up a little less than 10% of the entire wireless ecosystem, which is still quite a bit given the data it became deprecated.

### Why focus on an older security algorithm?

So you might be thinking, this is rather old, why focus on it? Well, it's still pretty relevant and sometimes seen in the wild still. Although not nearly as common as WPA and its close cousins, it's still out there.&#x20;

There are two main approaches when it comes to cracking WEP. A methodical manual approach or an automated brrrrr approach, using an automation tool known as [Wifite](https://github.com/derv82/wifite).&#x20;

### Manual Methodology

Like with all things hacking, it's best to learn all of the tricks of the trade as well as their ins and outs so that we can adapt in a multitude of scenarios and best use the Swiss army knife of tools at our disposal.&#x20;

#### Terminal Structure

1. <mark style="color:yellow;">`airmon-ng`</mark> -> 1st terminal tab
2. <mark style="color:yellow;">`airodump-ng`</mark> -> 2nd terminal tab
3. <mark style="color:yellow;">`aireplay-ng`</mark> -> 3rd terminal tab
4. <mark style="color:yellow;">`aircrack-ng`</mark> -> 4th terminal tab

