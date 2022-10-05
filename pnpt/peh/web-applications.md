---
description: My enemy
---

# Web Applications

## Installing Go

{% embed url="https://github.com/Dewalt-arch/pimpmykali" %}

## Finding Subdomains w/ Assetfinder

{% embed url="https://github.com/tomnomnom/assetfinder" %}

Automating Assetfinder:

<mark style="color:yellow;">assets.sh</mark>

```
#!/bin/bash

$url=$1

if [ ! -d "$url" ];then
    mkdir $url
fi

if [ ! -d "$url/recon" ];then
    mkdir $url/recon
fi

echo "[+] Harvesting subdomains with assetfinder..."
assetfinder $url >> $url/recon/assets.txt
cat $url/recon/assets.txt | grep $1 >> $url/recon/final.txt
rm $url/recon/assets.txt
```

