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

