```markdown
# MikroTik RouterOS – Konfigurationsguide

> RouterOS v7 | Getestet in PNetLab

---

## Inhaltsverzeichnis

1. [IP-Adressen verwalten](#1-ip-adressen-verwalten)
2. [DHCP Server konfigurieren](#2-dhcp-server-konfigurieren)
3. [NAT / Masquerade](#3-nat--masquerade)
4. [OSPF konfigurieren](#4-ospf-konfigurieren)
5. [Config speichern & exportieren](#5-config-speichern--exportieren)

---

## 1. IP-Adressen verwalten

### IP-Adresse setzen

```routeros
ip address add address=10.10.10.1/30 interface=ether2
ip address add address=10.10.10.5/30 interface=ether3
```

### Loopback Interface erstellen und IP setzen

```routeros
interface bridge add name=loopback1
ip address add address=3.3.3.3/32 interface=loopback1
```

> Loopbacks werden bei MikroTik als Bridge Interface erstellt, nicht als eigener Loopback-Typ.

### IP-Adressen anzeigen

```routeros
ip address print
```

### IP-Adresse entfernen

```routeros
# Methode 1 – nach Interface suchen und entfernen
ip address remove [find interface=ether2]

# Methode 2 – nach Adresse suchen und entfernen
ip address remove [find address="10.10.10.1/30"]

# Methode 3 – per Nummer (aus ip address print)
ip address remove 0
```

> Der `[find ...]` Befehl sucht den Eintrag dynamisch – du musst die ID nicht kennen.

---

## 2. DHCP Server konfigurieren

### Schnell-Setup mit dem Setup-Wizard (empfohlen)

```routeros
ip dhcp-server setup
```

Der Wizard fragt dich Schritt für Schritt:

```
Select interface to run DHCP server on: ether2
Select network for DHCP addresses: 192.168.1.0/24
Select gateway for given network: 192.168.1.1
Select pool of ip addresses given out by DHCP server: 192.168.1.2-192.168.1.254
Select DNS servers: 8.8.8.8
Select lease time: 1d
```

> Nach dem Wizard ist der DHCP Server sofort aktiv!

### Manuell konfigurieren

```routeros
# IP Pool definieren
ip pool add name=pool-lan ranges=192.168.1.50-192.168.1.200

# DHCP Server erstellen
ip dhcp-server add name=dhcp-lan interface=ether2 address-pool=pool-lan disabled=no

# Netzwerk-Infos (Gateway, DNS) hinterlegen
ip dhcp-server network add address=192.168.1.0/24 gateway=192.168.1.1 dns-server=8.8.8.8
```

### DHCP Status prüfen

```routeros
ip dhcp-server print          # Server anzeigen
ip dhcp-server lease print    # Aktive Leases anzeigen
```

### DHCP Client (für WAN-Interface)

```routeros
ip dhcp-client add interface=ether1 disabled=no
ip dhcp-client print
```

---

## 3. NAT / Masquerade

### Masquerade (dynamische Public-IP)

Wird verwendet wenn die WAN-IP per DHCP vergeben wird (z.B. in PNetLab über Cloud/NAT).

```routeros
ip firewall nat add chain=srcnat action=masquerade out-interface=ether1
```

> `ether1` = dein **WAN-Interface** (nach außen zum Internet)

### Src-NAT (feste Public-IP)

```routeros
ip firewall nat add chain=srcnat action=src-nat \
  to-addresses=203.0.113.1 out-interface=ether1
