---
description: 09/09/2025
---

# Elevated Host Persistence

This takes place after we've elevated our privileges on a host, we can also add persistence mechanisms to maintain `SYSTEM` access.

This allows us to be able to maintain elevated access to a machine without having to exploit a vulnerability again (because this may be patched or fixed at a later date).

{% hint style="info" %}
For these methods to work, you must be running in a _**High Integrity Beacon**_.
{% endhint %}

{% hint style="danger" %}
Remember that SYSTEM processes cannot authenticate to the web proxy, so we can't use HTTP Beacons. Use P2P or DNS Beacons instead.
{% endhint %}

## Windows Services

As we can clearly see through the enumeration process, there are many Windows services that run as `SYSTEM`.

Our various means of exploiting services for privilege escalation also act as persistence, but at the cost of breaking the legitimate service.

Instead, we can opt-in for a more reliable and safer approach of creating our own service which won't impact on existing services.

```
beacon> cd C:\Windows
beacon> upload C:\Payloads\tcp-local_x64.svc.exe
beacon> mv tcp-local_x64.svc.exe legit-svc.exe

beacon> execute-assembly C:\Tools\SharPersist\SharPersist\bin\Release\SharPersist.exe -t service -c "C:\Windows\legit-svc.exe" -n "legit-svc" -m add

[*] INFO: Adding service persistence
[*] INFO: Command: C:\Windows\legit-svc.exe
[*] INFO: Command Args: 
[*] INFO: Service Name: legit-svc

[+] SUCCESS: Service persistence added
```

#### What did this do?

This creates a new service in a _**STOPPED**_ state, but with the `START_TYPE` set to `AUTO_START`.&#x20;

_<mark style="color:yellow;">**This means the service won't run until the machine is rebooted. When the machine starts, so will the service, and it will be waiting for a connection**</mark>_.

<figure><img src="../.gitbook/assets/image (292).png" alt=""><figcaption></figcaption></figure>

## Windows Management Instrumentation (WMI) Event Subscriptions

Persistence can also be accomplished via WMI events.

They can be achieved by leveraging the following three classes:

* `EventConsumer`
* `EventFilter`
* `FilterToConsumerBinding`

### EventConsumer

An `EventConsumer` is the action that we want to perform. In this case, we want to execute a payload.

This can be via OS commands (e.g. PowerShell one-liner or VBScript).

### EventFilter

This is a trigger that we can act upon.&#x20;

Any arbitrary WMI query can be used as a filter which provides practically unlimited options. These can include when a particular process starts such as:

* When a user logs in
* When a USB device is inserted
* Any specific time of day or on a timed interval

### FilterToConsumerBinding

This links an `EventConsumer` and `EventFilter` together.

[PowerLurk](https://github.com/Sw4mpf0x/PowerLurk) is a PowerShell tool for building these WMI events.  In this example, I will upload a DNS payload into the Windows directory, import PowerLurk.ps1 and create a new WMI event subscription that will execute it whenever notepad is started.

You can view these classes afterwards using `Get-WmiEvent -Name WmiBackdoor`.  The _CommandLineTemplate_ for the EventConsumer will simply be `C:\Windows\dns_x64.exe`; and query for the EventFilter will be `SELECT * FROM Win32_ProcessStartTrace WHERE ProcessName='notepad.exe'`.

Open notepad on Workstation 2 and the DNS Beacon will appear:

<figure><img src="../.gitbook/assets/image (293).png" alt=""><figcaption></figcaption></figure>

The backdoor can be removed with: `Get-WmiEvent -Name WmiBackdoor | Remove-WmiObject`.
