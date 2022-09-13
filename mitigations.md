---
description: We broke some things... now let's fix them!
---

# 🛡 Mitigations

## Kerberos

### "Double Kerberos Reset"

* I strongly believe that it should be a best practice to restart Kerberos services daily or at minimum once a week to reissue tickets amongst users throughout the domain
* I need to find more documentation on this method, but you restart Kerberos services twice
* This will make kerberoasting very difficult to do for an attacker; hence the assumed breach mindset
