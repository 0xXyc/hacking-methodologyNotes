# PEH

## Information Gathering (Reconnaissance)

What kind of information can we gather from the target?

### Discovering Email Addresses

* Look at OSINT information
* Sometimes this involves guessing in determining the naming convention
* However, names of employees are very common to find on the web
  * Especially higher ranked employees

OSINT Email Finder:

{% embed url="https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwiy-ZbjnLX6AhUrj4kEHW6_DtkQFnoECAcQAQ&url=https%3A%2F%2Fhunter.io%2F&usg=AOvVaw2FGHtpfGhCqvtI-SLk45G5" %}
Email Hunter
{% endembed %}

* Input a domain and you will get OSINT information on email addresses pertaining to that domain
* Identify the naming convention

Another fantastic resource:

{% embed url="https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwiEooSmnbX6AhWVrYkEHbRVC0AQFnoECA4QAQ&url=https%3A%2F%2Fphonebook.cz%2F&usg=AOvVaw0WREdmla4Pcs0JtWMA7eDu" %}
Phonebook Intelligence: email, domains, URLs
{% endembed %}

Browser Extension:

Clearbit -- Must use Google Chrome!

Email Verification:

Email Hippo

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