```

### NAT-Regeln anzeigen

```routeros
ip firewall nat print
```

### NAT-Regel entfernen

```routeros
ip firewall nat remove [find action=masquerade]
# oder per Nummer:
ip firewall nat remove 0
```

---

## 4. OSPF konfigurieren

### Schritt 1 – OSPF Instanz erstellen

```routeros
routing ospf instance add name=OSPF1 router-id=3.3.3.3 disabled=no
```

> Die `router-id` sollte eindeutig sein – am besten die Loopback-Adresse verwenden.

### Schritt 2 – Area erstellen

```routeros
routing ospf area add name=Backbone area-id=0.0.0.0 instance=OSPF1 disabled=no
```

### Schritt 3 – Netzwerke/Interfaces hinzufügen

```routeros
routing ospf interface-template add area=Backbone networks=10.10.10.0/30 disabled=no
routing ospf interface-template add area=Backbone networks=10.10.10.4/30 disabled=no
routing ospf interface-template add area=Backbone networks=3.3.3.3/32 disabled=no
```

> Hier gibst du die **Netzwerke** an, nicht die Interface-Namen (anders als bei Cisco).

### Schritt 4 – Default Route ins OSPF injecten

Damit alle anderen Router (z.B. Cisco) eine Default Route vom MikroTik lernen:

```routeros
routing ospf instance set OSPF1 originate-default=always
```

> `originate-default=always` = Default Route wird **immer** advertised, auch wenn keine Default Route in der Routing-Tabelle vorhanden ist.
>
> `originate-default=if-installed` = Default Route wird nur advertised wenn eine Default Route existiert (z.B. via DHCP-Client auf ether1).

### OSPF überprüfen

```routeros
routing ospf neighbor print      # OSPF Nachbarn anzeigen
routing ospf route print         # OSPF Routen anzeigen
ip route print                   # Alle Routen (inkl. OSPF)
```

### Beispiel – Fertige OSPF Config (wie in PNetLab)

```routeros
# Loopback
interface bridge add name=loopback1
ip address add address=3.3.3.3/32 interface=loopback1

# IPs auf Interfaces
ip address add address=10.10.10.1/30 interface=ether2
ip address add address=10.10.10.5/30 interface=ether3

# OSPF
routing ospf instance add name=OSPF1 router-id=3.3.3.3 originate-default=always disabled=no
routing ospf area add name=Backbone area-id=0.0.0.0 instance=OSPF1 disabled=no
routing ospf interface-template add area=Backbone networks=3.3.3.3/32 disabled=no
routing ospf interface-template add area=Backbone networks=10.10.10.0/30 disabled=no
routing ospf interface-template add area=Backbone networks=10.10.10.4/30 disabled=no
```

---

## 5. Config speichern & exportieren

### Export (wie `copy run start` bei Cisco)

```routeros
export file=startup
```

Speichert die Config als lesbare `.rsc` Textdatei.

### Config anzeigen (ohne speichern)

```routeros
export
```

### Config wiederherstellen

```routeros
import file=startup.rsc
```

### Sauber herunterfahren in PNetLab

```routeros
system shutdown
```

> ⚠️ Wichtig: In PNetLab den Node **immer mit `system shutdown` beenden**, nicht einfach stoppen! Sonst gehen ungespeicherte Änderungen verloren.

---

## Nützliche allgemeine Befehle

```routeros
# System Info anzeigen (RouterOS Version etc.)
system resource print

# Routen anzeigen
ip route print

# Interfaces anzeigen
interface print

# ARP-Tabelle
ip arp print

# DNS konfigurieren
ip dns set servers=8.8.8.8

# Ping
ping 8.8.8.8

# Traceroute
tool traceroute 8.8.8.8
```

---

> **Hinweis:** Diese Anleitung gilt für **RouterOS v7**. In v6 ist die OSPF-Syntax anders (`routing ospf network add ...` statt `interface-template`).
```

# MikroTik RouterOS – WireGuard VPN über OSPF

> RouterOS v7 | Getestet in PNetLab

---

## Topologie

```
<img width="1482" height="719" alt="image" src="https://github.com/user-attachments/assets/28adec16-e71d-4aec-bbd9-ddf072aa0ae2" />

```

| Router | Loopback | LAN Interface | LAN Netz | WAN Interface | WAN IP |
|--------|----------|---------------|----------|---------------|--------|
| R1 | `1.1.1.1/32` | ether2 | 192.168.10.0/24 (.1) | ether3 | 10.10.10.1/30 |
| R2 | `2.2.2.2/32` | – | – | ether3 / ether2 | .2 / .5 |
| R3 | `3.3.3.3/32` | ether3 | 192.168.20.0/24 (.1) | ether1 | 10.10.10.6/30 |

**WireGuard-Tunnel:** `10.0.0.1/30` (R1) ↔ `10.0.0.2/30` (R3)

---

## Ziel

Ein verschlüsselter WireGuard-Tunnel zwischen R1 und R3, sodass:
- `192.168.10.0/24` (links) ↔ `192.168.20.0/24` (rechts) kommunizieren kann
- R2 nur als "Internet"-Simulation via OSPF dient
- Der Tunnel die Loopback-Adressen (`1.1.1.1`, `3.3.3.3`) als Endpunkte nutzt

---

## Schritt 1 – WireGuard Interface erstellen & Keys generieren

