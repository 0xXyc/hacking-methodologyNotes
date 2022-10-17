---
description: 'PWK: 300-348'
---

# SQLi

## Introduction

Structured Query Language Injection (SQLi) is an attack on a web application database server that allows malicious queries to be executed.

* This vulnerability occurs when a web application communicates with a back end DB using user input that has not been properly validated

General Rule of Thumb:

<mark style="color:yellow;">Simply put, if you see a login page, ATTEMPT SQLi</mark>

## How a DB Works

* The SQL statement used to <mark style="color:green;">retrieve data</mark> is "<mark style="color:yellow;">SELECT</mark>"
* The SQL clause that can be used to <mark style="color:green;">retrieve data from multiple tables</mark> is "<mark style="color:yellow;">UNION</mark>"
* The SQL statement that is used to add data is "INSERT"

Conclusion:

Retrieving data from a <mark style="color:yellow;">single table in a database</mark> -- SELECT&#x20;

Retrieving data from <mark style="color:yellow;">multiple tables in a databse</mark> -- UNION

## Links

{% embed url="https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection" %}
PayloadsAllTheThings GitHub Repo
{% endembed %}

{% embed url="https://www.securityidiots.com/Web-Pentest/SQL-Injection/" %}
SQLi Logic
{% endembed %}

## Manual MSSQL Injection

Do you see a seemingly vulnerable input field that you believe queries the database?

<mark style="color:yellow;">Make a request to it and capture it in burp // send to repeater tab</mark>

```
q=500' union select 1 -- 
q=500' union select 1,2 -- 
q=500' union select 1,2,3 -- 
q=500' union select 1,2,3,4 -- 
q=500' union select 1,2,3,4,5 -- 
q=500' union select 1,2,3,4,5,6,7 -- 
q=500' union select 1,2,3,4,5,6,7,8 -- 
```

* Be sure to check the response tab after you send every request as you do not know when it will return data!

### URL-based SQLi

* ID-based, this means that an object could possibly be referencing a back-end database
* Of course this is similar to Indirect Object Reference, or IDOR

Example:

`https://website/blog?id=1`

* We can attempt to mess with the ID parameter because it is calling the database

Instead, we can try:

`https://website.thm/blog?id=`<mark style="color:yellow;">`2;--`</mark>

* This will then make a call to a different table from the database as long as it is set to public

### In-Band SQLi

* In-Band SQLi is the easiest type of SQLi to detect and exploit
* In-Band means that the same method of communication that is used to exploit the database also has the ability to allow the attacker to receive the results
* <mark style="color:yellow;">Example: Exploit</mark> SQLi on page & <mark style="color:yellow;">Receive</mark> on the same page

### Error-Based SQLi

* This type of SQLi is the most useful for easily obtaining information about the database structure and what type of database is operating on the back-end
* Database errors are printed to the screen
* Can be used to enumerate a whole database

### Union-Based SQLi

* This type of SQLi utilizes the SQL UNION operator alongside a SELECT statement to return additional results
* This method is most common for extracting large amounts of information because it pulls from multiple tables

## Semi-Automatic SQLi

{% embed url="https://www.youtube.com/watch?v=c2OFrDVb3EM&t=2558s" %}
Go to 45 mins
{% endembed %}

1. Set URL variable ($URL) to your target you are attacking

```
export URL=http://192.168.1.181/index.php
echo $URL
```

&#x20; 2\. Find the backend processing for the handling of your input in the web application

* View source code
* Search for form elements they look like this: <mark style="color:yellow;">\<form</mark>
* Locate the script or action that it takes within that line of code it should be executing a php file
  * i.e. if it is <mark style="color:yellow;">checklogin.php</mark>
  * Make it your URL variable

```
export URL="http://192.168.1.181:80/checklogin.php"
```

&#x20;  3\. Locate quick-SQLi.txt, it should be something like: usr/share/seclists/Fuzzing/SQLi/quick-SQLi.txt

&#x20;  4\. Load up wfuzz:

* Check <mark style="color:yellow;">\<input name</mark>, in this case, it is <mark style="color:yellow;">myusername</mark>

```
wfuzz -c -z file,/usr/share/seclists/Fuzzing/SQLi/quick-SQLi.txt -d "myusername=admin&mypassword=FUZZ&Submit=Login" --hc 404 "$URL"
```

* You may get get a lot back
* Look at the chars
* Go back to the web app and manually authenticate using that payload and username
* Go through them until you get an error
* Determine what a valid and invalid error is depending on char size
* Be sure to block out the invalid errors in wfuzz as seen below
* For example, these chars are 109 and 444

```
wfuzz -c -z file,/usr/share/seclists/Fuzzing/SQLi/quick-SQLi.txt -d "myusername=admin&mypassword=FUZZ&Submit=Login" --hc 404 --hh 109,444 "$URL"
```

* Anything you get now should be attempted manually once more in the browser
