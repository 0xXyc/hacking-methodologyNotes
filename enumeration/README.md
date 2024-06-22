---
cover: ../.gitbook/assets/dbdhoju-2953e0d9-f588-4611-8856-a1669bd066b1.gif
coverY: 0
---

# 🔎 Logical Port Enumeration

> ## <mark style="color:green;">"Always keep a form of recon going in the background" \~Ippsec</mark>

## Introduction

On every single computer and device exists a technology known as "ports". Both coming in form factors of physical and logical ports, they have very different missions.&#x20;

### Physical Ports

A physical port is a physical, tangible, port that humans can see (e.g.) USB 3.0 port. They allow for a physical extension to the I/O implementation of the computer; meaning, keyboard, mouse, web camera, etc. Essentially, allowing the user to be able to add additional extensibility to their device in any way they seem fit. The only limit of physical ports on a computer are the amount of space available on the PCB/motherboard or on the adapter or dock.

### Logical Ports

Logical ports are very different from physical ports. We cannot see them. They act as a gateway or a door for the computer to be able to communicate with other applications or devices over a network.&#x20;

There is a limit of 65,535 possible logical ports that a computer can operate with and support. Logical ports are also directly associated with firewalls and their corresponding rules. When you are blocking a particular application or port within your "advanced firewall rules", these logical ports are essentially what you are telling the firewall to either allow or deny inbound/outbound connections from.&#x20;

As a result, you can enumerate a system remotely and find out what types of applications and other technologies are running on the system; greatly growing the overall attack surface. Increasing the likelihood of a successful breach.&#x20;

{% hint style="info" %}
**🚨 An application can run on any port it wants. These are simply just standards. Please be aware of this. This means attackers or defenders can utilize applications such as SSH or FTP on any port that they want as long as the client/server is aware of these changes, it will work just fine! Often times, this is done for security through obscurity.**
{% endhint %}

There are a few different types of logical ports:

1. **Well known ports: 0 - 1023**
2. **Registered ports: 1024 - 49151**
3. **Ephemeral ports: 49151 - 65535**

<mark style="color:yellow;">Well known ports</mark> are reserved for common port numbers that correspond to a series of computer protocols. This includes (but is not limited to):

* HTTP (80)
* HTTPs (443)
* DNS (53)
* SMB (445)
* SSH (22)
* Telnet (23)
* FTP (21)
* Many more&#x20;

<mark style="color:yellow;">Registered ports</mark> are often reserved for popular applications, platforms, etc. for logical ports. For example, with the game Doom, runs on port 666.

Ephemeral ports are "quick use" ports that are often times opened just as quickly as they are closed. This allows for quick outbound ports to be opened up to carry out important information. This behavior can be observed with Wireshark or other packet capture utilities. You can see when using HTTPs or SSH, each time you initiate a new connection or carry out different data, a new port usually within the 50,000's will be quickly opened up, followed by the inbound standard port to the corresponding application to complete the connection.

**For example:** SSH Connection DST\_IP -> SRC\_IP -> 54042:22 (Ephemeral port, SSH port)

## Questions to ask

At the time of enumeration, assemble all of the pieces within your notes

* Cycle through ports, accounts, and accesses
* Is authentication required?
* Which kind of authentication?
* Is it basic user/password?
* What do I need? One? Both?
* What is the service?
* What is the version?

{% content-ref url="21-ftp-enumeration.md" %}
[21-ftp-enumeration.md](21-ftp-enumeration.md)
{% endcontent-ref %}

{% content-ref url="22-ssh/" %}
[22-ssh](22-ssh/)
{% endcontent-ref %}

{% content-ref url="23-telnet.md" %}
[23-telnet.md](23-telnet.md)
{% endcontent-ref %}

{% content-ref url="25-smtp.md" %}
[25-smtp.md](25-smtp.md)
{% endcontent-ref %}

{% content-ref url="53-dns.md" %}
[53-dns.md](53-dns.md)
{% endcontent-ref %}

{% content-ref url="80-443-http-s/" %}
[80-443-http-s](80-443-http-s/)
{% endcontent-ref %}

{% content-ref url="88-kerberos.md" %}
[88-kerberos.md](88-kerberos.md)
{% endcontent-ref %}

{% content-ref url="110-995-pop.md" %}
[110-995-pop.md](110-995-pop.md)
{% endcontent-ref %}

{% content-ref url="135-rpc.md" %}
[135-rpc.md](135-rpc.md)
{% endcontent-ref %}

{% content-ref url="137-138-139-net-bios.md" %}
[137-138-139-net-bios.md](137-138-139-net-bios.md)
{% endcontent-ref %}

{% content-ref url="139-445-smb.md" %}
[139-445-smb.md](139-445-smb.md)
{% endcontent-ref %}

{% content-ref url="161-162-10161-10162-snmp.md" %}
[161-162-10161-10162-snmp.md](161-162-10161-10162-snmp.md)
{% endcontent-ref %}

{% content-ref url="389-636-3268-3269-ldap-s.md" %}
[389-636-3268-3269-ldap-s.md](389-636-3268-3269-ldap-s.md)
{% endcontent-ref %}

{% content-ref url="2049-nfs.md" %}
[2049-nfs.md](2049-nfs.md)
{% endcontent-ref %}

{% content-ref url="3306-mysql.md" %}
[3306-mysql.md](3306-mysql.md)
{% endcontent-ref %}

{% content-ref url="5985-5986-winrm.md" %}
[5985-5986-winrm.md](5985-5986-winrm.md)
{% endcontent-ref %}
