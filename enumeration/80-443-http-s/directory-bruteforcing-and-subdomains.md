# Directory Bruteforcing and Subdomains

## Directory Bruteforcing

```
direarch -u http://trick.htb
```

## Subdomain Bruteforcing

* Be sure to run without -fs or -ms for a few seconds in the beginning
* Identify the HTTP response size and place it after the the two for proper filtering and clean output to easily identify if there is a subdomain existent on the target

```
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/namelist.txt -H "Host: FUZZ.trick.htb" -u http://trick.htb -fs 5480
```

Sometimes, you might need to fuzz in a different level of a subdomain:

```
ffuf -c -u http://preprod-payroll.trick.htb/ -w /usr/share/amass/wordlists/subdomains-top1mil-5000.txt -H "Host: preprod-FUZZ.trick.htb" -ms 9660
```

