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
3. Disable password authentication -- Keys ONLY
4. Lockdown logins -- harden SSH (disable root login)
5. Firewall

## SSH and ssh-copy-id!

Generate a new user with useradd

* Be sure to remember or save the local password in a secure way
* You will need it later for ssh-copy-id to transfer the new key to the host

How to:

1. `ssh-keygen`
2. `ssh-copy-id -i key.pub new-user@8.8.8.8`
3. `ssh new-user@8.8.8.8 -i key`
