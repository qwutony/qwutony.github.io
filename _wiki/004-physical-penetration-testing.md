---
layout: wiki
title: Physical Penetration Testing
cate1: Physical
cate2: 
description: Physical Penetration Testing Notes
keywords: Physical
---

# Physical Penetration Testing
Covert entry strategies to allow attackers to bypass physical security mechanisms to gain unauthorized entry to a building. Resources retrieved from various talks - credit provided below.

## Covert Methods of Entry - Door Security
Assess the physical security of doors to identify potential vulnerabilities that could be exploited by unauthorized individuals to gain access to secure areas.

### Hinge Removal
Hinge removal is a common vulnerability in door security where an attacker removes the hinge pins, allowing the door to be lifted off its hinges.

<img width="479" alt="image" src="https://github.com/qwutony/qwutony.github.io/assets/45024645/8d23d3c1-313c-4ce8-9436-729007deac44">

**Requires:** 
  - Nail and Hammer
    - Basic tools that can be used to drive out the hinge pins from standard door hinges.

**Targets:** Hinge Pins

**Remediations:** 
  - Security Hinges
    - Use hinges with non-removable pins.
    - Opt for hinges with set screws that hold the pin in place.
  - Jamb Pin Screws
    - Install jamb pin screws in the door and frame. These are screws that extend from the door into the frame, preventing the door from being lifted off even if the hinge pins are removed.
  - Reinforced Hinge Plates
    - Use reinforced hinge plates that provide additional security and make it more difficult to access and remove the pins.  

### Door Latch
Door latches can be vulnerable to unauthorized access if manipulated with specialized tools.

<img width="397" alt="image" src="https://github.com/qwutony/qwutony.github.io/assets/45024645/65c8cfc8-dc2c-4b31-9da5-9ba63733d5bb">

**Requires:** 
  - Latch Tool
    - A specialized tool used to manipulate the latch mechanism of a door.
  - Wires
    - Thin wires or other tools used to bypass protective plates and access the latch mechanism directly.

**Targets:** Door Latches (can be manipulated to open the door without a key or proper access)

**Remediation:** 
  - Dead Latch mechanism
    - This mechanism engages a secondary latch that locks the primary latch in place, making it more resistant to tampering
  - Correct door fitment
    - Ensure that the door is properly fitted within its frame. A well-fitted door reduces gaps and spaces that can be exploited to manipulate the latch. Proper fitment also ensures that the dead latch mechanism functions correctly
  - Use of protective plates can be bypassed

<img width="492" alt="image" src="https://github.com/qwutony/qwutony.github.io/assets/45024645/db59430e-2ed3-45f7-b29a-2bf4a6bdb423">

### Crash Bars
Also known as push bars, panic bars or exit devices - used to allow quick egress in case of emergency. Consists of a horizontal bar spanning the width of the door that can be pushed from the inside to ensure a fast and efficient escape route.

<img width="395" alt="image" src="https://github.com/qwutony/qwutony.github.io/assets/45024645/64e5a395-72d3-48d3-bcbc-409300508bbb">

**Vulnerability:** Weather stripping (such as brush or rubber) in between the crash bars is not a security mechanism and is different to a security plate. This can be exploited by slipping an object through to open the crash bar.

**Requires:** Bend Rod or other long object

**Targets:** Crash Bar

**Remediation:** 
  - Alarmed exit devices
  - Tamper-resistent crash bars with minimal exposed parts

### Deadbolts
Deadbolts are knobs on a door that can be turned with a thumb or key. They typically require a key to operate from the outside and a thumb turn or key on the inside.

<img width="384" alt="image" src="https://github.com/qwutony/qwutony.github.io/assets/45024645/6b52302b-0978-4108-b27c-0457bf92123b">

**Vulnerability:** Standard deadbolts can be susceptible to lock picking and bumping, allowing unauthorized individuals to unlock them without a key. 

**Requires:**
  - Thumb Turn Flipper
  - Lock Picking Tools
  - Bump Key
  - Drill

<img width="406" alt="image" src="https://github.com/qwutony/qwutony.github.io/assets/45024645/ea90a6e3-bb2c-49a3-a139-41b2f79f4a42">

**Targets:** Deadbolt Mechanism

### Door Edge Gap

**Vulnerability:** REX (Request to Exit) Sensors use passive IR (Infrared) to detect motion. This sensor attempts to detect motion by monitoring for temperature change, such as using canister gas or a stack of paper (sliding underneath).

**Requires:**
  - Stack of paper
  - Gas canister
  - Vapes

**Targets:** REX Sensor

**Remediation:** Certain types of sensors now allow for microwave radar instead of just passive IR.

**Additional Resources:**
  - [White Oak Security - Bypassing Doors Rex Sensor](https://www.whiteoaksecurity.com/blog/bypassing-doors-part-3-rex-sensor/)

### Door Levers - Handles

**Vulnerability:** Modern door levers requires handles for accessibility and ease. We can bypass this by attempting to pull on the inside handle via an under/over the door tool.

**Requires:**
  - Under the door tool
  - Film (over the door)

**Targets:** 
  - Modern door lever
    - Under
    - Over (Sometimes possible if the lever can be tilted upwards to open the door) 

**Remediation:**
  - Dynamic door bottoms
  - Prevent access to the bottom of the door
  - Shroud to prevent access to inside door lever
  - Mounted door handles
  - Clips

<img width="354" alt="image" src="https://github.com/qwutony/qwutony.github.io/assets/45024645/653ffe7c-3f5e-4ac6-af0f-b7416fbf8e61">

## Electronic Bypasses

[TBD]

## Other Covert Entry Mechanisms
Some niche but practical physical penetration testing strategies that are uncategorised but could potentially lead to covert access or escalation.

### Stealing Keys
  - Keys that are left unguarded
  - Lock boxes / Key boxes

### Telephony Access Control Boxes
  - Potentially uses the same key - e.g. A126 Linear Key
  - Open the box to flip the relay - read the manual to bridge relays

<img width="407" alt="image" src="https://github.com/qwutony/qwutony.github.io/assets/45024645/3d34b345-2f21-4d64-ab4f-4158db2eb1ee">

### Keyed-Alike Systems
  - [Common Keys to Purchase](https://gist.github.com/lrvick/a3ff4d7331acfe88ad8f0949439df81c#keys)
  - C415A cabinets, A126 Linear elevators/misc, CH751 gas pumps/RVs/misc, 501CH cabinets/boxes, CC1 golf carts, 1284X Crown Victoria taxi/police, FEO-K1 fire/elevators, CG1 government, 2642 and 1620 fire/elevators.

<img width="408" alt="image" src="https://github.com/qwutony/qwutony.github.io/assets/45024645/c166f313-f9ea-4603-bd4c-30f526c5cdad">

## Lockpicking


## Credits
  - [Tactics of Physical Pen Testers](https://www.youtube.com/watch?v=VJ4FDOw9NcI)
