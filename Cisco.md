# Here you will be able to find Cisco Router and Switch Configurations for Certain Topics

Topics: Etherchanneling, OSPF, VLAN´s, STP and so on.
I am going to show some Configurations splitted in Router and Switch. I will describe how these Configs work and how you are able to connect it with a Router or a Switch.

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

