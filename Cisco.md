# Here you will be able to find Cisco Router and Switch Configurations for Certain Topics

Topics: Etherchanneling, OSPF, VLAN´s, STP and so on.
I am going to show some Configurations splitted in Router and Switch. I will describe how these Configs work and how you are able to connect it with a Router or a Switch.

# TABLE OF CONTENT

- [Switch](#switch)
  - [Basic´s](#basics)
    - [Hostname](#hostname)
    - [Interface list](#interface-list)
    - [Interface Ranges](#configure-more-than-one-interface)
    - [Console Logging](#stop-console-logging)
    - [Port Security](#port-security)
  - [VLAN´s](#vlans)
    - [VLAN create](#create-a-vlan)
    - [VLAN name](#name-a-vlan)
    - [VLAN assign](#assign-a-vlan)
      - [Access Port](#access-port)
      - [Trunk Port](#trunk-port)
    - [VLAN Debugging](#vlan-configuration)
  - [Layer 3 Switch](#layer-3-switch)
    - [Routing activate](#activate-routing)
    - [Subinterfaces](#sub-interfaces)
  - [Special VLAN Protocols](#special-vlan-protocols)
    - [VTP (VLAN Trunking Protocol)](#vtp)
      - [Problems with VTP](#problems)
      - [Configuration](#configuration)
        - [Server](#server)
        - [Client](#client)
        - [Transparent](#transparent)
    - [DTP (Dynamic Trunking Protocol)](#dtp)
      - [Problems with DTP](#problems-1)
      - [Configuration](#configuration-1)
  - [Spanning Tree Protocol (STP)](#spanning-tree-protocol)
    - [STP Konfiguration](#configuration-2)
      - [Root Bridge manuell set](#to-set-a-root-bridge-manually)
      - [Change Costs](#change-the-path-costs-for-an-interface)
      - [Portfast](#portfast)
    - [STP Problems and Security](#problems-2)
    - [STP Security Features](#security-configuration)
      - [BPDU-Guard](#bpdu-guard)
      - [Root Guard](#root-guard)
      - [BPDU-Filter](#bpdu-filter)
    - [STP Show-Befehle](#show-commands)
  - [EtherChannel](#etherchannel)
    - [XOR Truth Table](#truth-table)
    - [PAGP](#pagp)
    - [LACP](#lacp)
    - [Load Balancing](#load-balance)
    - [Konfiguration](#configuration-3)
      - [Load Balancing ](#load-balancing)
      - [PAGP Configuration](#pagp-1)
      - [LACP Configuration](#lacp-1)
    - [Show-Commands](#show-commands-1)

# Switch
Let´s start with some Configurations for a Cisco Switch. 

## Basic´s

### Hostname

Switch(config)# 
```bash
hostname <NAME>
```

### Interface list

```bash
show ip interface brief
```

### Configure more than one Interface
To configure interface ge 0/1 and 0/2 at the same time. 

Switch(config)# 
```bash
int range ge<int>-<int>
```

### Stop console logging

Switch(config)# 
```bash
no logging console
```

### Port Security
Port Security is a feature used on switch access ports to restrict input to an interface by limiting and identifying MAC addresses.

Switch(config)# interface <INTERFACE-ID>  
Switch(config-if)#  
```bash
switchport mode access
switchport port-security

```
You are able to block, maximize and shutdown the Port or a MAC.
Maximize
```bash
switchport port-security maximum 2
```
Define a MAC
```bash
switchport port-security mac-address <MAC-ADDRESS>
```
Learn MAC
```bash
switchport port-security mac-address sticky
```

Protect
```bash
switchport port-security violation protect

```
Shutdown Port
```bash
switchport port-security violation shutdown

```
Show Port Security
```bash
show port-security
```
Show learned MAC´s
```bash
show port-security address
```


## VLAN´s
Vlans are used to segment a Network. You are able to set a border between Clients. For example segmentation between 2 Departments -> Production (VLAN 10) and Marketing (VLAN 20). 

### Create a VLAN

The VLAN-ID is the number for the VLAN. ID 10 means VLAN 10.

Switch(config)# 
```bash
vlan <VLAN-ID>
```

### Name a VLAN
To create a better overview, name your VLAN´s.

Switch(config)# 
```bash
vlan <VLAN-ID>
name <VLAN-Name>
exit
```

## Assign a VLAN

### Access Port
You can assign a VLAN to an Access-Port. Access Ports are only used for connection to Clients. So, a Port connection to a Router is no Access-Port.

Switch(config)# 
```bash
interface <Interface-ID>
switchport mode access
switchport access vlan <VLAN-ID>
```

### Trunk Port
A Trunk is used to communicate over only one Connection with many VLAN´s. Without a Trunk you would need one Connection for each VLAN.

Switch(config)# 
```bash

interface <Interface-ID>
switchport trunk encapsulation dot1Q
switchport mode trunk
switchport trunk allowed vlan <VLAN-id>, vlan <VLAN-id>

```

## Debugging

### VLAN Configuration

Switch(config)# 
```bash
show vlan brief
```

# Layer 3 Switch
A Layer 3 Switch is a mixture between a Router and a Switch. Basically a Switch with routing functionallity.
The Configuration of VLAN´s is the same for L3 as for L3.

## Activate Routing

Switch(config)#
```bash
ip routing
```

## Sub Interfaces
Subinterfaces are used to route between VLAN´s. Without routing, you are not able to communicate between VLAN 10 and 20. It´s mostly used for the Router-on-a-Stick methode. But you are also able to configure Sub Interfaces on a L3 Switch. But be aware that you should use a Router for this.

Switch(config)#
```bash
int vlan <id>
ip address <ip> <subnetmask>
no shut
exit
```

# Special VLAN Protocols
## VTP 
VTP (CLAN TRUNKING PROTOCOL) is used to share VLAN´s. It contain a VTP-Server, a VTP-Transparent and a VTP-Client. The VTP domain is created by the Server. In this domain are all Switches which are wanted to receive the VLAN´s. The Border of a domain is a Router. The VTP-Server should always be the Root-Bridge.
### Problems
VTP can only share created VLAN´s. VTP can not assign a VLAN to an Access Port or a Trunking Port. You should also think about Security.
Every VLAN in the Databse of a Server-Switch create one entry. So 3 VLAN´s mean a revison number of 3. The VTP-Server prove the revision number of the Client´s and if the number does not match, he share his VLAN-Database. Hackers can abuse this mechanism by adding one VTP-Server with a higher revision number. After adding, every VLAN in your Network will be overwritten and your Network wont be able to work as before.

### Configuration
#### Server

Switch(config)#
```bash
vtp domain <DOMAIN-Name>
vtp mode server
```

#### Client

Switch(config)#
```bash
vtp domain <DOMAIN-NAME>
vtp mode client
```

#### Transparent

Switch(config)#
```bash
vtp domain <DOMAIN-NAME>
vtp mode transparent
```

## DTP

DTP (DYNAMIC TRUNKING PROTOCOL) is used to create a Trunk or an Access Port between to Switches. There are different Mode´s which cause a different Solution.
The states are listed in the table below.

| Port Mode           | Dynamic Auto       | Dynamic Desirable  | Trunk             | Access             |
|---------------------|--------------------|---------------------|-------------------|--------------------|
| **Dynamic Auto**     | Access             | Trunk               | Trunk             | Access             |
| **Dynamic Desirable**| Trunk              | Trunk               | Trunk             | Access             |
| **Trunk**            | Trunk              | Trunk               | Trunk             | Limited connectivity |
| **Access**           | Access             | Access              | Limited connectivity | Access          |

### Problems
DTP is not best practise. Same as VTP, DTP is Cisco proprietary. You should be aware that DTP may cause problems by Etherchanneling. DTP is also subsceptible for missmatches by missconfiguration. Also VTP-Domain missmatches can cause a Problem.

## Configuration

Switch(config)#
```bash
int <INT>
switchport mode <access>/<trunk>
switchport <depend on the mode you want>
no shut
exit
```

## Spanning Tree Protocol
STP is a protocol to prevent Loops between connected Switches. A Loop is caused by redundant connections between Switches. The main problem is that Frames can not be delieverd to the DST Device.
This can cause:
Broadcast Storms
Not functional Networks

STP use a Root Bridge which is the "Leader" of the Protocol. The Root Bridge will be elected in the electing the Root Bridge Process. At the beginning, every Switch want to be the Root Bridge. The Switch with the lowest Bridge-id (Priority + MAC) will be the Root Bridge in the Network. After electing the Root, every Switch calculate the Path with the lowest Costs to it. After that, the Path with the highest Costs will be blocked which make your Network Loop-free.
There are Different State:
RP (Root Port) is the fastes way to the Root. Only one Port of a Switch is the RP, exect of the Root Bridge.
DP (Designated Port) are all Other Ports of a non blocked Switch.
BP (Blocked Port) is the Port with the highest Cost in the Loop. This state prevent the Network to Loop Frames.

### Configuration

### To set a Root Bridge manually.
Be sure that the priority has to be a multiple of 4096.
So, best case set it to 1.

Switch(config)#
```bash
spanning-tree vlan <VLAN-ID> priority <PRIORITY>
```
You are also able to set the root directly.

Switch(config)#
```bash
spanning-tree vlan <VLAN-ID> root primary (or secondary)
```

### Change the Path Costs for an Interface

Switch(config)#
```bash
int <INT>
spanning-tree cost <PATH COST>
no shut
exit

```
### Portfast
Portfast is used to skip the states from Blocking to Forwarding. This need 50 seconds. With Portfasrt, the Port is instant up in FW. Be sure, that Portfast will only be configured on Access Ports. You need to secure this Port so Hackers cant spam BPDU´s in your Network to make DOS Attacks.

Switch(config)#
```bash
int <INT>
spanning-tree portfast
no shut
exit
```

## Problems
Spanning tree is a very usefull Protocol. Be aware that a Root Bridge in bad hands can cause a DOS of your Network. Hackers can flood BPDU´s in your Network with Tools like Yersinia or Ettercap. So a Kali Linux is able to be the Root Bridge by just connecting to an Access Port. Hackers could no longer converge your Network. So you have to secure your Access Ports and your Root Bridge to prevent a malicious Root Bridge take over.

## Security Configuration

### BPDU-Guard
BPDU-Guard should be configured on all Access Ports. The Guard prevent that Switches or Hackers can flood BPDU´s in your Network to take over the Root or make STP Topologie changes.

Switch(config)#
```bash
int <INT>
switchport mode access
spanning-tree bpduguard enable
no shut
exit
```
You should secure Portfast Access Ports. You can secure every Portfast Port per default with this Command.

Switch(config)#
```bash
spanning-tree portfast bpduguard default

```

### Root-Guard
Root Guard is used for every Uplink Port (Ports to other Switches) to prevent a take over of the Root by having a lower priority. If Root-Guard is configured on all Ports of a Switch, you can add a new Switch in your Network that have a lower priority. The other Switches wont let him become the new Root.

Switch(config)#
```bash
int <INT>
spanning-tree guard root
```

### BPDU-Filter
BPDU-Filter prevent receiving and sending of BPDU´s to a Port. But be aware that this can cause STP to not work normally. If you configure this on RP or not Access Ports, STP will create Loops because the electing the Root Bridge Process and may Block no Ports.

Switch(config)#
```bash
int <INT>
spanning-tree bpdufilter enable
```

Same as before, you can set BPDU-Filter per default to your Portfast Ports.

Switch(config)#
```bash
int <INT>
spanning-tree portfast bpdufilter default
```

## Show Commands


```bash
show spanning-tree /<detail>
```


## Etherchannel
Channelbonding is e method to expand Interfaces. With a Bond, you are able to have one Connection to a Router or a Switch with 2-8 Ports combined. There are 2 different Protocols. PAGP (PORT AGGREGATION PROTOCOL) and LACP (LINK AGGREGATION CONTROL PROTOCOL). The first one is Cisco proprietary and the second one is for multi vendor Networks. You are able to Load Balance the Traffic. So a Connection to a DST went not every time over the same Link. A perfect Load Balance only work with 2,4 and 8 Link Channels. To Balance the Load, a X-OR Alghorytmen is used. 
### Truth Table

| Inputs | Output |
|--------|--------|
| 0 0    |   0    |
| 0 1    |   1    |
| 1 0    |   1    |
| 1 1    |   0    |

If you want to know, which Link will be used if you communicate to a Source, you can calculate it like that.
SRC: 172.16.1.1
DST: 10.10.10.46
Channel: 8 Links (0-7, because you begin count by 0)

At first, you write down the last Octet in binary.
SRC: 0000 0000
DST: 0010 1110

An 8 Link Channel need the last 3 digits to go through the X-OR Operation. 2 Link 1 and 4 Link 2.
000 and 110

1 XOR 0 = 1
0 XOR 1 = 1
0 XOR 1 = 1

Result = 111
111 is 7 if you read it in Binary. So for this Connection, Link 7 (the last) is used.

PAGP and LACP have different modes. You can compare it with DTP where you negotiate a Channel.

### PAGP
Let´s begin with the truth table of PAGP.

| Seite A                | Verbindung     | Seite B        |
|------------------------|----------------|----------------|
| On                    | **Channel**     | On             |
| On/Auto/Desirable     | **No Channel**  | Off            |
| Auto/Desirable        | **Channel**     | Desirable      |
| Auto/On               | **No Channel**  | Auto           |

Best practise is a desirable Configuration on both sides.

### LACP

| Seite A     | Verbindung     | Seite B     |
|-------------|----------------|-------------|
| Active      | **Channel**     | Active      |
| Active      | **Channel**     | Passive     |
| Passive     | **No Channel**  | Passive     |
| On          | **Channel**     | On          |
| On          | **No Channel**  | Passive     |
| On          | **No Channel**  | Active      |


As before, use Active Active to form a Channel.

### Load Balance  

| Load-Balance Option  | Beschreibung               | Hash Operation |
|----------------------|----------------------------|----------------|
| dst-ip               | Ziel-IP-Adresse            | bits           |
| dst-mac              | Ziel-MAC-Adresse           | bits           |
| src-dst-ip           | Quell XOR Ziel IP-Adresse  | XOR            |
| src-dst-mac          | Quell XOR Ziel MAC-Adresse | XOR            |
| src-ip               | Quell-IP-Adresse           | bits           |
| src-mac              | Quell-MAC-Adresse          | bits           |



### Configuration

#### Load Balancing
There are different Options to balance. You can choose betwen src and dst.

Switch(config)# 
```bash
port-channel load-balance <MODE from the Table>
```

The Channel-group number is the number of the channel. You can have 2 Channels (num 1 and num 2) on the same Switch. Both are connected to the same or to different Switches.

#### PAGP

Switch(config)# 
```bash
interface range <INT>-<INT>
channel-protocol pagp
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan <VLAN-ID>
channel-group 1 mode desirable
```

#### LACP

Switch(config)# 
```bash
interface range <INT>-<INT>
channel-protocol lacp
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan <VLAN-ID>
channel-group 1 mode active

```

### Show Commands

```bash
show etherchannel protocol

```

```bash
show etherchannel load-balance

```


```bash
show etherchannel summary

```

# Router

Let´s start with the Router. You will see that the Configuration schema is nearly the same as for the Switch. 

## Basic´s

### Hostname

Router(config)# 
```bash
hostname <NAME>
```

### Interface list

```bash
show ip interface brief
```

### Configure more than one Interface
To configure interface ge 0/1 and 0/2 at the same time. 

Router(config)# 
```bash
int range ge<int>-<int>
```

### Stop console logging

Router(config)# 
```bash
no logging console
```

## VLAN
As discussed before, VLAN´s are used for Layer 2 segmentation. But to route between VLAN´s, a Router is needed. There are 2 different methods to do this. You can use the Router-On-A-Stick method, which is the best practise for inter VLAN routing.

### Router-On-A-Stick
Let´s say, we have a connection between a Switch with VLAN 10 and 20. There is already a Trunk on the Switch to the Router. So, we need to create 2 Sub-Interfaces with the VLAN-ID. The Router will be the Default-Gateway for the VLAN-Clients. If the Interface which will be the Trunk Port is ge 0/0, you create the Sub-Interface ge0/0.10 and ge0/0.20. Now, the router is able to route between the VLAN´s.

Router(config)# 
```bash
int <INT>.<VLAN-ID>
encapsulation dot1Q <VLAN-ID>
ip address <IP of the default-gateway> <SUBNETMASK>
no shut
exit
```

### Seperate Links for every VLAN
For the second method, every VLAN need one Link. This is not the best method because Router mostly have not enough Interfaces. So you waste Resources which are maybe usefull for other Configurations. 
You need to configure the default Gateway on the Interface for the VLAN. Be sure, that ge 0/0 is the gateway of VLAN 10 and on the same Access Port connection as the Switch. If you Connect Access Port VLAN 20 with the Gateway of VLAN 10, the Inter-VLAN routing wont work.

Router(config)# 
```bash
int <INT>
ip address <IP of the default-gateway> <SUBNETMASK>
no shut
exit 
```
