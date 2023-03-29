---
description: OpSec is everything
---

# 🌩 Covert Cloud Infrastructure

## Operational Security Considerations <a href="#covertcloudinfrastructure-securityconsiderations" id="covertcloudinfrastructure-securityconsiderations"></a>

* Connect to VPN <mark style="color:yellow;">before Tor</mark>
* <mark style="color:yellow;">When using Tor, we need to be aware that the VPN IP address will be visible, not our actual external IP address</mark>
* This way <mark style="color:green;">your ISP is entirely unknowing of your usage of Tor</mark>
* <mark style="color:yellow;">Connect to VPS after Tor</mark> -- Only the VPS will see our exit node IP address obtained from Tor

## The Setup <a href="#covertcloudinfrastructure-thesetup" id="covertcloudinfrastructure-thesetup"></a>

<mark style="color:yellow;">The infrastructure will consist of a multitude of technologies where we can establish a layered-security approach</mark>. Ultimately, we will SSH into or utilize proxychains with our command syntax to route all traffic through our server as a proxy. I want to create a VPN concentrator as well as a proxy for us to route all of our traffic through.

I am still up in the air on weather or not we should utilize Tor.

Keep in mind that a VPN will ACTIVELY encrypt all communications whereas a proxy will not.

* These servers will provide us with a reliable and stable gateway to our attack infrastructure
* We would SSH into the server after ensuring our VPN or Tor connection has been established
* Store necessary tools on the server to ensure proper OpSec and that nothing can be locally traced back to us in any way

## OpSec Checklist <a href="#covertcloudinfrastructure-opsecchecklist" id="covertcloudinfrastructure-opsecchecklist"></a>

1. Connect to VPN
2. Conduct DNS Leak Test and be sure to `curl ifconfig.me` to ensure that your IP is that of the VPN you are connecting to
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
