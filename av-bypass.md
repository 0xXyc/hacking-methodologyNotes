---
description: 09-26-22
---

# 😇 AV Bypass

There are two main methods that attackers will use in an effort to mitigate anti virus detection.

Those are **Obfuscation** and **Encryption.**

* <mark style="color:yellow;">Obfuscation</mark> distorts the malware while keeping its form
* <mark style="color:yellow;">Encryption</mark> eliminates the ability for AV to detect malware through signature alone
  * This can be done using a "crypter"
  * There are two types of crypters: **scantime** and **runtime.**

<mark style="color:yellow;">Scantime crypters are simple and very portable. This must be placed on the disk to be executed</mark>.

<mark style="color:yellow;">Runtime crypters use various injection techniques to decrypt the malicious payload and execute it in memory without ever touching the disk</mark>.

{% embed url="https://www.lmgsecurity.com/common-antivirus-bypass-techniques/" %}

### AV Evasion Tools:

* Veil-Framework: [https://github.com/Veil-Framework/Veil](https://github.com/Veil-Framework/Veil)
* Shellter: [https://www.shellterproject.com/](https://www.shellterproject.com/)
* Unicorn [https://github.com/trustedsec/unicorn](https://github.com/trustedsec/unicorn)
* UniByAV: [https://github.com/Mr-Un1k0d3r/UniByAv](https://github.com/Mr-Un1k0d3r/UniByAv)

### Obfuscation Tools:

PowerShell:

* Invoke-Obfuscation: [https://github.com/danielbohannon/Invoke-Obfuscation](https://github.com/danielbohannon/Invoke-Obfuscation)
* Chimera: [https://github.com/tokyoneon/Chimera](https://github.com/tokyoneon/Chimera)

Python:

* Pyarmor: [https://pypi.org/project/pyarmor/](https://pypi.org/project/pyarmor/)
* PyObfx: [https://github.com/PyObfx/PyObfx](https://github.com/PyObfx/PyObfx)

C#:

* ConfuserEx: [https://github.com/yck1509/ConfuserEx](https://github.com/yck1509/ConfuserEx)

### Payload Testing

This is by far the most effective way of testing your payloads. Remember that some distributors will send the results to other AV engines (i.e. virustotal).

* Nodistribute: [https://nodistribute.com/](https://nodistribute.com/)
* Virustotal: [https://www.virustotal.com/gui/home](https://www.virustotal.com/gui/home)
* Hybrid-Analysis: [https://www.hybrid-analysis.com/](https://www.hybrid-analysis.com/)
* Any-Run: [https://app.any.run](https://app.any.run/)
* Reverse.it: [https://reverse.it](https://reverse.it/)
* Anti-Virus Evasion Tool: [https://github.com/govolution/avet](https://github.com/govolution/avet)
* DefenderCheck: [https://github.com/matterpreter/DefenderCheck](https://github.com/matterpreter/DefenderCheck)
* ThreatCheck: [https://github.com/rasta-mouse/ThreatCheck](https://github.com/rasta-mouse/ThreatCheck)
