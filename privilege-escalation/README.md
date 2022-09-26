---
description: >-
  This is a post compromise goal where a threat actor is attempting to achieve
  root or administrator level access.
---

# 🤓 Privilege Escalation

## Goal: low priv -> root/administrator

This can be simple as a kernel exploit or require lots of recon on the compromised system.

Privilege escalation is essentially an example of access control violations.

## Linux Privilege Escalation

{% content-ref url="linux/" %}
[linux](linux/)
{% endcontent-ref %}

### Linux Privilege Escalation Guides

The only guide I probably ever used to help me understand privilege escalation techniques in Linux systems was from g0tmi1k post.&#x20;

This blog is a must that everyone should have for preparing for the OSCP in my opinion. You can find his guide here: [https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)

GTFOBins (I have to thank Ippsec for sharing this with me): Contains a curated list of Unix binaries that that have the ability to be exploited by an attacker to bypass local security restrictions on a Linux system. [https://gtfobins.github.io/](https://gtfobins.github.io/)

PayloadsAllTheThings Linux Priv Esc Guide: [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md)

**Linux Privilege Escalation Tools:**

LinEnum: A great Linux privilege escalation checker that is still maintained by the guys at rebootuser.com. You can find there tool here: [https://github.com/rebootuser/LinEnum](https://github.com/rebootuser/LinEnum)

* Linux Exploit Suggester 2: [https://github.com/jondonas/linux-exploit-suggester-2](https://github.com/jondonas/linux-exploit-suggester-2)
* LinPEAS: \[https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS]

## Windows Privilege Escalation

{% content-ref url="windows/" %}
[windows](windows/)
{% endcontent-ref %}

### **Windows Privilege Escalation Guides**

* Fuzzysecurity Windows Privilege Escalation Fundamentals: Shout out to fuzzysec for taking the time to write this because this is an amazing guide that will help you understand Privilege escalation techniques in Windows. [http://www.fuzzysecurity.com/tutorials/16.html](http://www.fuzzysecurity.com/tutorials/16.html)
* Pwnwiki Windows Privilege Escalation Commands: [http://pwnwiki.io/#!privesc/windows/index.md](http://pwnwiki.io/#!privesc/windows/index.md)
* Absolomb’s Security Blog: Windows Privilege Escalation Guide [https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/](https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/)
* Pentest.blog: Windows Privilege Escalation Methods for Pentesters [https://pentest.blog/windows-privilege-escalation-methods-for-pentesters/](https://pentest.blog/windows-privilege-escalation-methods-for-pentesters/)
* PayloadAllTheThings: [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)
* SharpAllTheThings: [https://github.com/N7WEra/SharpAllTheThings](https://github.com/N7WEra/SharpAllTheThings)
* LOLBAS (Created by Oddvar Moe): [https://lolbas-project.github.io/](https://lolbas-project.github.io/)

Windows Privilege Escalation Tools:

* JAWS (Created by 411Hall): A cool windows enumeration script written in PowerShell. [https://github.com/411Hall/JAWS/commits?author=411Hall](https://github.com/411Hall/JAWS/commits?author=411Hall)
* Windows Exploit Suggester Next Generation: [https://github.com/bitsadmin/wesng](https://github.com/bitsadmin/wesng)
* Sherlock (Created by RastaMouse): Another cool PowerShell script that finds missing software patches for local privilege escalation techniques in Windows. [https://github.com/rasta-mouse/Sherlock](https://github.com/rasta-mouse/Sherlock)
* WinPeas: [https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)
* Watson: [https://github.com/rasta-mouse/Watson](https://github.com/rasta-mouse/Watson)
* Seatbelt: [https://github.com/GhostPack/Seatbelt](https://github.com/GhostPack/Seatbelt)
* Powerless: [https://github.com/M4ximuss/Powerless](https://github.com/M4ximuss/Powerless)
* Powerview: [https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon](https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon)















