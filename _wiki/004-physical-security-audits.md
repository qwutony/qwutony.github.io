---
layout: wiki
title: Physical Security Audits
cate1: Physical
cate2: 
description: Physical Security Audits
keywords: Physical
---

# Physical Security Auditing
Covert entry strategies to allow attackers to bypass physical security mechanisms to gain unauthorized entry to a building. Resources retrieved from various talks - credit provided below.

## Auditing Checklist
  - Hinge Inspection
  - Latch Loiding (Door Latch)
  - Crash Bars
  - Deadbolts
  - Door Edge Gap - Rex Sensors
  - Door Levers (Handles)
  - Over the Door Attacks
  - Under the Door Attacks
  - Triggering Sensors and Exit Mechanisms
  - Stealing Keys
  - Telephony Access Control Boxes
  - Keyed-Alike Systems

## To Bring Checklist
  - Traveller Hook
  - Air Wedge Bag
  - Metal Coathanger (for REX buttons)
  - Rope (UTD/OTD)
  - Measuring Tape
  - A4 Laminated Paper

-------------------------------------------------------------------------------------

# Physical Security Auditing Theory

## Pre-Engagement Requirements
  - Floor Plans - Obtain detailed and updated floor plans of the facility including floors, rooms and access points.
  - Policies and Procedures - Security policies and procedures, emergency response and evacuation plans.
  - Security Systems Documentation - List of access control systems, CCTV, alarms, sensors etc. May require brand and type if necessary.
  - Permission and Authorization - And a get out of jail free card!
  - Scopes/Objectives/Rules of Engagement
  - Scheduling and Timing (if necessary)

## Reconnaissance and Observation
During the beginning of the engagement, become hyperaware of the surrounding in and out of the target building. Document the information in the following format:
  - Photographs and descriptions
  - Detailed description of use, security features and potential vulnerabilities
  - Observe usage patterns and time of day

### Entry and Exit Points
*Potentially include photography of all the interesting locations and document each point of entry and exit*
  -  Main Entrances and Exits
  -  Side or Secondary Entrances and Exits *(for employees, staff etc)*
  -  Emergency Exits *(usually with crash bars)*
  -  Fire Escapes
  -  Accessible Windows *(Ground floor or other accessible windows)*
  -  Roof Access Points *(Ladders, latches, doors that provide access to the roof)*
  -  Loading Docks
  -  Garage Doors
  -  Large HVAC Vents/Ducts
  -  Service Entrances and Exits
  -  Basement Entrances and Exits
  -  Gates and Fences

### CCTV security cameras
*Security cameras that monitor key access points in and out of a building with various functionality. This serves as an incomplete list of all the types of CCTV cameras to observe.*
  - Dome Cameras *(Unable to determine the direction of the camera and angle)*

<img width="240" alt="image" src="https://github.com/qwutony/qwutony.github.io/assets/45024645/237630e2-bc9c-413e-87ed-f61c0edc58c5">

  - Bullet Cameras *(Long distance, good range and more capable outdoors)*

<img width="180" alt="image" src="https://github.com/qwutony/qwutony.github.io/assets/45024645/d7c37997-7ec6-4ed6-a303-24cf8b3f8098">

  - Pan-Tilt-Zoom (PTV) CCTV Cameras

<img width="166" alt="image" src="https://github.com/qwutony/qwutony.github.io/assets/45024645/2fe31839-22a6-44c0-88db-101d79f3fb8c">

  - Wireless Cameras

<img width="121" alt="image" src="https://github.com/qwutony/qwutony.github.io/assets/45024645/421bb725-4063-4257-ba94-630e2194083e">

  - Day and Night Security Cameras *(Works well in low light, record in colour and B/W, used for outdoor surveillance)*

<img width="273" alt="image" src="https://github.com/qwutony/qwutony.github.io/assets/45024645/92384109-b3ea-493e-89c4-2ecd9b994d64">

