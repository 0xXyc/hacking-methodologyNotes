---
description: 08/31/2025
---

# Beacon Interaction

By selecting **Payloads -> Windows Stageless Generate All Executable payloads, we can generate all payloads to a directory of our choice for convenience and testing.**

<figure><img src="../.gitbook/assets/image (263).png" alt=""><figcaption></figcaption></figure>

For example, I placed them all within the `C:\Payloads` directory that I made.

We can now see that we have a ton of different payloads to work with as we explored in the previous chapter.

<figure><img src="../.gitbook/assets/image (264).png" alt=""><figcaption></figcaption></figure>

## Executing `http_x64.exe`

as we can see, we get an alert in the **Event Log** stating `initial beacon from Attacker@10.10.5.40`...

This indicates a successful callback to our Team Server (C2).

<figure><img src="../.gitbook/assets/image (265).png" alt=""><figcaption></figcaption></figure>

We can type `help` for a list of commands within the `beacon` terminal.

<figure><img src="../.gitbook/assets/image (266).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Every minute, this Beacon will "check in" with the Team Server to check if there are "jobs" pending for it. This is in terms of `sleep` setting.

The longer the `sleep` setting, the stealthier we are.

For example, we can change the `sleep` time to `sleep 5` (this changes this setting from one minute to five seconds).
{% endhint %}

<figure><img src="../.gitbook/assets/image (267).png" alt=""><figcaption></figcaption></figure>

## Connections

`help connect` for more `connect` options.

**We can verify connections that we established with our payloads via the following command on a victim machine:**

```
netstat -anop tcp | findstr 4444
```

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

## Examining Over the Wire (OTW)

Launch Wireshark to examine the C2 traffic in an OTW context.

Since we are using HTTP for our C2 communications, we can follow the HTTP stream and examine the job queue as well as the communication between the Team Server and the Beacon.

We can right-click on the `GET` request and follow the HTTP stream for verbose information on the packet.

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

Upon doing so, we can follow the color coordinated text that corresponds with the client and the server HTTP communication.

<figure><img src="../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

* Red is the client (request made by the beacon via HTTP `GET` request) — it is asking the Team Server if there are any jobs that it needs to execute.
  * The random characters embedded within the URI are encoded metadata
* Blue is the server communication (Since there was no job available at the time, a `NOP` (no-operation) frame was sent.

{% hint style="info" %}
Note: Everything about how the traffic appears can be completely custom tailored within the _**Malleable C2 Profile**_.
{% endhint %}

### Creating a job and examining traffic

Let's list out files and see what it looked like within Wireshark.

**In the Beacon window, we can use `ls` to do so:**

<figure><img src="../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

We can see the following HTTP `GET` request for our current `ls` job:

<figure><img src="../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

Immediately proceeding is the corresponding HTTP `POST` request:

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

This contains the directory listing that is then returned back to our Team Server for file system analysis/enumeration.

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>
