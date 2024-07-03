---
layout: wiki
title: Poisoning and Relay
cate1: Internals
cate2: Active Directory
description: Poisoning and Relay
keywords: Internals
---

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
