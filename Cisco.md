# Here you will be able to find Cisco router and switch configurations for certain topics

Topics: Etherchanneling, OSPF, VLAN´s, STP, RIP, FHRP and so on.
I am going to show some configurations splitted in Router and Switch. I will describe how these configs work and how you are able to connect it with a Router or a Switch.

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
Let´s start with some configurations for a Cisco switch. 

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
Port-security is a feature used on switch access ports to restrict input to an interface by limiting and identifying MAC-addresses.

Switch(config)# interface <INTERFACE-ID>  
Switch(config-if)#  
```bash
switchport mode access
switchport port-security

```
You are able to block, maximize and shutdown the port or a MAC.
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
Vlans are used to segment a network. You are able to set a border between clients. For example segmentation between 2 departments -> Production (VLAN 10) and Marketing (VLAN 20). 

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
You can assign a VLAN to an Access-Port. Access-Ports are only used for connection to clients. So, a port connection to a router is no Access-Port.

Switch(config)# 
```bash
interface <Interface-ID>
switchport mode access
switchport access vlan <VLAN-ID>
```

### Trunk Port
A Trunk is used to communicate over only one connection with many VLAN´s. Without a Trunk you would need one connection for each VLAN.

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
A layer 3 Switch is a mixture between a router and a switch. Basically a switch with routing functionallity.
The Configuration of VLAN´s is the same for L3 as for L3.

## Activate Routing

Switch(config)#
```bash
ip routing
```

## Sub Interfaces
Subinterfaces are used to route between VLAN´s. Without routing, you are not able to communicate between VLAN 10 and 20. It´s mostly used for the Router-on-a-Stick methode. But you are also able to configure Sub-Interfaces on a L3 switch. But be aware that you should use a router for this.

Switch(config)#
```bash
int vlan <id>
ip address <ip> <subnetmask>
no shut
exit
```

# Special VLAN Protocols
## VTP 
VTP (CLAN TRUNKING PROTOCOL) is used to share VLAN´s. It contain a VTP-Server, a VTP-Transparent and a VTP-Client. The VTP domain is created by the server. In this domain are all switches which are wanted to receive the VLAN´s. The border of a domain is a router. The VTP-Server should always be the Root-Bridge.

### Problems
VTP can only share created VLAN´s. VTP can not assign a VLAN to an Access-Port or a Trunking-Port. You should also think about security.
Every VLAN in the databse of a Server-Switch create one entry. So 3 VLAN´s mean a revison number of 3. The VTP-Server prove the revision number of the client´s and if the number does not match, he share his VLAN-Database. Hackers can abuse this mechanism by adding one VTP-Server with a higher revision number. After adding, every VLAN in your network will be overwritten and your network wont be able to work as before.

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

DTP (DYNAMIC TRUNKING PROTOCOL) is used to create a Trunk- or an Access-Port between to switches. There are different mode´s which cause a different solution.
The states are listed in the table below.

| Port Mode           | Dynamic Auto       | Dynamic Desirable  | Trunk             | Access             |
|---------------------|--------------------|---------------------|-------------------|--------------------|
| **Dynamic Auto**     | Access             | Trunk               | Trunk             | Access             |
| **Dynamic Desirable**| Trunk              | Trunk               | Trunk             | Access             |
| **Trunk**            | Trunk              | Trunk               | Trunk             | Limited connectivity |
| **Access**           | Access             | Access              | Limited connectivity | Access          |

### Problems
DTP is not best practise. Same as VTP, DTP is Cisco proprietary. You should be aware that DTP may cause problems by Etherchanneling. DTP is also subsceptible for missmatches by missconfiguration. Also VTP-Domain missmatches can cause a problem.

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
STP is a protocol to prevent loops between connected switches. A loop is caused by redundant connections between switches. The main problem is that frames can not be delieverd to the DST device.
This can cause:
Broadcast Storms
Not functional networks

