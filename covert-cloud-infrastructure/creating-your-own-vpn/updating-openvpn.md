---
description: 07/21/2024
---

# Updating OpenVPN

## Introduction

In order to prevent breaches to the Confidentiality, Integrity, and Availability (CIA) of your data on your server, it is important to ensure that your underlying software is running the latest versions.

```
apt-mark hold openvpn-as
sudo apt-get update
sudo apt-get upgrade
sudo apt update
sudo rm /etc/apt/sources.list.d/openvpn-packages.list
sudo tee /etc/apt/sources.list.d/openvpn-aptrepo.list <<EOF
deb http://build.openvpn.net/debian/openvpn/stable $(lsb_release -cs) main EOF

wget -O - https://build.openvpn.net/debian/openvpn/stable/pubkey.gpg | sudo apt-key add -
sudo apt update -y && sudo apt upgrade -y
sudo apt install openvpn
sudo apt upgrade openvpn
openvpn --version
```
