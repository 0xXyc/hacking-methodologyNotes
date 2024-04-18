---
description: The coolest toolkit in the world
---

# Impacket Toolkit

## Introduction

* Impacket provides us with many different ways to enumerate, interact, and exploit Windows protocols and find the information we need using Python
* This tool is well maintained and has many contributors

## psexec.py

* This is <mark style="color:yellow;">one of the most useful tools</mark>
* This is a <mark style="color:yellow;">clone of the Sysinternals psexece executable</mark> but works slightly different
* <mark style="color:yellow;">The tool works by creating a remote service by uploading a randomly-named executable to the ADMIN$ share on the host</mark>
* It then registers the service via RPC
* <mark style="color:yellow;">Once established, communication occurs over a pipe, allowing for an interactive shell as SYSTEM on the victim</mark>

```
psexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.125
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## wmiexec.py

* wmiexec.py utilizes a semi-interactive shell where commands are executed through Windows Management Instrumentation or WMI
* It does not drop any files or executables on the target host and generates fewer logs than other modules
* Once connected, it will run the local admin user we connected with&#x20;
  * This is good because it is must stealthier than seeing SYSTEM executing commands

```
wmiexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.5
```

<figure><img src="../../.gitbook/assets/image (1) (7) (1).png" alt=""><figcaption></figcaption></figure>

## Windapsearch.py

* This is a tool that can be used to enumerate users, groups, and computers from a Windows admin by utilizing LDAP queries

### Domain Admins

Enumerate all Domain Admins:

```
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 --da
```

### Privileged Users

Enumerate all privileged users:

```
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 -PU
```

## BloodHound.py

Similar to the <mark style="color:yellow;">BloodHound and SharpHound duo</mark>, this is an <mark style="color:yellow;">all-in-one package</mark> that <mark style="color:yellow;">allows a remote attacker with valid domain credentials to be able to enumerate the entire AD domain</mark>!

Help:

```
bloodhound-python -h
```

Collection Method "all":

```
sudo bloodhound-python -u 'forend' -p 'Klmcargo2' -ns 172.16.5.5 -d inlanefreight.local -c all
```

<figure><img src="../../.gitbook/assets/image (8) (6).png" alt=""><figcaption></figcaption></figure>

* After the command completes, you will see four .json files

<figure><img src="../../.gitbook/assets/image (5) (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

* We need to upload these into BloodHound, but the easiest way is to zip them and just place one zip file in BloodHound

```
zip -r domain_BH_info.zip *.json
```

* This will zip up all of the .json files and we can upload this data into BloodHound!