STP use a Root-Bridge which is the "Leader" of the protocol. The Root-Bridge will be elected in the electing the Root-Bridge process. At the beginning, every switch want to be the Root- Bridge. The switch writh the lowest Bridge-id (Priority + MAC) will be the Root-Bridge in the network. After electing the root, every Switch calculate the path with the lowest costs to it. After that, the path with the highest costs will be blocked which make your network loop-free.
There are different state`s:
RP (Root Port) is the fastes way to the root. Only one port of a switch is the RP, exect of the Root-Bridge.
DP (Designated Port) are all other ports of a non blocked switch.
BP (Blocked Port) is the port with the highest cost in the loop. This state prevent the network to loop frames.

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
Portfast is used to skip the states from Blocking to Forwarding. This need 50 seconds. With portfast, the port is instant up in FW. Be sure, that portfast will only be configured on Access-Ports. You need to secure this port so hackers cant spam BPDU´s in your network to make DOS-attacks.

Switch(config)#
```bash
int <INT>
spanning-tree portfast
no shut
exit
```

## Problems
Spanning-tree is a very usefull protocol. Be aware that a Root-Bridge in bad hands can cause a DOS of your network. Hackers can flood BPDU´s in your network with tools like Yersinia or Ettercap. So, a Kali Linux is able to be the Root-Bridge by just connecting to an Access-Port. Hackers could no longer converge your network. So, you have to secure your Access-Ports and your Root-Bridge to prevent a malicious Root-Bridge take over.

## Security Configuration

### BPDU-Guard
BPDU-Guard should be configured on all Access-Ports. The Guard prevent that switches or hackers can flood BPDU´s in your network to take over the root or make STP-topologie changes.

Switch(config)#
```bash
int <INT>
switchport mode access
spanning-tree bpduguard enable
no shut
exit
```
You should secure portfast Access-Ports. You can secure every portfast port per default with this command.

Switch(config)#
```bash
spanning-tree portfast bpduguard default

