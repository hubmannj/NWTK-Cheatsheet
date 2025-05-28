# Here you will be able to find Cisco Router and Switch Configurations for Certain Topics

Topics: Etherchanneling, OSPF, VLAN´s, STP and so on.
I am going to show some Configurations splitted in Router and Switch. I will describe how these Configs work and how you are able to connect it with a Router or a Switch.

# TABLE OF CONTENT
- [Basic´s](#basics)
- [VLAN´s](#vlans)
- [Special VLAN Protocols](#special-vlan-protocols)

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