Auf **R1 und R3** je einmal ausführen:

```routeros
/interface wireguard add name=wg1 listen-port=51820
/interface wireguard print
```

> RouterOS generiert automatisch ein **Private/Public Key Paar**.  
> Den `public-key` von jedem Router notieren – er wird beim Peer des anderen Routers eingetragen.

**Beispiel-Output:**
```
Flags: X - disabled; R - running
 0  R name="wg1" mtu=1420 listen-port=51820
      private-key="<PRIVAT - NICHT TEILEN>"
      public-key="vZtkTWRd5ySO3/PEqlZSb7fhSar4zBXGPaWvK/fveBs="
```

---

## Schritt 2 – IP-Adressen auf dem Tunnel

```routeros
# R1
/ip address add address=10.0.0.1/30 interface=wg1

# R3
/ip address add address=10.0.0.2/30 interface=wg1
```

> Die Tunnel-IP ist eine **eigene virtuelle Adresse** auf dem `wg1`-Interface.  
> Sie hat nichts mit der WAN- oder LAN-IP zu tun.

---

## Schritt 3 – WireGuard Peers konfigurieren

### R1 – Peer eintragen (zeigt auf R3)

```routeros
/interface wireguard peers add \
    interface=wg1 \
    public-key="<PUBLIC-KEY-VON-R3>" \
    endpoint-address=3.3.3.3 \
    endpoint-port=51820 \
    allowed-address=10.0.0.2/32,192.168.20.0/24 \
    persistent-keepalive=25s
```

### R3 – Peer eintragen (zeigt auf R1)

```routeros
/interface wireguard peers add \
    interface=wg1 \
    public-key="<PUBLIC-KEY-VON-R1>" \
    endpoint-address=1.1.1.1 \
    endpoint-port=51820 \
    allowed-address=10.0.0.1/32,192.168.10.0/24 \
    persistent-keepalive=25s
```

**Erklärung der Parameter:**

| Parameter | Bedeutung |
|-----------|-----------|
| `public-key` | Öffentlicher Schlüssel des **Gegenstücks** |
| `endpoint-address` | **WAN/Loopback-IP** des Gegenstücks (via OSPF erreichbar) |
| `endpoint-port` | UDP-Port auf dem der Gegenstück lauscht |
| `allowed-address` | Welche IPs **durch den Tunnel** dürfen (Tunnel-IP + LAN) |
| `persistent-keepalive` | Hält den Tunnel aktiv (wichtig hinter NAT) |

---

## Schritt 4 – Routen setzen

```routeros
# R1 – Route zum LAN von R3 über den Tunnel
/ip route add dst-address=192.168.20.0/24 gateway=wg1

# R3 – Route zum LAN von R1 über den Tunnel
/ip route add dst-address=192.168.10.0/24 gateway=wg1
```

---

## Schritt 5 – Firewall konfigurieren

### R1

```routeros
# WireGuard UDP-Port erlauben (eingehend von R3)
/ip firewall filter add \
    action=accept chain=input \
    dst-port=51820 protocol=udp \
    src-address=3.3.3.3

# Forward zwischen den LANs erlauben
/ip firewall filter add action=accept chain=forward \
    dst-address=192.168.20.0/24 src-address=192.168.10.0/24
/ip firewall filter add action=accept chain=forward \
    dst-address=192.168.10.0/24 src-address=192.168.20.0/24
```

### R3

```routeros
# WireGuard UDP-Port erlauben (eingehend von R1)
/ip firewall filter add \
    action=accept chain=input \
    dst-port=51820 protocol=udp \
    src-address=1.1.1.1

# Forward zwischen den LANs erlauben
/ip firewall filter add action=accept chain=forward \
    dst-address=192.168.10.0/24 src-address=192.168.20.0/24
/ip firewall filter add action=accept chain=forward \
    dst-address=192.168.20.0/24 src-address=192.168.10.0/24
```

> ⚠️ **Reihenfolge beachten:** Regeln müssen **vor** einer Default-Drop-Regel stehen.  
> Prüfen mit: `/ip firewall filter print`

---

## Vollständige Konfiguration (Export)

### R1 – Kompletter Export