```

### Root-Guard
Root-Guard is used for every Uplink-port (ports to other switches) to prevent a take over of the root by having a lower priority. If Root-Guard is configured on all ports of a switch, you can add a new switch in your network that have a lower priority. The other switches wont let him become the new root.

Switch(config)#
```bash
int <INT>
spanning-tree guard root
```

### BPDU-Filter
BPDU-Filter prevent receiving and sending of BPDU´s to a port. But be aware that this can cause STP to not work normally. If you configure this on RP or not Access-Ports, STP will create loops because the electing the Root-Bridge process and may block no ports.

Switch(config)#
```bash
int <INT>
spanning-tree bpdufilter enable
```

Same as before, you can set BPDU-Filter per default to your portfast ports.

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
Channelbonding is e method to expand interfaces. With a Bond, you are able to have one connection to a router or a switch with 2-8 ports combined. There are 2 different protocols. PAGP (PORT AGGREGATION PROTOCOL) and LACP (LINK AGGREGATION CONTROL PROTOCOL). The first one is Cisco proprietary and the second one is for multi vendor networks. You are able to load balance the traffic. So a connection to a DST went not every time over the same link. A perfect load balance only work with 2,4 and 8 link channels. To balance the load, a X-OR alghorytmen is used. 
### Truth Table

| Inputs | Output |
|--------|--------|
| 0 0    |   0    |
| 0 1    |   1    |
| 1 0    |   1    |
| 1 1    |   0    |

If you want to know, which link will be used if you communicate to a source, you can calculate it like that.
SRC: 172.16.1.1
DST: 10.10.10.46
Channel: 8 Links (0-7, because you begin count by 0)

At first, you write down the last octet in binary.
SRC: 0000 0000
DST: 0010 1110

An 8 link channel need the last 3 digits to go through the X-OR operation. 2 Link 1 and 4 Link 2.
000 and 110

1 XOR 0 = 1
0 XOR 1 = 1
0 XOR 1 = 1

Result = 111
111 is 7 if you read it in binary. So for this connection, link 7 (the last) is used.

PAGP and LACP have different modes. You can compare it with DTP where you negotiate a channel.

### PAGP
Let´s begin with the truth table of PAGP.

| Seite A                | Verbindung     | Seite B        |
|------------------------|----------------|----------------|
| On                    | **Channel**     | On             |
| On/Auto/Desirable     | **No Channel**  | Off            |
| Auto/Desirable        | **Channel**     | Desirable      |
| Auto/On               | **No Channel**  | Auto           |

Best practise is a desirable configuration on both sides.

### LACP

| Seite A     | Verbindung     | Seite B     |
|-------------|----------------|-------------|
| Active      | **Channel**     | Active      |
| Active      | **Channel**     | Passive     |
| Passive     | **No Channel**  | Passive     |
| On          | **Channel**     | On          |
| On          | **No Channel**  | Passive     |
| On          | **No Channel**  | Active      |


As before, use active active to form a channel.

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
There are different options to balance. You can choose betwen src and dst.

Switch(config)# 
```bash
port-channel load-balance <MODE from the Table>
```

The Channel-group number is the number of the channel. You can have 2 channels (num 1 and num 2) on the same switch. Both are connected to the same or to different switches.

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

Let´s start with the router. You will see that the configuration schema is nearly the same as for the switch. 

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

### DHCP-Server
In big networks, IP-Address management is very annoying. Setting static IP-Addresses for every client is very frustrating and leads to conflicts if you forgot which IP is already in use. To reduce the issue, include a DHCP server. Clients send DHCPDISCOVER Packets (Broadcasts) per default to search for any DHCP-Server giving them an IP-Address. If there is any server, they receive a Default-Gateway, DNS-Server and IP-Address which can be rotated or not. At first, exclude all addresses not used in the dhcp pool. If you want address from 100 to 150, exclude 0-99 and 151-255.

````bash
ip dhcp excluded-address <IP> <IP>

ip dhcp pool <NAME>
 network <IP> <SUBNETMASK>
 default-router <ROUTER-IP>
 dns-server <IP>
````

## VLAN
As discussed before, VLAN´s are used for layer 2 segmentation. But to route between VLAN´s, a router is needed. There are 2 different methods to do this. You can use the Router-On-A-Stick method, which is the best practise for inter VLAN routing.

### Router-On-A-Stick
Let´s say, we have a connection between a switch with VLAN 10 and 20. There is already a Trunk on the switch to the router. So, we need to create 2 Sub-Interfaces with the VLAN-ID. The router will be the Default-Gateway for the VLAN-Clients. If the interface which will be the Trunk-Port is ge 0/0, you create the Sub-Interface ge0/0.10 and ge0/0.20. Now, the router is able to route between the VLAN´s.

Router(config)# 
```bash
int <INT>.<VLAN-ID>
encapsulation dot1Q <VLAN-ID>
ip address <IP of the default-gateway> <SUBNETMASK>
no shut
exit
```

### Seperate Links for every VLAN
For the second method, every VLAN need one link. This is not the best method because router mostly have not enough interfaces. So, you waste resources which are maybe usefull for other configurations. 
You need to configure the Default-Gateway on the interface for the VLAN. Be sure, that ge 0/0 is the gateway of VLAN 10 and on the same Access-Port connection as the switch. If you connect Access-Port VLAN 20 with the Gateway of VLAN 10, the Inter-VLAN routing wont work.

Router(config)# 
```bash
int <INT>
ip address <IP of the default-gateway> <SUBNETMASK>
no shut
exit 
```

## FHRP

FHRP (FIRST HOP REDUNDANCY PROTOCOL) is used for a redundand Gateway. FHRP is a group of protocols which are used to have a Gateway for your hosts although the standard Gateway isn´t available. In a normal network is a Host not able to communicate to other networks if the IP of the Gateway is not reachable or not available. FHRP uses virtuell IP-Addresses. A router group (mostly 2 routers) share the same virtuell-IP which is the entry for the host. So, if one router fall out, the host does not regocnize the failure.


### HSRP

The Hot Standby Router Protocol is only for Cisco devices. HSRP uses two routers in a group. One is the active router and the second is the standby. The active router is used to route traffic whereas the standby router only work after a failover.

### Configuration

In HSRP, a numbered group is used. If you work with Router-on-a-stick, use the vlan-id as the group number for the router group on the Sub-Interface. The number must be identical for both routers. Same for the virtuell IP-Address. Interface tracking is a mechanism where the router reduce it´s priority automatically if there is a problem with the outside interface.

```bash
int <int>
standby <group(vlan-id>) ip <virtuell IP>
standby <group(vlan-id>) priority <priority(default 100)>
standby <group(vlan-id>) preempt
standby <group(vlan-id>) track <int> 10
no shut
exit
```

The 10 after track is the priority value which is given to the Router after a failover.


## Dynamic Routing Protocols

Dynamic routing protocols are used to exchange data automatically in networks. Where static routes are configured per hand, protocols like RIP or OSPF are able to adapt network changes. These protocols use different metrics to forward the traffic to the destination.

### Static Route

```bash
ip route <DESTINATION_NETWORK> <SUBNET_MASK> <NEXT_HOP_IP>

