# Web Login Page Checklist

## Bruteforcing

### Hydra MYSQL

```
hydra -l root -P /root/rockyou.txt mysql://192.168.52.88:3306
```

## Hydra HTTP Form

{% code overflow="wrap" %}
```
hydra 192.168.167.70 http-form-post "/ctl00%24ContentPlaceHolder1%24UsernameTextBox=^USER^&ctl00%24ContentPlaceHolder1%24PasswordTextBox=^PASS^: Incorrect username or password..." -L users.txt -P /root/rockyou_utf8.txt -vV -f
```
{% endcode %}

Or

{% code overflow="wrap" %}
```
hydra 10.10.95.178 http-form-post "/squirrelmail/src/login.phpp:login_username=^USER^&secretkey=^PASS^&js_autodetect_results=1&just_logged_in=1:Unknown user or password incorrect." -l milesdyson -P passwords.txt -vV
```
{% endcode %}

## Wordpress (wpscan)

{% code overflow="wrap" %}
```
wpscan  --url  http://192.168.132.78/wordpress/ -U max -P /usr/share/wordlists/rockyou.txt
```
{% endcode %}

## SQLi and Bypass

Username:

```
' or 1=1-- -
```

Password:

```
' or 1=1-- -
```

{% embed url="https://book.hacktricks.xyz/pentesting-web/login-bypass" %}

## Common or default passwords

* Look up CMS/version or common passwords for login
* Try the basics

## Password Reset Functionality

Make sure that a user can only reset her own password. If a privileged user can reset other users’ passwords, make sure that the super user cannot see the new password.&#x20;

During password change, intercept the traffic with burp and check if username is present in the request. If it is, change the username to see what happens.&#x20;

You might change someone else’s password this way. After a password reset, user should be informed via e-mail or sms.&#x20;

If the system requires high level security, password reset functionality should try to id the user before sending a password reset e-mail or code.&#x20;

If the system sends back your forgotten password, it means they are keeping it in somewhere which is always a bad idea. (MediaMarkt used to do this)

## Can you register as a new user?

Try to register as a new user, this could get us RCE as a low level user or get us further clarifying information (like CMS version)

​Testing for user registration and provisioning

Ask the following questions and try to find answers. There is no wright or wrong answer. You should decide if the answer makes sense or not.

1. Can anyone register for access?
2. Are registrations vetted by a human prior to provisioning, or are they automatically granted if the criteria are met?
3. Can the same person or identity register multiple times?
4. Can users register for different roles or permissions?
5. What proof of identity is required for a registration to be successful?
6. Are registered identities verified?
7. Can identity information be easily forged or faked?
8. Can the exchange of identity information be manipulated during registration?
9. Is there any verification, vetting and authorization of provisioning requests?
10. Is there any verification, vetting and authorization of de-provisioning requests?
11. Can an administrator provision other administrators or just users?
12. Can an administrator or other user provision accounts with privileges greater than their own?
13. Can an administrator or user de-provision themselves?
14. How are the files or resources owned by the de-provisioned user managed? Are they deleted? Is access transferred?

## Username Enumeration

Enter a valid username and a wrong password. Note the warning you get. then Enter an invalid username and a password. If you get a different warning this means you can collect the valid usernames using brute-force. For example I registered to etsy.com with desidero44@yahoo.com mail address and tried to login with a wrong password, then tried to login with desidero55@yahoo.com which is not registered to etsy.com at all.

* If there is a directory that can be accessed with the username like site.com/users/my\_user\_name we can try to access the page with different user names. You might get a 403 forbidden for registered users and 404 for invalid usernames. Test forgot password functionality to see if generates different error messages for valid and invalid usernames. An advanced technique is to measure the server response time for valid and invalid usernames. Server might take longer to respond to valid user names.































