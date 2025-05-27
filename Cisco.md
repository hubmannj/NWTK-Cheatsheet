# Here you will be able to find Cisco Router and Switch Configurations for Certain Topics

Topics: Etherchanneling, OSPF, VLAN´s, STP and so on.
I am going to show some Configurations splitted in Router and Switch. I will describe how these Configs work and how you are able to connect it with a Router or a Switch.

# Switch
Let´s start with some Configurations for a Cisco Switch. 

## VLAN´s
Vlans are used to segment a Network. You are able to set a border between Clients. For example segmentation between 2 Departments -> Production (VLAN 10) and Marketing (VLAN 20). 

### Create a VLAN

The VLAN-ID is the number for the VLAN. ID 10 means VLAN 10.

```bash
Switch(config)# vlan <VLAN-ID>
```

### Name a VLAN
To create a better overview, name your VLAN´s.

```bash
Switch(config)# clan <VLAN-ID>
Switch(config-vlan)# name <VLAN-Name>
Switch(config-vlan)# exit
```

## Assign a VLAN

### Access Port
You can assign a VLAN to an Access-Port. Access Ports are only used for connection to Clients. So, a Port connection to a Router is no Access-Port.

```bash
Switch(config)# interface <Interface-ID>
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan <VLAN-ID>
```

### Trunk Port
A Trunk is used to communicate over only one Connection with many VLAN´s. Without a Trunk you would need one Connection for each VLAN.

```bash

Switch(config)# interface <Interface-ID>
Switch(config)# switchport trunk encapsulation dot1Q
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan <VLAN-Liste>

```

## Debugging

### VLAN Configuration

```bash

Switch# show vlan brief


```

## Layer 3 Switch
A Layer 3 Switch is a mixture between a Router and a Switch. Basically a Switch with routing functionallity.