```
or 
```bash
ip route <DESTINATION_NETWORK> <SUBNET_MASK> <EXIT_INTERFACE>

```


## RIP 

ROUTING INFORMATION PROTOCOL or RIP is the first protocol to discuss. RIP is a distance-vector protocol. RIP is an old protocol and only used for small networks. To find the perfect path, RIP work with Hopp-Count. This Hopp-Count go from 0-15. So, you are able to have a network with 15 routers. The 16 hopp is for RIP the same as infinite. The protocol work with the Bellman-Ford-Algorythem and send packeges every 30 seconds. There are 2 versions, v1 and v2. No matter what happen, use v2.

### Split-Horizon

Split-Horizon is activated per default on Cisco devices. YOu don´t need to configure it. Split-Horizon means that a router do not send a route back to an interface which he learned from this interface. 

### Route Poisining

It´s configured per default. If a route isn´t available because it´s not valid anymore, the router set the metric to 16 which means infinite. YOu can debug it with the following command.

```bash
debug ip rip
```

If a route is poisened, it should look like this.
```bash
RIP: sending v2 update to 192.168.1.2 via Serial0/0
  suppressed 10.0.0.0/8 metric 16

```

### Administrative Distance (AD)

The default AD in RIP is 120. A lower AD is more appealing. You can change the AD if you want your traffic to go through a certain router.
For this example you set the AD to 95 for the router 192.168.1.1. 
```bash
router rip
 distance 95 192.168.1.1 0.0.0.0

```

### Configuration

Passive interfaces are used for interfaces where no router is directly connected. For example if a switch is connected on ether 1, configure ether 1 as a passive interface. So, ether 1 do not send RIP updates but receive them.

```bash
router rip
version 2
network <IP>
network <IP>
passive-interface <int>
no auto-summary

