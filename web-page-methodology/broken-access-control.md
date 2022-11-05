# Broken Access Control

## Introduction

This one is very simple, but very dangerous.

Simply put, this is a vulnerability that allows an unauthenticated user to go anywhere they want on the site.

## Exploit Opportunities

* We can exploit this in a number of ways

This includes:

* &#x20;IDOR vulnerabilities
* Simply navigating to a page and it grants you access
* Breaking access to certain authentications

Remember there are three levels of authentication:

* Admin
* Unauthenticated
* Authenticated

## Exploitation

* Be sure to use the Inspector tool in the developer tools
* Look for hidden fields
* You may be able to reveal these fields and replace them with user ID's such as 1 for admin