### Other Security Devices
*Also document the location of other security devices.*
  - Key Card Access Systems
  - Biometric Access Systems
  - Turnstiles and Security Gates
  - Intrusion Detection Systems
    - Motion Sensors
    - Glass Break Sensors
    - Door and Window Contacts
    - Pressure Mats
  - Intrusion Alarm Systems
  - Barriers and Physical Obstructions
  - Serveillance - *refer to CCTV cameras above*
  - Exposed Ethernet Ports
  - Accessible Office Technology
  - Printers
  - Light Controllers
  - Magnetic Sensors
  - Fire and Hazard Locations

### Other Observations
  - Employee behaviour
  - Vulnerability / Blind spots and low traffic areas
  - Key Locations
  - Third Party Contractors (cleaners, security companies etc.)

### People Watching
  - Badge Behaviour
  - Interaction Points
  - Reaction towards foreign objects such as Key Loggers, MITM devices etc.

---------------------------------------------------------------------------------------

# Physical Security Attack Flow

## Initial Access
  - Tailgating if entrance gates take too long to close
  - Obtain visitor or temporary staff access via reception
  - Unattended access points (backdoors, vents etc.) - refer to "Entry and Exit Points"

## Privilege Escalation (Post Exploitation)
  - Hiding in the toilet / elevators
  - Finding credentials near devices / workstations
  - Stealing devices and objects from employees
  - Door security
  - Access to sensitive locations
    - Security lifts
    - Server rooms
    - Storage rooms
    - Network ports and ethernet ports
    - HVAC, electrical or engineering systems  

---------------------------------------------------------------------------------------

## Door Security
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

### Door Latch (Latch Loiding)
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

### Door Edge Gap - Rex Sensors

**Vulnerability:** REX (Request to Exit) Sensors use passive IR (Infrared) to detect motion. This sensor attempts to detect motion by monitoring for temperature change, such as using canister gas or a stack of paper (sliding underneath).

**Requires:**
  - Stack of paper
  - Gas canister
  - Vapes

**Targets:** REX Sensor

**Remediation:** Certain types of sensors now allow for microwave radar instead of just passive IR. Security Astrogal for certain edges of the door.

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

### Door Frame
  - Jacking and spreading attacks
  - Can use air wedge to determine if latch attacks are effective

### Reaching Door Release
  - Exit buttons
  - REX (Request to Exit) systems - can interact with it from outside of the door

### Magnetic Lock Systems
  - Crash Bars
  - Junction Boxes - can take these apart to release the door
  - Throwing A4 laminated paper to trick infrared systems
  - Measuring tape to trick infrared systems

<img width="532" alt="image" src="https://github.com/qwutony/qwutony.github.io/assets/45024645/064e6065-38f5-413a-9bba-88636d7b4c35">

### Key Lock Box
  - Some combination key lock boxes only allow for unique numbers in each combination, which reduces a 4 digit lock to ~250 combinations
  - Observe fingerprints to determine lock combinations
  - Inserting a lock pick into a combination lock and rotating the last digit until compromise
  - Bypassing wafer tumbler locks to access higher security keys

**Additional Resources**
  - [Hack a Key Lock Box](https://bugadvisor.com/2019/08/24/its-easy-to-hack-a-key-lock-box-theyre-not-as-secure-as-you-might-think/)
  - [Bypassing Common Key Boxes](https://www.youtube.com/watch?v=-o_l69ZoKgs)

## Electronic Bypasses

### Cloning Electronic Credentials
  - RFID Bypasses

## Locks

### Padlocks
  - Padlock Shimming
  - Dual Latch Padlocks
  - Solution: Double-ball mechanism

### Lockpicking
  - Warded Locks

<img width="521" alt="image" src="https://github.com/qwutony/qwutony.github.io/assets/45024645/7038bdb8-c44a-4530-8a35-3e286fa01820">

  - Overlifting

<img width="510" alt="image" src="https://github.com/qwutony/qwutony.github.io/assets/45024645/19fc6bbb-5521-432f-9665-0018d5552be1">

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

## Credits
  - [Tactics of Physical Pen Testers](https://www.youtube.com/watch?v=VJ4FDOw9NcI)
  - [Covert Access Team](https://covertaccessteam.substack.com/p/getting-started-with-physical-penetration)

## Other Resources
  - [RTCG Training](https://www.redteamalliance.com/RTCG.html)