```routeros
/interface bridge
add name=Loopback1
/interface wireguard
add listen-port=51820 mtu=1420 name=wg1
/routing ospf instance
add disabled=no name=OSPF1 router-id=1.1.1.1
/routing ospf area
add disabled=no instance=OSPF1 name=Internet
/interface wireguard peers
add allowed-address=10.0.0.2/32,192.168.20.0/24 \
    endpoint-address=3.3.3.3 endpoint-port=51820 \
    interface=wg1 persistent-keepalive=25s \
    public-key="<PUBLIC-KEY-VON-R3>"
/ip address
add address=1.1.1.1 interface=Loopback1 network=1.1.1.1
add address=10.10.10.1/30 interface=ether3 network=10.10.10.0
add address=192.168.10.1/24 interface=ether2 network=192.168.10.0
add address=10.0.0.1/30 interface=wg1 network=10.0.0.0
/ip dhcp-client
add interface=ether1
/ip firewall filter
add action=accept chain=input dst-port=51820 protocol=udp src-address=3.3.3.3
add action=accept chain=forward dst-address=192.168.20.0/24 src-address=192.168.10.0/24
add action=accept chain=forward dst-address=192.168.10.0/24 src-address=192.168.20.0/24
/ip route
add dst-address=192.168.20.0/24 gateway=wg1
/routing ospf interface-template
add area=Internet disabled=no networks=10.10.10.0/30
add area=Internet disabled=no networks=1.1.1.1/32
```

### R3 – Kompletter Export

```routeros
/interface bridge
add name=Loopback1
/interface wireguard
add listen-port=51820 mtu=1420 name=wg1
/routing ospf instance
add disabled=no name=OSPF1 router-id=3.3.3.3
/routing ospf area
add disabled=no instance=OSPF1 name=Internet
/interface wireguard peers
add allowed-address=10.0.0.1/32,192.168.10.0/24 \
    endpoint-address=1.1.1.1 endpoint-port=51820 \
    interface=wg1 persistent-keepalive=25s \
    public-key="<PUBLIC-KEY-VON-R1>"
/ip address
add address=10.10.10.6/30 interface=ether1 network=10.10.10.4
add address=3.3.3.3 interface=Loopback1 network=3.3.3.3
add address=192.168.20.1/24 interface=ether3 network=192.168.20.0
add address=10.0.0.2/30 interface=wg1 network=10.0.0.0
/ip dhcp-client
add interface=ether1
/ip firewall filter
add action=accept chain=input dst-port=51820 protocol=udp src-address=1.1.1.1
add action=accept chain=forward dst-address=192.168.10.0/24 src-address=192.168.20.0/24
add action=accept chain=forward dst-address=192.168.20.0/24 src-address=192.168.10.0/24
/ip route
add dst-address=192.168.10.0/24 gateway=wg1
/routing ospf interface-template
add area=Internet disabled=no networks=10.10.10.4/30
add area=Internet disabled=no networks=3.3.3.3/32
```

---

## Tunnel überprüfen

```routeros
# Peer-Status anzeigen (last-handshake sollte aktuell sein)
/interface wireguard peers print

# Tunnel-IP pingen
ping 10.0.0.2   # von R1
ping 10.0.0.1   # von R3

# LAN-zu-LAN pingen
ping 192.168.20.1   # von R1
ping 192.168.10.1   # von R3

# OSPF-Routen prüfen (Loopbacks müssen gelernt sein)
/ip route print
/routing ospf neighbor print
```

**Erwartete Route auf R1:**
```
1.1.1.1/32    – direkt (Loopback)
3.3.3.3/32    – via OSPF (muss vorhanden sein!)
10.0.0.0/30   – direkt (wg1)
192.168.20.0/24 – via wg1
```

> ⚠️ Wenn `3.3.3.3` **nicht** in der Routing-Tabelle von R1 steht, kann der Tunnel nicht aufgebaut werden – OSPF zuerst prüfen!

---

## Häufige Fehler

| Problem | Ursache | Lösung |
|---------|---------|--------|
| Tunnel baut sich nicht auf | Endpoint nicht erreichbar | OSPF-Route zur Loopback prüfen |
| `last-handshake` fehlt | Falsche Keys | Public Keys nochmal vergleichen |
| Ping durch Tunnel schlägt fehl | Fehlende Route oder Firewall | `/ip route print` und Firewall-Regeln prüfen |
| Interface-IP fehlt Prefix | `/32` statt `/30` gesetzt | `ip address` nochmal mit `/30` setzen |

---

> **Hinweis:** Diese Anleitung gilt für **RouterOS v7**. In v6 ist die OSPF-Syntax anders.
