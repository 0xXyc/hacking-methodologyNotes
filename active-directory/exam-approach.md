# Exam Approach

## AD Methodology

1. Initial Foothold -> Escalate Privileges to Local Admin -> Escalate to Domain Admin
2. Scanning/Enumeration
3. Pivoting & Tunneling
4. Exploitation
5. Post Exploitation&#x20;
6. Move onto standalone boxes

* My biggest question still remains, how does the initial foothold into the AD set work on the exam?

## Initial Foothold

* Enumerate the entire box
* Escalate privileges to Local Admin
* Network Enumeration
  * What do we need to pivot to?
  * Multiple network interfaces?
  * What tool do we need to do this?
    * Attempt SSHuttle, plink.exe, and port forwarding

## Scanning/Enumeration

* Configure tool or technique to pivot to other boxes in the domain
* Port/service scan the new boxes
* Check the network shares

## Post Exploitation

* Deploy <mark style="color:yellow;">ADPeas</mark>
  * <mark style="color:yellow;">Go through slowly</mark>
  * <mark style="color:yellow;">Hash? Crack with Hashcat</mark>
* Deploy <mark style="color:yellow;">WinPEAS</mark>
* <mark style="color:yellow;">AS-REP Roasting</mark> (User with Kerberos pre-authentication disabled) -- Username required no password

Cracking:

```
hashcat -m 18200 -a 0 <AS_REP_responses_file> <passwords_file>
```

* <mark style="color:yellow;">Kerberoasting</mark> (User with SPN) -- Requires username and password

Cracking:

```
hashcat -m 13100 --force <TGSs_file> <passwords_file>
```

[https://gist.github.com/TarlogicSecurity/2f221924fef8c14a1d8e29f3cb5c5c4a](https://gist.github.com/TarlogicSecurity/2f221924fef8c14a1d8e29f3cb5c5c4a)

