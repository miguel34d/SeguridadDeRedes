# Configuración VPN Client to Site

## Resumen de estado

![Estado](https://img.shields.io/badge/Estado-Completado-brightgreen)
![Resultado](https://img.shields.io/badge/Resultado-VPN%20Client--to--Site%20funcional-brightgreen)
![Modelo](https://img.shields.io/badge/Modelo-Red%20VPN%20%2F24-blue)
![Simulador](https://img.shields.io/badge/Simulador-GNS3-lightgrey)
![IKE](https://img.shields.io/badge/IKE-IKEv2%20%2F%20DES--SHA256-orange)
![Dispositivos](https://img.shields.io/badge/Dispositivos-Router1%20%7C%20Switch1%20%7C%20Switch2%20%7C%20FortiGate%20%7C%20Windows10--1%20%7C%20Windows10--2-lightgrey)

## Diagrama de topología

![Topología VPN Client to Site](topologia-vpn-client-to-site.png)

## Topología y Direccionamiento

| Elemento | Dirección | Interfaz/VLAN |
|---|---|---|
| Windows10-1 (cliente VPN) | 10.13.67.10/24 | VLAN 10 |
| Gateway VLAN 10 | 10.13.67.1 | Router-ISP e0/1.10 |
| Router-ISP <-> Cloud1 | 10.10.10.2/24 | e0/1.1 |
| Router-ISP <-> FortiGate | 200.13.67.1/30 | e0/0 |
| FortiGate Port1 (WAN) | 200.13.67.2/30 | port1 |
| FortiGate Port2 (LAN) | 20.13.67.1/24 | port2 |
| Windows10-2 | 20.13.67.10/24 | VLAN 20 |
| Pool VPN (clientes remotos) | 30.13.67.10 - 30.13.67.254 /24 | Tunnel |

---

## 1. FortiGate - Interfaces

**GUI: Network > Interfaces > port1**

| Campo | Valor |
|---|---|
| Interface Name | port1 |
| Role | WAN |
| Addressing mode | Manual |
| IP/Netmask | 200.13.67.2/255.255.255.252 |
| Access | PING, HTTPS, FGFM |

**GUI: Network > Interfaces > port2**

| Campo | Valor |
|---|---|
| Interface Name | port2 |
| Role | LAN |
| Addressing mode | Manual |
| IP/Netmask | 20.13.67.1/255.255.255.0 |
| Access | PING, HTTPS, HTTP |

**CLI**

```
config system interface
    edit "port1"
        set mode static
        set ip 200.13.67.2 255.255.255.252
        set allowaccess ping https fgfm
    next
    edit "port2"
        set mode static
        set ip 20.13.67.1 255.255.255.0
        set allowaccess ping https http
    next
end
```

---

## 2. FortiGate - Ruta Estática

**GUI: Network > Static Routes > Create New**

| Campo | Valor |
|---|---|
| Destination | 0.0.0.0/0.0.0.0 |
| Interface | port1 |
| Gateway | 200.13.67.1 |

**CLI**

```
config router static
    edit 1
        set gateway 200.13.67.1
        set device "port1"
    next
end
```

---

## 3. FortiGate - Grupo de Usuarios VPN

**GUI: User & Authentication > User Groups > Create New**

| Campo | Valor |
|---|---|
| Name | VPN-USERS |
| Type | Firewall |
| Members | vpnuser1 |

**CLI**

```
config user group
    edit "VPN-USERS"
        set member "vpnuser1"
    next
end
```

---

## 4. FortiGate - Crear Usuario VPN

**GUI: User & Authentication > User Definition > Create New**

1. Local User > Next
2. Login Credentials:

| Campo | Valor |
|---|---|
| Username | vpnuser1 |
| Password | Vpnuser123! |

3. Contact Info: Next
4. Extra Info: User Account Status = Enabled
5. Submit

**CLI**

```
config user local
    edit "vpnuser1"
        set type password
        set passwd Vpnuser123!
    next
end
```

---

## 5. FortiGate - VPN IPsec Dial-Up (Wizard)

**GUI: VPN > IPsec Wizard > Create New**

| Campo | Valor |
|---|---|
| Name | Client-To-Site |
| Template type | Remote Access |
| Remote device type | FortiClient (Windows, Mac OS, Android) |

**Paso 2 - Authentication**

| Campo | Valor |
|---|---|
| Incoming Interface | port1 |
| Authentication method | Pre-shared Key |
| Pre-shared key | Fortinet123! |
| User Group | VPN-USERS |

**Paso 3 - Policy & Routing**

| Campo | Valor |
|---|---|
| Local interface | port2 |
| Local Address | 20.13.67.0/24 (objeto "LAN-INTERNA") |
| Client Address Range | 30.13.67.10-30.13.67.254 |
| Subnet Mask | 255.255.255.0 |
| Enable IPv4 Split Tunnel | Desactivado |

**Paso 4 - Client Options**

| Campo | Valor |
|---|---|
| Save Password | Activado |
| Auto Connect | Desactivado |
| Always Up (Keep Alive) | Desactivado |

**Paso 5**: Review Settings > Create

**CLI**

```
config firewall address
    edit "LAN-INTERNA"
        set type iprange
        set subnet 20.13.67.0 255.255.255.0
    next
end
```

---

## 6. FortiGate - Ajustar Fase 1 y Fase 2 del Túnel

**GUI: VPN > IPsec Tunnels > Client-To-Site > Edit**

**XAUTH**: User Group = Inherit from policy

**Authentication**

| Campo | Valor |
|---|---|
| Authentication Method | Pre-shared Key |
| IKE Version | 2 |
| Peer Options / Accept Types | Any peer ID |

**Phase 1 Proposal**

| Campo | Valor |
|---|---|
| Encryption | DES |
| Authentication | SHA256 |
| Diffie-Hellman Groups | 14 |
| Key Lifetime (seconds) | 86400 |

**Phase 2 Selectors**

| Campo | Valor |
|---|---|
| Local/Remote Address | 0.0.0.0/0.0.0.0 |
| Encryption | DES |
| Authentication | SHA256 |
| Diffie-Hellman Group | 14 |
| Enable PFS | Activado |
| Key Lifetime | 43200 segundos |

**CLI**

```
config vpn ipsec phase1-interface
    edit "Client-To-Site"
        set type dynamic
        set interface "port1"
        set ike-version 2
        set peertype any
        set net-device disable
        set mode-cfg enable
        set proposal des-sha256
        set dhgrp 14
        set dpd on-idle
        set dpd-retryinterval 60
        set ipv4-start-ip 30.13.67.10
        set ipv4-end-ip 30.13.67.254
        set ipv4-netmask 255.255.255.0
        set psksecret Fortinet123!
    next
end

config vpn ipsec phase2-interface
    edit "Client-To-Site"
        set phase1name "Client-To-Site"
        set proposal des-sha256
        set dhgrp 14
        set pfs enable
        set replay enable
        set keylifeseconds 43200
    next
end
```

---

## 7. FortiGate - Política: Túnel -> LAN

**GUI: Policy & Objects > Firewall Policy > Create New**

| Campo | Valor |
|---|---|
| Name | vpn_Client-To-Site_remote_0 |
| Incoming Interface | Client-To-Site (tunnel) |
| Outgoing Interface | port2 |
| Source | all |
| Destination | all |
| Schedule | always |
| Service | ALL |
| Action | ACCEPT |
| NAT | Desactivado |

**CLI**

```
config firewall policy
    edit 1
        set name "vpn_Client-To-Site_remote_0"
        set srcintf "Client-To-Site"
        set dstintf "port2"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set nat disable
    next
end
```

---

## 8. FortiGate - Política: LAN -> Túnel

**GUI: Policy & Objects > Firewall Policy > Create New**

| Campo | Valor |
|---|---|
| Name | LAN-TO-VPN |
| Incoming Interface | port2 |
| Outgoing Interface | Client-To-Site (tunnel) |
| Source | all |
| Destination | all |
| Schedule | always |
| Service | ALL |
| Action | ACCEPT |
| NAT | Desactivado |

**CLI**

```
config firewall policy
    edit 2
        set name "LAN-TO-VPN"
        set srcintf "port2"
        set dstintf "Client-To-Site"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set nat disable
    next
end
```

---

## 9. FortiGate - Política: LAN -> WAN (Internet)

**GUI: Policy & Objects > Firewall Policy > Create New**

| Campo | Valor |
|---|---|
| Name | LAN-to-WAN |
| Incoming Interface | port2 |
| Outgoing Interface | port1 |
| Source | all |
| Destination | all |
| Schedule | always |
| Service | ALL |
| Action | ACCEPT |
| NAT | Activado (Enable NAT) |

**CLI**

```
config firewall policy
    edit 3
        set name "LAN-to-WAN"
        set srcintf "port2"
        set dstintf "port1"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set nat enable
    next
end
```

---

## 10. FortiGate - Política: Túnel -> Túnel (Cliente a Cliente VPN)

**GUI: Policy & Objects > Firewall Policy > Create New**

| Campo | Valor |
|---|---|
| Name | VPN-TO-VPN |
| Incoming Interface | Client-To-Site (tunnel) |
| Outgoing Interface | Client-To-Site (tunnel) |
| Source | all |
| Destination | all |
| Schedule | always |
| Service | ALL |
| Action | ACCEPT |
| NAT | Desactivado |

**CLI**

```
config firewall policy
    edit 4
        set name "VPN-TO-VPN"
        set srcintf "Client-To-Site"
        set dstintf "Client-To-Site"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set nat disable
    next
end
```

---

## 11. FortiClient - Configuración de Conexión VPN

**GUI: FortiClient > Remote Access > Configure VPN**

| Campo | Valor |
|---|---|
| Tipo de VPN | VPN IPsec |
| Nombre de Conexión | Client-To-Site |
| Gateway Remoto | 200.13.67.2 |
| Método de Autenticación | Clave pre-compartida |
| Clave (PSK) | Fortinet123! |
| Autenticación (EAP) | Deshabilitar |

**Ajustes avanzados - Configuración de VPN**

| Campo | Valor |
|---|---|
| IKE | Versión 2 |
| Address Assignment | Modo de Configuración |
| Encapsulation | IKE UDP Port 500 |

**Fase 1**

| Campo | Valor |
|---|---|
| Encripción | DES |
| Autenticación | SHA256 |
| Grupo DH | 14 |
| Vida de Clave | 86400 seg |
| DPD | Habilitado |
| NAT Transversal | Habilitado |

**Fase 2**

| Campo | Valor |
|---|---|
| Encripción | DES |
| Autenticación | SHA256 |
| Grupo DH | 14 |
| Vida de Clave | 43200 seg |
| Detección de Repeticiones | Habilitado |
| PFS | Habilitado |

---

## 12. Windows10-2 (Host VLAN 20)

| Campo | Valor |
|---|---|
| IP | 20.13.67.10 |
| Máscara | 255.255.255.0 |
| Gateway | 20.13.67.1 |

---

## 13. Router-ISP (CLI)

```
enable
configure terminal
!
hostname Router-ISP
!
no ip domain-lookup
ip name-server 8.8.8.8
!
ip cef
no ipv6 cef
!
interface Ethernet0/0
 description WAN-hacia-FortiGate
 ip address 200.13.67.1 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 duplex auto
 no shutdown
!
interface Ethernet0/1
 description hacia-Switch1-trunk
 no ip address
 duplex auto
 no shutdown
!
interface Ethernet0/1.1
 description hacia-Cloud1-Internet
 encapsulation dot1Q 1 native
 ip address 10.10.10.2 255.255.255.0
 ip nat outside
!
interface Ethernet0/1.10
 description Segmento-Cliente-VPN-Windows10-1
 encapsulation dot1Q 10
 ip address 10.13.67.1 255.255.255.0
 ip nat inside
!
ip forward-protocol nd
no ip http server
no ip http secure-server
!
ip nat inside source list ACL-NAT interface Ethernet0/1.1 overload
!
ip route 0.0.0.0 0.0.0.0 10.10.10.1
ip route 20.13.67.0 255.255.255.0 200.13.67.2
!
ip access-list standard ACL-NAT
 permit 10.13.67.0 0.0.0.255
 permit 20.13.67.0 0.0.0.255
!
line con 0
 logging synchronous
line vty 0 4
 login
 transport input telnet ssh
!
end
write memory
```

---

## 14. Switch1 (CLI)

```
enable
configure terminal
!
hostname Switch-Windows
!
no ip domain-lookup
!
vlan 10
 name VLAN-CLIENTE-VPN
exit
!
interface Ethernet0/0
 description hacia-Cloud1-Internet
 switchport mode access
 switchport access vlan 1
 no shutdown
exit
!
interface Ethernet0/1
 description hacia-Windows-NIC1
 switchport mode access
 switchport access vlan 10
 no shutdown
exit
!
interface Ethernet0/2
 description hacia-Router-ISP-trunk
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 1
 switchport trunk allowed vlan 1,10
 no shutdown
exit
!
interface Ethernet0/3
 shutdown
exit
!
interface range Ethernet1/0 - 3 , Ethernet2/0 - 3 , Ethernet3/0 - 3
 shutdown
exit
!
line con 0
 logging synchronous
line vty 0 4
 login
 transport input telnet ssh
!
end
write memory
```

---

## 15. Switch2 (CLI)

```
enable
configure terminal
!
hostname Switch2
!
no ip domain-lookup
!
vlan 20
 name VLAN-LAN-INTERNA
exit
!
interface Ethernet0/0
 description hacia-FortiGate-Port2
 switchport mode access
 switchport access vlan 20
 no shutdown
exit
!
interface Ethernet0/1
 description hacia-Windows10-2
 switchport mode access
 switchport access vlan 20
 no shutdown
exit
!
interface Ethernet0/2
 shutdown
exit
!
interface Ethernet0/3
 shutdown
exit
!
line con 0
 logging synchronous
line vty 0 4
 login
 transport input telnet ssh
!
end
write memory
```
