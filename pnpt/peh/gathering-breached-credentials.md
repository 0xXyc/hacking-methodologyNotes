# Gathering Breached Credentials

## Gathering Breached Credentials with Breach-Parse

<mark style="color:yellow;">This script will cross reference valid usernames/emails with credentials obtained from past breaches</mark>.

{% embed url="https://github.com/hmaverickadams/breach-parse" %}

Syntax Usage:

```
./breach-parse.sh @tesla.com tesla.txt

Extracting usernames...
Extracting passwords...
```

* With the information that we obtain from this script, we can execute a <mark style="color:yellow;">Credential Stuffing</mark> attack against the login form on Tesla's site.

### Difference between Credential Stuffing vs Password Spraying

<mark style="color:yellow;">Credential Stuffing</mark> is when you KNOW the password is valid for one or more accounts.

<mark style="color:yellow;">Password Spraying</mark> is more of a brute force and "blind fire" method where you run valid accounts against a large wordlist of possible passwords.
