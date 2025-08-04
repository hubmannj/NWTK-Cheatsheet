# IP-Addresses
IP addresses act like names for devices in a network. Within the same subnet, each address must be unique. Otherwise, communication and routing may fail due to address conflicts.

## Interface Conflicts on Routers
A router can only have one interface in a given subnet. For example, if interface eth0 has the IP address 192.168.12.1 and eth1 has 192.168.12.2, and both are in the same subnet (/24), this setup will break ARP logic and confuse the routing process.
The router won't be able to determine which interface should handle traffic destined for 192.168.12.0/24, leading to unpredictable behavior and routing failure. So, use every time only one interface in the subnet. To reduce address spaces, split networks in /25. First interface .1 and second interface .129.

## Private vs Public IP-Addresses

There are 3 different spaces for private networks. All other IP`s are public and unique.

| SPACE    | TYPE     |  USECASE |
|----------|----------|----------|
| `10.0.0.0-10.255.255.255`  | private  | big companies (8 bit)  |
| `172.16.0.0-172.31.255.255` | private   | medium companies (12 bit) |
| `192.168.0.0-192.168.255.255 `| private | small networks (16 bit) |
| all other | public | internet |

Private IP-Addresses are not allowed to be used in the internet. NAT (Network Address Translation) change private IP´s to public IP`s. When a client want to ping the DNS of Google, the router requests with the public IP given from the ISP.

## Design
A short phrase to remember more easily. If you need halft the hosts, reduce the subnet. A /24 subnet has 254 hosts (0 Network and 255 Broadcast) and a /25 subnet has 126 hosts. So, the formula is 2ⁿ - 2 while n is 32-subnetmask. 

### Example
For a /24 subnet:

n = 32 – 24 = 8

2⁸ = 256

256 – 2 = 254 usable hosts.

| Subnet Size | Subnet Mask       | Usable Hosts | Suitable For                                    |
| ----------- | ----------------- | ------------ | ----------------------------------------------- |
| /30         | `255.255.255.252` | 2            | Point-to-point links (e.g., router-to-router)   |
| /29         | `255.255.255.248` | 6            | Small groups, management, firewalls             |
| /28         | `255.255.255.240` | 14           | VLANs, small departments                        |
| /27         | `255.255.255.224` | 30           | Small offices                                   |
| /26         | `255.255.255.192` | 62           | Medium-sized groups                             |
| /25         | `255.255.255.128` | 126          | Larger VLANs                                    |
| /24         | `255.255.255.0`   | 254          | Standard LANs                                   |
| /16         | `255.255.0.0`     | 65,534       | Very large networks (Warning: often not ideal!) |

## Best Practise

### Structure

Plan your networks with a suitable subnet. If  there are 100 hosts in the network, use a /25 subnetmask. Always plan some addresses for infrastructure. Use .1 -.10 for gateways, printers or servers to make it easier to understand.

### DHCP
For big networks, use always a DHCP-Server. DHCP reduce the risk of address-conflicts. 
DHCP offers a default gateway, DNS-Server and a pool range for clients. Every client address will be part of the configured range.
