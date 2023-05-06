# 💿 Linux Commands & Tricks

## Find any binary on a system

```
sudo find / -type f -name "openvpn"
```

## Locking Down Linux Cloud Servers

1. Enable automatic updates
   1. `apt update`
   2. `apt dist-upgrade`
   3. `apt install unattended-upgrades`
   4. `dpkg-reconfigure --priority=low unattended-upgrades`
2. Limited user account
   1. `add user ares`
   2. `usermod -aG sudo ares`
3. Lockdown logins -- harden SSH (disable root login, restrict to IPv4 only, and change default port) -- `sudo nano /etc/ssh/sshd_config`
4. Firewall!! -- `sudo apt install ufw`

## SSH and ssh-copy-id!

Generate a new user with useradd

* Be sure to remember or save the local password in a secure way
* You will need it later for ssh-copy-id to transfer the new key to the host

How to:

1. `ssh-keygen`
2. `ssh-copy-id -i key.pub new-user@8.8.8.8`
3. `ssh new-user@8.8.8.8 -i key`

## UFW

1. Installation: `sudo apt install ufw`
2. Establish an allow list for ports: `sudo ufw allow 1337`

## Problems with APT packages?

* Missing features
* Missing files
* etc

I found that you may need to do a hard reset on the APT package with the following commands:

```
sudo apt remove package
sudo atp-get purge --remove package
sudo apt update --fix-missing
sudo apt-get install package
```

## Installing the LATEST Impacket

{% embed url="https://github.com/fortra/impacket" %}

Remove current impacket:

```
python3 -m pip uninstall impacket
sudo rm -f /usr/bin/impacket*
```

install.sh:

```
git clone https://github.com/fortra/impacket.git
cd impacket
pip3 install -r requirements.txt
pip3 install .
cd ../
rm -rf impacket/
```

Update DB:

```
sudo updatedb
```

Locate new impacket directory (will usually be /home/user/.local/bin/here:

```
locate GetUserSPNs.py
```

Add to PATH:

Add the following to the end of \~/.zshrc:

```
export PATH=$PATH:/home/user_here/.local/bin
```

reset shell:

```
reset
```

Source \~/.zshrc:

```
source ~/.zshrc
```

You can now python3 GetUserSPNs.py from anywhere!

Uninstalling:

```
python3 -m pip uninstall impacket
```

Install latest:

```
./install.sh
```

## Installing the LATEST CME

{% embed url="https://github.com/Porchetta-Industries/CrackMapExec" %}

Please read this... it's so good lol:

{% embed url="https://github.com/Porchetta-Industries/CrackMapExec" %}

<mark style="color:yellow;">We will need to use Docker because me and a couple friends kept getting errors.</mark>

Install:

```
git clone https://github.com/Porchetta-Industries/CrackMapExec.git
cd CrackMapExec
```

Docker:

```
sudo docker build -t cme .
sudo docker run -it --entrypoint=/bin/bash --rm --name cmexec cme:latest
```
