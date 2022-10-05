---
description: My enemy
---

# Web Applications

## Installing Go

{% embed url="https://github.com/Dewalt-arch/pimpmykali" %}
Repo
{% endembed %}

## Finding Subdomains w/ Assetfinder

{% embed url="https://github.com/tomnomnom/assetfinder" %}
Repo
{% endembed %}

Automating Assetfinder:

<mark style="color:yellow;">assets.sh</mark>

```
#!/bin/bash

url=$1

if [ ! -d "$url" ];then
    mkdir $url
fi

if [ ! -d "$url/recon" ];then
    mkdir $url/recon
fi

echo "[+] Harvesting subdomains with Assetfinder..."
assetfinder $url >> $url/recon/assets.txt
cat $url/recon/assets.txt | grep $1 >> $url/recon/final.txt
rm $url/recon/assets.txt

#echo "[+] Harvesting subdomains with Amass..."
#amass enum -d $url >> $url/recon/f.txt
#sort -u $url/recon/f.txt >> $url/recon/final.txt
#rm $url/recon/f.txt
```

Dry run:

```
chmod +x run.sh
./run.sh tesla.com
cd tesla.com/recon
cat final.txt
```

## Finding Subdomains w/ Amass

{% embed url="https://github.com/OWASP/Amass" %}
Repo
{% endembed %}

```
amass enum -d tesla.com
```

## HTTProbe

{% embed url="https://github.com/tomnomnom/httprobe.git" %}
Repo
{% endembed %}

