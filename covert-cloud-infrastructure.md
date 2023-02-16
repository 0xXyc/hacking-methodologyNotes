---
description: OpSec is everything
---

# Covert Cloud Infrastructure

## Security Considerations <a href="#covertcloudinfrastructure-securityconsiderations" id="covertcloudinfrastructure-securityconsiderations"></a>

* When using Tor, we need to be cautious of when we first connect to the first relay node as it will ALWAYS know your IP address.

## The Setup <a href="#covertcloudinfrastructure-thesetup" id="covertcloudinfrastructure-thesetup"></a>

Preferably, a place where we can SSH into or utilize proxychains with our command syntax to route all traffic through our server as a proxy. I want to create a VPN concentrator as well as a proxy for us to route all of our traffic through.

I am still up in the air on weather or not we should utilize Tor.

Keep in mind that a VPN will ACTIVELY encrypt all communications whereas a proxy will not.

* These servers will provide us with a reliable and stable gateway to our attack infrastructure
* We would SSH into the server after ensuring our VPN or Tor connection has been established
* Store necessary tools on the server to ensure proper OpSec and that nothing can be locally traced back to us in any way

## OpSec Checklist <a href="#covertcloudinfrastructure-opsecchecklist" id="covertcloudinfrastructure-opsecchecklist"></a>

1. Connect to VPN
2. Conduct DNS Leak Test and be sure to Google "What is my IP" to ensure that your IP is that of the VPN you are connecting to
3. SSH into the attack infrastructure
4. Begin operating

## Choosing Providers <a href="#covertcloudinfrastructure-choosingproviders" id="covertcloudinfrastructure-choosingproviders"></a>

I noticed that when it comes to security/privacy, there's always pro's and cons; choosing a cloud provider can come down to a bitter trade off of supporting much more anonymous cryptocurrency payment options or the ease of automation.

Cloud Providers some additional options (Although expensive):

* BitLaunch – Bitcoin payments and spawns servers on DigitalOcean/Linode
* Bithost – Similar

VPN Providers:

* AirVPN – [https://airvpn.org/](https://airvpn.org/)
* ProtonVPN – [https://protonvpn.com/](https://protonvpn.com/)
