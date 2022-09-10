---
description: Goal low priv -> root
---

# 🐧 Linux

## Introduction

* Understanding how Linux handles permissions is critical to having success at privilege escalation

## Users, Groups, Files, and Directories

* Permissions in Linux are a relationship between users, groups, files, and directories
* <mark style="color:yellow;">Users</mark> can have multiple <mark style="color:yellow;">groups</mark>
* <mark style="color:yellow;">Groups</mark> can have multiple <mark style="color:yellow;">users</mark>
* Every <mark style="color:yellow;">file</mark> and <mark style="color:yellow;">directory</mark> defines its permissions in terms of a <mark style="color:yellow;">user</mark>, <mark style="color:yellow;">group</mark>, and <mark style="color:yellow;">others</mark>

### Users

* User <mark style="color:yellow;">accounts</mark> are configured within the <mark style="color:yellow;">/etc/passwd file</mark>
* User <mark style="color:yellow;">password</mark> hashes are stored in the <mark style="color:yellow;">/etc/shadow file</mark>
* Users are identified by an integet user ID (UID)
* Root has a UID of 0, which grants this user access to every file

### Special Permissions

#### setuid (SUID) bit

* When set, files will get executed with the privileges of the file owner

#### setgid (SGID) bit

* When set on a file, the file will get executed with the privileges of the file group
* When set on a directory, files created within that directory will inherit the group of the directory itself

## How to read permissions

#### Example

`-rwxr-xr-x 1 root root 60416 Apr 28 2022 /bin/date`

* Permissions are defined in terms of read, write, and execute
* There are three sets of permissions:
  * owner
  * group
  * other users ("world")
* Only the owner can change permissions

## Spawning Root Shells

* As stated prior, the ultimate goal is to spawn a root shell
* The end result is always going to be executing /bin/sh or /bin/bash
  * With that said, there are multiple ways of achieving this execution

### "rootbash SUID"

* You can spawn a root shell by creating a copy of the /bin/bash executable file
* This can be accomplished by making sure that /bin/bash is owned by root, and has the SUID bit set
* Once verified, the root shell can be spawned by executing the newly copied /bin/bash file with the -p command line option
  * The -p argument will allow the shell to be run with SUID permissions
* The benefit of this method is that you will be able to maintain persistence by&#x20;

#### How-To:

```
cp /bin/bash rootbash

./rootbash -p
```

### Catching a Reverse Shell

{% embed url="https://github.com/mthbernardes/rsg" %}

* All of these shells can be "caught" via netcat

## Automated Tools

* Tools allow us to automate the reconnaissance that can identify potential privilege escalations

### Linux Smart Enumeration (lse.sh)

{% embed url="https://github.com/diego-treitos/linux-smart-enumeration" %}
GitHub Repository
{% endembed %}

#### How to use it?

The idea is to get the information gradually.

First you should execute it just like `./lse.sh`. If you see some green `yes!`, you probably have already some good stuff to work with.

If not, you should try the `level 1` verbosity with `./lse.sh -l1` and you will see some more information that can be interesting.

If that does not help, `level 2` will just dump everything you can gather about the service using `./lse.sh -l2`. In this case you might find useful to use `./lse.sh -l2 | less -r`.

### LinEnum

* LinEnum is an advanced Bash script that extracts a large amount of useful information from the compromised system

{% embed url="https://github.com/rebootuser/LinEnum" %}
GitHub Repository
{% endembed %}

Example:

```
./LinEnum.sh -k password -e export -t
```
