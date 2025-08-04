# IP-Addresses
IP addresses act like names for devices in a network. Within the same subnet, each address must be unique. Otherwise, communication and routing may fail due to address conflicts.

## Interface Conflicts on Routers
A router can only have one interface in a given subnet. For example, if interface eth0 has the IP address 192.168.12.1 and eth1 has 192.168.12.2, and both are in the same subnet (/24), this setup will break ARP logic and confuse the routing process.
The router won't be able to determine which interface should handle traffic destined for 192.168.12.0/24, leading to unpredictable behavior and routing failure. So, use every time only one interface in the subnet. To reduce address spaces, split networks in /25. First interface .1 and second interface .129.

## Private vs Public IP-Addresses

There are 3 different spaces for private networks. All other IP`s are public and unique.

| SPACE    | TYPE     |  USECASE |
|----------|----------|----------|
| Wert 1   | Wert 2   | Wert 3   |
| Wert 4   | Wert 5   | Wert 6   |

| Spalte 1 | Spalte 2 | Spalte 3 |
|----------|----------|----------|
| Wert 1   | Wert 2   | Wert 3   |
| Wert 4   | Wert 5   | Wert 6   |
