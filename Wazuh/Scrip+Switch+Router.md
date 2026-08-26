# 🔐 Segmentación de Red DMZ / LAN con VLANs y ACLs — GNS3

![Autor](https://img.shields.io/badge/Autor-Miguel%20Ramirez%20Meli-blueviolet?style=for-the-badge)
![GNS3](https://img.shields.io/badge/GNS3-Network%20Simulation-1f6f8b?style=for-the-badge&logo=gns3)
![Cisco IOS](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=for-the-badge&logo=cisco)
![Status](https://img.shields.io/badge/Status-Completado-success?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)

---

## 📑 Índice

- [1. Router1](#1-router1)
  - [1.1 Direccionamiento IP de interfaces](#11-direccionamiento-ip-de-interfaces)
  - [1.2 Aplicando configuración de ACL](#12-aplicando-configuración-de-acl)
  - [1.3 Ruteo hacia Internet](#13-ruteo-hacia-internet)
- [2. SwitchDMZ](#2-switchdmz)
  - [2.1 Aplicando VLAN 10](#21-aplicando-vlan-10)
  - [2.2 Direccionamiento IP de gestión](#22-direccionamiento-ip-de-gestión)
- [3. SwitchUSERs](#3-switchusers)
  - [3.1 Aplicando VLAN 20](#31-aplicando-vlan-20)
  - [3.2 Direccionamiento IP de gestión](#32-direccionamiento-ip-de-gestión)
- [4. Tabla de direccionamiento](#4-tabla-de-direccionamiento)

---

## 1. Router1

### 1.1 Direccionamiento IP de interfaces

```
enable
configure terminal
hostname Router1

interface e0/0
 ip address 10.10.10.2 255.255.255.0
 no shutdown

interface e0/1
 ip address 10.13.67.1 255.255.255.0
 no shutdown

interface e0/2
 ip address 20.13.67.1 255.255.255.0
 no shutdown
```

### 1.2 Aplicando configuración de ACL

```
ip access-list extended ACL_INTERNET_IN
 permit tcp any host 10.13.67.10 eq 80
 permit tcp any host 10.13.67.10 eq 443
 permit tcp any any established
 permit icmp any any echo-reply
 permit icmp any any time-exceeded
 permit icmp any any unreachable
 deny ip any any log

ip access-list extended ACL_DMZ_IN
 deny ip 10.13.67.0 0.0.0.255 20.13.67.0 0.0.0.255 log
 permit ip any any

ip access-list extended ACL_LAN_IN
 permit ip any any

interface e0/0
 ip access-group ACL_INTERNET_IN in

interface e0/1
 ip access-group ACL_DMZ_IN in

interface e0/2
 ip access-group ACL_LAN_IN in
```

### 1.3 Ruteo hacia Internet

```
ip route 0.0.0.0 0.0.0.0 10.10.10.1

end
write memory
```

---

## 2. SwitchDMZ

### 2.1 Aplicando VLAN 10

```
enable
configure terminal
hostname SwitchDMZ

vlan 10

interface e0/0
 switchport mode access
 switchport access vlan 10

interface e0/1
 switchport mode access
 switchport access vlan 10
```

### 2.2 Direccionamiento IP de gestión

```
interface vlan 10
 ip address 10.13.67.2 255.255.255.0
 no shutdown

ip default-gateway 10.13.67.1

end
write memory
```

---

## 3. SwitchUSERs

### 3.1 Aplicando VLAN 20

```
enable
configure terminal
hostname SwitchUSERs

vlan 20

interface e0/0
 switchport mode access
 switchport access vlan 20

interface e0/1
 switchport mode access
 switchport access vlan 20

interface e0/2
 switchport mode access
 switchport access vlan 20
```

### 3.2 Direccionamiento IP de gestión

```
interface vlan 20
 ip address 20.13.67.2 255.255.255.0
 no shutdown

ip default-gateway 20.13.67.1

end
write memory
```

---

## 4. Tabla de direccionamiento

| Dispositivo | Interfaz | IP | Máscara | Gateway |
|---|---|---|---|---|
| Cloud1 / Fedora | ISP | 10.10.10.1 | /24 | — |
| Router1 | e0/0 | 10.10.10.2 | /24 | 10.10.10.1 |
| Router1 | e0/1 | 10.13.67.1 | /24 | — |
| Router1 | e0/2 | 20.13.67.1 | /24 | — |
| SwitchDMZ | vlan 10 | 10.13.67.2 | /24 | 10.13.67.1 |
| SwitchUSERs | vlan 20 | 20.13.67.2 | /24 | 20.13.67.1 |
| WEB-1 | eth0 | 10.13.67.10 | /24 | 10.13.67.1 |
| Windows10-1 | NIC1 | 20.13.67.11 | /24 | 20.13.67.1 |
| UbuntuServer-1 | eth0 | 20.13.67.10 | /24 | 20.13.67.1 |
