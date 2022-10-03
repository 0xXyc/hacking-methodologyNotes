# Active Directory Overview

## Intro

* Active Directory is the most commonly used identity management service in the world
* 95% of Fortune 1000 companies implement the service in their networks
* It can be exploited without ever attacking patchable exploits
  * Instead, you can abuse features, trusts, components, and more

## The Reality...

* It is usually way harder to get into a network
* However, most of the time, the network's security begins to fall apart once you are in

## Physical AD Components

### Domain Controllers (DC)

* A DC is a server with the AD DS server role installed that has specifically been promoted to a domain controller

### DC Characteristics

* Host a copy of the AD DS directory store
* Provide authentication and authorization services
* Replicate updates to other DC's in the domain and forest
* Allow administrative access to manage user accounts and network resources

### AD DS Data Store

* This data store contains the db files and processes that store and manage directory information for users, services, and applications
* Consists of the Ntds.dit file
  * This file contains the hashes of all users in the domain
* Stored by default in the %SystemRoot%\NTDS folder on all DC's
* The data store is only accessible through the DC processes and protocols
