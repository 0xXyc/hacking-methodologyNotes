---
description: 11/12/2025
---

# ✅ Exam Prep

## Enabling Windows Defender

Windows Defender is disabled on most of the machines in the RTO lab.  This is so that you can get familiar with the various attacks and techniques without that added layer of difficulty.  In contrast, Windows Defender is enabled on every machine in the exam, which can be quite a step up if you're not prepared.  The most effective way you can prepare for the exam is to go back over the TTPs covered in the RTO course and lab, but with Defender enabled.  This page will guide you through the process of how you can do that.

Defender is disabled via GPO, which you can see by opening Group Management Console (GPMC) on Domain Controller 2 (_dc-2.dev.cyberbotic.io_).  The GPO is simply called "Windows Defender" and is linked to the Domain Controllers, SQL Servers, Web Servers and Workstations OU.

<figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

To re-enable Defender on a group of the machines, simply disable the GPO link on the corresponding OU by right-clicking on the GPO and unchecking "Link Enabled".

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

You can wait for the natural group policy refresh cycle; or log into the console of each machine and manually run `gpupdate` from a command prompt and then reboot each machine.

```
C:\Users\bfarmer>gpupdate /force
Updating policy...

Computer Policy update has completed successfully.
User Policy update has completed successfully.
```

Defender should be enabled on the next boot.  If see this error message inside the Windows Security settings, click _Restart now_ and reboot again.

![](https://files.cdn.thinkific.com/file_uploads/584845/images/44c/eee/c3e/windows-security.png)

**To test AMSI, use the AMSI Test Sample PowerShell cmdlet:**

```
Invoke-Expression 'AMSI Test Sample: 7e72c3ce-861b-4339-8740-0ac1484c1386'
```

**You should see that the cmdlet gets blocked:**

```
At line:1 char:1
+ Invoke-Expression 'AMSI Test Sample: 7e72c3ce-861b-4339-8740-0ac1484c ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This script contains malicious content and has been blocked by your antivirus software.
```

**However, this error means that Defender is not running properly:**

```
AMSI: The term 'AMSI' is not recognized as a name of a cmdlet, function, script file, or executable program.
Check the spelling of the name, or if a path was included, verify that the path is correct and try again.
```

**To test on-disk detections, drop the EICAR test file somewhere such as the desktop:**

```
X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*
```

![](https://files.cdn.thinkific.com/file_uploads/584845/images/f2b/a36/dfb/eicar.png)
