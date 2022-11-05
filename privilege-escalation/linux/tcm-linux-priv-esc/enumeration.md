# Enumeration

## System Enumeration

### You Just Landed on a box...

1. <mark style="color:red;">`uname -a`</mark> -- Look for Kernel Exploit
2. <mark style="color:red;">`lscpu`</mark> <mark style="color:red;"></mark><mark style="color:red;"></mark> -- CPU Architecture
3. &#x20;<mark style="color:red;">`ps aux`</mark> -- What services are running?

## User Enumeration

* <mark style="color:yellow;">You are performing this type of enumeration to find out who you are, what permissions you have, and what you are capable of doing</mark>

1. <mark style="color:red;">`whoami`</mark> -- Display current user
2. <mark style="color:red;">`id`</mark> -- Permissions of current user
3. <mark style="color:red;">`sudo -l`</mark> -- What are the SUDO permissions of the current user | <mark style="color:yellow;">Very IMPORTANT</mark>
4. Ask yourself what files do you know of and what files can you access
5. Access to sensitive files? /etc/shadow /etc/passwd
6. <mark style="color:red;">`history`</mark> -- Display command/Terminal history

## Network Enumeration

* This type of enumeration allows you to identify internal ports and gives you a lay of the land

1. <mark style="color:red;">`ifconfig`</mark> or <mark style="color:red;">`ip a`</mark> -- General IP and Networking information -- Is this dual-homed (two NICs)
2. <mark style="color:red;">`ip route`</mark> -- View routing table
3. <mark style="color:red;">`arp -a`</mark> or <mark style="color:red;">`ip neigh`</mark> -- View ARP table
4. <mark style="color:red;">`netstat -ano`</mark> or <mark style="color:red;">`netstat -tulnp`</mark>

## Password Hunting

* Password hunting or sensitive file hunting

### Grepping for Potentially Lucrative Strings

password:

```
grep --color=auto -rnw '/' -ie "PASSWORD" --color=always 2> /dev/null

grep --color=auto -rnw '/' -ie "PASSWORD=" --color=always 2> /dev/null

grep --color=auto -rnw '/' -ie "PWD" --color=always 2> /dev/null

grep --color=auto -rnw '/' -ie "PASS" --color=always 2> /dev/null
```

File name:

```
locate password | more 
```

### SSH Keys

```
find / -name authorized_keys 2> /dev/null

find / -name id_rsa 2> /dev/null
```

