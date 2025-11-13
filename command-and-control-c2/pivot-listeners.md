---
description: 09/01/2025
---

# ✅ Pivot Listeners

## What is a Pivot Listener?

A pivot listener can only be created on an existing Beacon, and not via the normal Listeners menu.&#x20;

Essentially, these listeners work in the same way as traditional TCP listeners, but in reverse.

A standard TCP Beacon payload will bind to local host (`127.0.0.1` or `0.0.0.0`) and listens for an incoming connection on the specified port.&#x20;

You then initiate a connection to it from an existing Beacon (with the `connect` command).

The pivot listener works the other way around by telling the existing Beacon to bind and listen on a port. The new Beacon TCP payload initiates a connection to it instead.

### Creating a Pivot Listener

Right-click on a Beacon and select **Pivoting** -> **Listener**. This will open a **New Listener** window.

<figure><img src="../.gitbook/assets/image (9) (1) (1).png" alt=""><figcaption></figcaption></figure>

Notice how the payload type is a `beacon_reverse_tcp`, rather than a `beacon_bind_tcp`.&#x20;

Meaning that rather than opening a port on the victim via Beacon, a reverse connection is established, connecting back to the Team Server (reverse shell vs. bind shell).

I created the new listen port to listen on `1337`.&#x20;

<figure><img src="../.gitbook/assets/image (10) (1) (1).png" alt=""><figcaption></figcaption></figure>

As a result, we can now generate payloads for this listener, it becomes available in all the usual commands such as `spawn`, `elevate`, and `jump`.

Once executed, the reverse TCP Beacon will appear immediately within the UI and the arrow in the graph view (GUI) will show the direction of the connection.