```

## OSPF

OPEN SHORTEST PATH FIRST or OSPF is another dynamic routing protocol. It´s the better version of RIP. The main fanctionalitys are the same, but OSPF has some efficient extras. It´s a link-state protocol and works with the Dijkstra alghorythm that will be described below. The metric work with costs and not with Hopp-Count. SO, the router find the best and fastest way. You are able to authenticate with MD5 or SHA. OSPF work in Single-Area and Multi-Area which give you the ability to increase your network.


### Dijkstra-Alghorythm

The algorithm calculate the shortest path with the lowest path-costs to the DST. This algorithm is the heart of OSPF. To understand Djikstra, a basic knowledge about graph theory is needed. Every node knew all connection in the whole network. The alghoritm calculate the shortest path with the local router as root. Here is a short example for calculation. suppose this is the graph:

<img width="763" height="478" alt="image" src="https://github.com/user-attachments/assets/0bc5a037-459e-47e1-9a78-26d62280480e" />



### LSA´s

OSPF send different types of LSA (Link State Advertisments). In a Single-Area, only type 1 and 2 LSA´s are sent. Type 3 LSA´s are summarized type 1 LSA´s that are generated by an Area-Border-router in a Multi-Area network. They are sent to all routers in other areas. Type 4 and 5 LSA´s are generated by an ABR with a ASBR (Autonomous System Boundary Router) in it´s area. Type 4 shows the route to the ASBR while Type 5 is generetad by the ASBR itsself to show a route to external resources. 

Here is a Picture from the routers and areas.

![image](https://github.com/user-attachments/assets/f67c9ca9-1fc8-48fe-8401-db2efda1f7f4)


### LSDB

The Link State Database is the heart of OSPF. In RIP, a router only know it´s direct connected neighbours. A router working with OSPF knows the whole network topologie. Every information is stored in this database. To reduce the size and the amount of resources, work with Multi-Area OSPF. A Single Area OSPF LSDB has the whole information stored which may leads to performance problems. In Multi-Area, summerized information will be stored.

### Configuration

OSPF works nearly same as RIP. For best practise, create an interface loopback with an ID which will be the router ID for easier identification. For example Router 1 receive the ID 1.1.1.1.

````bash
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
````
After creating a loopback interface (e.g., Loopback0), configure OSPF with a process ID such as 1.
As a best practice, set all interfaces to passive by default using the passive-interface default command. This reduces unnecessary OSPF Hello traffic. Then, explicitly enable OSPF on the required interfaces using no passive-interface <interface>.
Make sure to include all directly connected networks that should participate in OSPF using appropriate network statements.
On Cisco devices, OSPF area IDs must be numerical (e.g., area 0, area 1, etc.). Unlike MikroTik, Cisco does not support named areas. On MikroTik, you can assign descriptive area names, such as area=backbone, for better readability.
Use area 0 for the backbone area and incrementing numbers for additional areas. For consistency and clarity, assign loopback IP addresses like 1.1.1.1, 1.1.1.2, etc., within the same area. Otherwise you may be forced to give one router the id 17.17.17.17.

````bash
router ospf 1
 router-id 1.1.1.1
 passive-interface default         
 no passive-interface <INT>
 no passive-interface <INT> 
 network 1.1.1.1 0.0.0.0 area 0       
 network <NETWORK IP> <WILDCARD> area <ID>
 network <NETWORK IP> <WILDCARD> area <ID>
````
#### Virtual-link

A virtual link is a virtual tunnel through a transit area. Sometimes, an area is not directly connected to the backbone area (Area 0).
In this case, this area – for example Area 2 – needs a virtual connection through Area 1 into Area 0.
Otherwise, clients in Area 2 cannot reach clients in Area 0. To create a virtual link, two routers are needed:

The ABR between Area 0 and Area 1 and the ABR between Area 1 and Area 2.

The virtual link connects these two routers logically through Area 1.
One important requirement is the router ID of both routers, which must be included in the command. A virtual link is only possible through a transit area. A stub area cannot be used as a transit area.

Transit area: An area that connects other areas and allows OSPF traffic to pass through (e.g., Area 1 in this example).

Stub area: A dead-end area that does not forward external or inter-area routes, and typically only receives a default route

````bash
router ospf 1
area <transit-area> virtual-link <Router-ID>
````

If the router with the router-id 1.1.1.1 is the ABR of area 0-1 and the router with the router-id 2.2.2.2 is the ABR of the area 1-2, the link would look like this.

ABR area 0-1:

````bash
 area 1 virtual-link 2.2.2.2
````

ABR area 1-2:

````bash
 area 1 virtual-link 1.1.1.1
````

#### Default route

If a router has no information (no route entry in its routing table) about a destination network, it does not know what to do with the packet. In such cases, the router may respond with an error message like "Destination Host Unreachable".
To prevent this issue, you can use default routes. In multi-area OSPF environments, clients might not be able to reach the internet because they don't know how to forward packets to external networks. To solve this, configure a default route on the ASBR (Autonomous System Boundary Router) and redistribute it into OSPF.
This ensures that all routers in the OSPF domain learn a default route and can forward traffic for unknown destinations to the ASBR.

````bash
ip route 0.0.0.0 0.0.0.0 <OUTGOING-INTERFACE>
router ospf 1
 default-information originate

````

