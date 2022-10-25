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

