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

## Hunting Breached Credentials w/ DeHashed

{% embed url="https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwipn4-Cg8T6AhX1UjUKHZ_9DrsQFnoECAsQAQ&url=https%3A%2F%2Fwww.dehashed.com%2F&usg=AOvVaw0fxW3Et6Alk8dM7ZsWquw9" %}

* We can search my email, username, IP, name, address, phone, VIN, or even domain
* <mark style="color:yellow;">Should we consider buying this for a week? -- $5</mark>
