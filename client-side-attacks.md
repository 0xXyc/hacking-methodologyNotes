---
description: 09-26-22
---

# 👨💼 Client Side Attacks

Client side attacks are attacks in which the threat actor is attempting to gain access to the network that utilizes <mark style="color:yellow;">techniques that require a victim's user interaction</mark>.&#x20;

This includes:

* <mark style="color:yellow;">Social Engineering</mark>
* <mark style="color:yellow;">Enticing them to click a link</mark>
* <mark style="color:yellow;">Open a document</mark>
* <mark style="color:yellow;">Send them to your malicious website</mark>

### Social Engineering

This is one of the most common tactics that can be used to execute proper client side attacks. Depending on the tactic you use or the information you gathered during the information gathering stage, you will have a higher chance of success for the client to click on it.

<mark style="color:yellow;">Examples of common client side attacks</mark>:

* <mark style="color:yellow;">Understanding HTA Attacks</mark>: [https://www.trustedsec.com/blog/malicious-htas/](https://www.trustedsec.com/blog/malicious-htas/)
* <mark style="color:yellow;">Creating HTA Files with Empire</mark>: [https://dmcxblue.gitbook.io/red-team-notes/initial-acces/spear-phishing-links/tools](https://dmcxblue.gitbook.io/red-team-notes/initial-acces/spear-phishing-links/tools)
* <mark style="color:yellow;">Template for creating your own</mark>: [https://github.com/tjnull/OSCP-Stuff/blob/master/Client-Side-Attacks/Template.HTA](https://github.com/tjnull/OSCP-Stuff/blob/master/Client-Side-Attacks/Template.HTA)

<mark style="color:yellow;">Tools to use for HTA Attacks</mark>:

* Demiguise: [https://github.com/nccgroup/demiguise](https://github.com/nccgroup/demiguise)
* WeirdHTA: [https://github.com/felamos/weirdhta](https://github.com/felamos/weirdhta)
* SharpShooter: [https://github.com/mdsecactivebreach/SharpShooter](https://github.com/mdsecactivebreach/SharpShooter)

<mark style="color:yellow;">Microsoft Office Macros (Maldoc)</mark>:

* Malicious Macros: [https://www.trustedsec.com/blog/malicious-macros-for-script-kiddies/](https://www.trustedsec.com/blog/malicious-macros-for-script-kiddies/)
* Creating your own Maldoc: [https://www.pentestpartners.com/security-blog/how-to-create-poisoned-office-documents-for-your-staff-awareness-training-part-1/](https://www.pentestpartners.com/security-blog/how-to-create-poisoned-office-documents-for-your-staff-awareness-training-part-1/)
* Building Obfuscated Macros: [https://blog.focal-point.com/how-to-build-obfuscated-macros-for-your-next-social-engineering-campaign](https://blog.focal-point.com/how-to-build-obfuscated-macros-for-your-next-social-engineering-campaign)

<mark style="color:yellow;">Tools to help you build your own Macros</mark>:

I would use these tools to learn how to make your own. Be creative when you are building your own Macros as using tools like this will be flagged by AV

* MSFVenom Vbscript Injections: [https://www.offensive-security.com/metasploit-unleashed/vbscript-infection-methods/](https://www.offensive-security.com/metasploit-unleashed/vbscript-infection-methods/)
* Macropack: [https://github.com/sevagas/macro\_pack](https://github.com/sevagas/macro\_pack)
* EvilClippy: [https://github.com/outflanknl/EvilClippy](https://github.com/outflanknl/EvilClippy)
