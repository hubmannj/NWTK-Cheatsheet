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
