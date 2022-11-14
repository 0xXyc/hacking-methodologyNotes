# 🛠 Troubleshooting

## Resetting Linux Password

* Reboot into the GRUB menu
* Press 'e' on the live boot option
* You will be presented with single user mode

<figure><img src=".gitbook/assets/image (2) (7).png" alt=""><figcaption></figcaption></figure>

* You will now be changing the <mark style="color:yellow;">`ro`</mark> to <mark style="color:yellow;">`rw`</mark> (Read-Only -> Read-Write)
* Also, add the following <mark style="color:yellow;">`init=/bin/bash`</mark> at the end of the line
* Then, press CTRL+X to boot into /bin/bash as root!

It should look something like this:

<figure><img src=".gitbook/assets/image (1) (2) (2) (1).png" alt=""><figcaption></figcaption></figure>

* Then utilize `passwd <username>` to change the password of the specified user!
* Use `passwd` by itself to change the root password
* Reboot, and login!

<figure><img src=".gitbook/assets/image (4) (5) (1).png" alt=""><figcaption></figcaption></figure>

## Installing and Configuring Fish Shell

{% embed url="https://www.vultr.com/docs/installing-fish-shell-on-ubuntu/" %}

* Use <mark style="color:yellow;">`fish_config`</mark> to modify the fish shell from the browser!

## Unzipping rockyou.txt

```
sudo gzip -d /usr/share/wordlists/rockyou.txt.gz
```

## Installing JohnTheRipper (Password Cracking Utility)

* I recently ran into an issue where John would not work from the apt repo
* The one fix I found was downloading the source from GitHub and building it yourself, locally.

Install recommended tools:

```
sudo apt-get install yasm libgmp-dev libpcap-dev libnss3-dev libkrb5-dev pkg-config libbz2-dev zlib1g-dev
```

Create a \~/src directory:

```
mkdir ~/src

cd ~/src
```

Download the latest version:

```
git clone git://github.com/magnumripper/JohnTheRipper -b bleeding-jumbo john
```

Navigate to project:

```
cd ~/src/john/src
```

Build:

```
./configure && make -s clean && make -sj4
```

Run test:

```
../run/john --test
```

* What weirdly worked for me may not work for you, but attempt to run <mark style="color:red;">`sudo apt install john`</mark> to see if john can be ran globally now
