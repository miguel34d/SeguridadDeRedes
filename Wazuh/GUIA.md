<div align="center">

# 🔐 Segmentación de Red DMZ / LAN con VLANs, ACLs y SIEM (Wazuh) — GNS3

![Autor](https://img.shields.io/badge/Autor-Miguel%20Ramirez%20Meli-blueviolet?style=for-the-badge)
![Matricula](https://img.shields.io/badge/Matricula-2025--1367-informational?style=for-the-badge)
![Institucion](https://img.shields.io/badge/Instituci%C3%B3n-ITLA-red?style=for-the-badge)
![Materia](https://img.shields.io/badge/Materia-Seguridad%20de%20Redes-orange?style=for-the-badge)
![GNS3](https://img.shields.io/badge/GNS3-Network%20Simulation-1f6f8b?style=for-the-badge&logo=gns3)
![Cisco IOS](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=for-the-badge&logo=cisco)
![Wazuh](https://img.shields.io/badge/SIEM-Wazuh%204.9.2-005571?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completado-success?style=for-the-badge)

</div>

---

## 📑 Índice

- [1. Descripción general](#1-descripción-general)
- [2. Requisitos del entorno](#2-requisitos-del-entorno)
- [3. Dispositivos del laboratorio](#3-dispositivos-del-laboratorio)
- [4. Topología](#4-topología)
- [5. Tabla de direccionamiento](#5-tabla-de-direccionamiento)
- [6. Router1](#6-router1)
- [7. SwitchDMZ](#7-switchdmz)
- [8. SwitchUSERs](#8-switchusers)
- [9. Wazuh SIEM (UbuntuServer-1)](#9-wazuh-siem-ubuntuserver-1)
- [10. Pruebas de validación](#10-pruebas-de-validación)
- [11. Lecciones técnicas clave](#11-lecciones-técnicas-clave)

---

## 1. Descripción general

Laboratorio de segmentación de red en tres zonas (WAN, DMZ, LAN interna) sobre GNS3, con un router Cisco IOS como firewall perimetral, VLANs de gestión en los switches, y un SIEM Wazuh centralizando la telemetría de seguridad de un servidor web (contenedor Docker) en la DMZ, una estación Windows de administración y una máquina Kali de pruebas de intrusión en la LAN/DMZ.

---

## 2. Requisitos del entorno

| Recurso | Mínimo recomendado | Notas |
|---|---|---|
| **Disco duro (host Fedora)** | **100 GB libres** | GNS3 + imágenes QEMU/Docker (Windows 10, Ubuntu Server, Cisco IOS, Kali) consumen espacio rápidamente; Wazuh Manager por sí solo requiere holgura para el indexer (OpenSearch) |
| RAM del host | 16 GB | Repartidos entre Router1, 2 switches, WEB-1 (Docker), Windows10-1, UbuntuServer-1 y KaliDocker-1 corriendo simultáneamente |
| RAM — UbuntuServer-1 (Wazuh) | 6 GB | Por debajo de 4GB, el instalador de Wazuh rechaza continuar salvo con la bandera `-i` |
| CPU | 4 núcleos o más | GNS3 + QEMU + Docker en paralelo son intensivos en CPU |
| Software | GNS3 2.2+, Docker Engine, soporte QEMU/KVM | Host Fedora con `virbr-gns3` y `tap0` configurados para salida a Internet |

---

## 3. Dispositivos del laboratorio

| Dispositivo | Tipo | Rol | SO / Imagen |
|---|---|---|---|
| 🌐 **Cloud1** | Nube GNS3 | Simula el ISP / Internet | — |
| 🔀 **Router1** | Router virtual | Firewall / Gateway perimetral | Cisco IOSv |
| 🔌 **SwitchDMZ** | Switch virtual | Segmento DMZ (VLAN 10) | Cisco IOSvL2 |
| 🔌 **SwitchUSERs** | Switch virtual | Segmento LAN (VLAN 20) | Cisco IOSvL2 |
| 🐳 **WEB-1** | Contenedor **Docker** | Servidor web expuesto (DMZ) | Ubuntu 22.04 (contenedor) |
| 🪟 **Windows10-1** | Máquina virtual **Windows** | Estación de administración (LAN) | Windows 10 Enterprise LTSC |
| 🐧 **UbuntuServer-1** | Máquina virtual **Ubuntu Server** | **SIEM — Wazuh Manager** (LAN) | Ubuntu Server 22.04 |
| 🐉 **KaliDocker-1** | Contenedor **Docker** | Equipo atacante / pruebas de intrusión | Kali Linux (contenedor) |

> El laboratorio combina intencionalmente **3 tipos de virtualización** (Cisco IOS, VM completa, contenedor Docker) para representar un entorno híbrido realista, y usa **Kali como atacante controlado** para validar la segmentación desde una perspectiva ofensiva.

---

## 4. Topología

```
                              ┌────────────┐
                  (Internet)  │   Cloud1   │
                  10.10.10.1  │    (ISP)   │
                              └─────┬──────┘
                                    │ e0/0 → 10.10.10.2/24 (WAN)
                              ┌─────┴──────┐
                              │   Router1  │  🔀 Firewall / Gateway
                              │ Cisco IOSv │     NAT + ACL_INTERNET_IN
                              └──┬──────┬──┘
                        e0/1     │      │     e0/2
             10.13.67.1/24 ──────┘      └────── 20.13.67.1/24
                DMZ · VLAN 10                    LAN · VLAN 20
                  (ACL_DMZ_IN)                    (ACL_LAN_IN)
                     │                                  │
              ┌──────┴──────┐                   ┌───────┴───────┐
              │  SwitchDMZ  │ 🔌                │  SwitchUSERs  │ 🔌
              │ 10.13.67.2  │                   │  20.13.67.2   │
              └───┬─────┬───┘                   └───┬───────┬───┘
             e0/1  │     │ e0/2                 e0/1 │       │ e0/2
                   │     │                            │       │
         ┌─────────┴──┐ ┌┴────────────┐    ┌──────────┴──┐  ┌─┴───────────────┐
         │   WEB-1    │ │ KaliDocker-1│    │ Windows10-1 │  │ UbuntuServer-1  │
         │ 🐳 Docker  │ │ 🐉 Docker   │    │ 🪟 Windows  │  │ 🐧 Ubuntu Server│
         │ Servidor   │ │  Atacante   │    │Administrador│  │  SIEM — Wazuh   │
         │   Web      │ │  (pruebas)  │    │             │  │    Manager      │
         │10.13.67.10 │ │ 10.13.67.20 │    │ 20.13.67.11 │  │  20.13.67.10    │
         └────────────┘ └─────────────┘    └─────────────┘  └─────────────────┘
              DMZ              DMZ               LAN               LAN
```

---

## 5. Tabla de direccionamiento

| Dispositivo | Tipo | Interfaz | IP | Máscara | Gateway | Zona |
|---|---|---|---|---|---|---|
| Cloud1 (ISP) | Nube | — | 10.10.10.1 | /24 | — | WAN |
| Router1 | Cisco IOSv | e0/0 | 10.10.10.2 | /24 | — | WAN |
| Router1 | Cisco IOSv | e0/1 | 10.13.67.1 | /24 | — | DMZ (gateway) |
| Router1 | Cisco IOSv | e0/2 | 20.13.67.1 | /24 | — | LAN (gateway) |
| SwitchDMZ | Cisco IOSvL2 | vlan 10 | 10.13.67.2 | /24 | 10.13.67.1 | DMZ (gestión) |
| SwitchUSERs | Cisco IOSvL2 | vlan 20 | 20.13.67.2 | /24 | 20.13.67.1 | LAN (gestión) |
| 🐳 **WEB-1** | **Docker** (Ubuntu 22.04) | eth0 | 10.13.67.10 | /24 | 10.13.67.1 | DMZ |
| 🐉 **KaliDocker-1** | **Docker** (Kali Linux) | eth0 | 10.13.67.20 | /24 | 10.13.67.1 | DMZ |
| 🪟 **Windows10-1** | **Windows** 10 Enterprise LTSC | NIC1 | 20.13.67.11 | /24 | 20.13.67.1 | LAN |
| 🐧 **UbuntuServer-1** | **Ubuntu Server** 22.04 (Wazuh) | eth0 | 20.13.67.10 | /24 | 20.13.67.1 | LAN |

---

## 6. Router1

### 6.1 Direccionamiento IP de interfaces

```cisco
enable
configure terminal
hostname Router1
no ip domain-lookup

interface e0/0
 description WAN-Internet-Cloud1
 ip address 10.10.10.2 255.255.255.0
 ip nat outside
 no shutdown

interface e0/1
 description DMZ-SwitchDMZ
 ip address 10.13.67.1 255.255.255.0
 ip nat inside
 no shutdown

interface e0/2
 description LAN-SwitchUSERs
 ip address 20.13.67.1 255.255.255.0
 ip nat inside
 no shutdown
```

### 6.2 NAT hacia Internet

```cisco
ip access-list standard NAT_ACL
 permit 10.13.67.0 0.0.0.255
 permit 20.13.67.0 0.0.0.255

ip nat inside source list NAT_ACL interface Ethernet0/0 overload
```

> ⚠️ **Nota crítica:** el NAT es indispensable y fácil de perder al reescribir configuraciones. Sin estas líneas, ninguna red interna tiene salida a Internet aunque el enrutamiento esté correcto.

### 6.3 ACLs de segmentación

```cisco
! ACL_INTERNET_IN — aplicada en e0/0 (entrada desde Internet)
ip access-list extended ACL_INTERNET_IN
 10 permit tcp any host 10.13.67.10 eq 80
 20 permit tcp any host 10.13.67.10 eq 443
 30 permit tcp any any established
 40 permit udp any eq 53 any
 50 permit icmp any any echo-reply
 60 permit icmp any any time-exceeded
 70 permit icmp any any unreachable
 80 deny ip any any log

! ACL_DMZ_IN — aplicada en e0/1 (entrada desde la DMZ)
ip access-list extended ACL_DMZ_IN
 10 permit tcp 10.13.67.0 0.0.0.255 host 20.13.67.10 eq 1514
 20 permit tcp 10.13.67.0 0.0.0.255 host 20.13.67.10 eq 1515
 30 permit tcp 10.13.67.0 0.0.0.255 20.13.67.0 0.0.0.255 established
 40 deny ip 10.13.67.0 0.0.0.255 20.13.67.0 0.0.0.255 log
 50 permit ip any any

! ACL_LAN_IN — aplicada en e0/2 (entrada desde la LAN)
ip access-list extended ACL_LAN_IN
 permit ip any any

interface e0/0
 ip access-group ACL_INTERNET_IN in
interface e0/1
 ip access-group ACL_DMZ_IN in
interface e0/2
 ip access-group ACL_LAN_IN in
```

> ⚠️ **Nota crítica sobre el orden de las ACLs:** Cisco IOS agrega líneas nuevas al final de la lista si no se especifica número de secuencia. Esto puede colocar un `permit` amplio *antes* de un `deny` específico, anulando la restricción sin previo aviso. Siempre usar números de secuencia explícitos (10, 20, 30...) al construir o editar una ACL con múltiples reglas.

### 6.4 Ruteo

```cisco
ip route 0.0.0.0 0.0.0.0 10.10.10.1
end
write memory
```

---

## 7. SwitchDMZ

```cisco
enable
configure terminal
hostname SwitchDMZ
vtp mode transparent
no ip domain-lookup

vlan 10

interface e0/0
 description Uplink-Router1-e0/1
 switchport mode access
 switchport access vlan 10

interface e0/1
 description Host-WEB-1
 switchport mode access
 switchport access vlan 10

interface vlan 10
 ip address 10.13.67.2 255.255.255.0
 no shutdown

ip default-gateway 10.13.67.1
end
write memory
```

---

## 8. SwitchUSERs

```cisco
enable
configure terminal
hostname SwitchUSERs
vtp mode transparent
no ip domain-lookup

vlan 20

interface e0/0
 description Uplink-Router1-e0/2
 switchport mode access
 switchport access vlan 20

interface e0/1
 description Host-Windows10-1
 switchport mode access
 switchport access vlan 20

interface e0/2
 description Host-UbuntuServer-Wazuh
 switchport mode access
 switchport access vlan 20

interface vlan 20
 ip address 20.13.67.2 255.255.255.0
 no shutdown

ip default-gateway 20.13.67.1
end
write memory
```

---

## 9. Wazuh SIEM (UbuntuServer-1)

### 9.1 Instalación del Manager (UbuntuServer-1, 20.13.67.10)

```bash
curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh
sudo bash wazuh-install.sh -a -i
```

> El flag `-i` ignora el requisito mínimo de hardware (4GB RAM / 2 CPU) recomendado por Wazuh. Se recomienda asignar al menos 6GB de RAM a la VM para evitar inestabilidad del indexer.

Acceso al dashboard: `https://20.13.67.10:443` (usuario `admin`, password generado en el resumen de instalación).

### 9.2 🐳 Agente en WEB-1 (DMZ, contenedor Docker)

```bash
apt-get update && apt-get install -y curl
curl -o wazuh-agent.deb https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.9.2-1_amd64.deb
WAZUH_MANAGER='20.13.67.10' dpkg -i ./wazuh-agent.deb

/var/ossec/bin/agent-auth -m 20.13.67.10 -A WEB-1
/var/ossec/bin/wazuh-control start
```

> ⚠️ **Nota:** al ser un contenedor Docker sin `systemd`, el agente se administra con `wazuh-control` en lugar de `systemctl`. Además, el contenedor no tiene persistencia — cada vez que se reinicia, hay que reinstalar/reconfigurar el agente y volver a levantar cualquier servicio manual (servidor web, rsyslog, etc.).

### 9.3 🐉 KaliDocker-1 — Equipo atacante (DMZ)

Usado exclusivamente para las pruebas de intrusión y validación de la segmentación, no lleva agente Wazuh (representa un atacante externo/comprometido, no un endpoint gestionado).

```bash
# Configuración de red del contenedor
ip addr flush dev eth0
ip addr add 10.13.67.20/24 dev eth0
ip route add default via 10.13.67.1
echo "nameserver 8.8.8.8" > /etc/resolv.conf

# Herramientas usadas en las pruebas
which hydra nmap
```

### 9.4 🪟 Agente en Windows10-1 (LAN)

Instalación gráfica del `.msi`, luego:

```powershell
cd "C:\Program Files (x86)\ossec-agent"
agent-auth.exe -m 20.13.67.10 -A Administrador
```

Arranque vía `services.msc` → **Wazuh Agent** → Iniciar, o desde la GUI `win32ui.exe` (ejecutar como administrador) → Manager IP → Save → Manage → Start.

---

## 10. Pruebas de validación

| # | Prueba | Origen → Destino | Resultado esperado | Confirmado |
|---|---|---|---|---|
| 1 | Acceso web público | Internet → WEB-1:80/443 | Permitido | ✅ |
| 2 | Gestión desde LAN | Windows10-1 → WEB-1 | Permitido (`established`) | ✅ |
| 3 | Pivote lateral DMZ→LAN | WEB-1/Kali (DMZ) → LAN (20.13.67.0/24) | Bloqueado (`ACL_DMZ_IN`, log activo) | ✅ |
| 4 | Salida a Internet desde DMZ | WEB-1/Kali → Internet | Permitido vía NAT | ✅ |
| 5 | Comunicación agente-manager | WEB-1 (DMZ) → Wazuh (LAN) puertos 1514/1515 | Permitido (excepción explícita) | ✅ |
| 6 | Ataque de fuerza bruta SSH | Kali (DMZ) → WEB-1 (mismo segmento) | Detectado por Wazuh (agente WEB-1) | ✅ |
| 7 | Escaneo de puertos hacia LAN | Kali (DMZ) → Windows10-1 / UbuntuServer-1 | Todos los puertos `filtered`, log en Router1 | ✅ |

### Evidencia de bloqueo DMZ→LAN (Router1, logging en tiempo real)

```
%SEC-6-IPACCESSLOGP: list ACL_DMZ_IN denied tcp 10.13.67.20(...) -> 20.13.67.20(135), 1 packet
%SEC-6-IPACCESSLOGP: list ACL_DMZ_IN denied tcp 10.13.67.20(...) -> 20.13.67.10(3389), 1 packet
```

### Evidencia de detección de ataque SSH (WEB-1, /var/log/auth.log)

```
sshd[5679]: Failed password for root from 10.13.67.20 port 52998 ssh2
```

---

## 11. Lecciones técnicas clave

1. **Las ACLs de Cisco son stateless.** Un `deny` entre dos subredes también bloquea las respuestas legítimas de conexiones iniciadas por el lado permitido. Solución: agregar `permit tcp <origen> <destino> established` **antes** del `deny`.
2. **`established` solo aplica a TCP.** El tráfico UDP (como respuestas DNS en el puerto 53) necesita reglas `permit udp` explícitas; de lo contrario, cualquier ACL con un `deny` de por medio bloquea las respuestas DNS aunque el NAT y el enrutamiento estén correctos.
3. **El orden de las líneas en una ACL importa más que su existencia.** Cisco agrega reglas nuevas al final si no se usa número de secuencia — esto puede colocar un `permit ip any any` antes de un `deny` específico, anulando la restricción sin generar ningún error visible.
4. **`show access-lists <nombre>` con contadores limpios (`clear access-list counters`) antes de cada prueba** es la forma más rápida y confiable de aislar en qué línea exacta se está deteniendo o dejando pasar un paquete — mucho más rápido que adivinar.
5. **Los contenedores Docker no tienen persistencia por defecto.** Cualquier configuración manual (IP, agente Wazuh, servidor web, rsyslog) se pierde al reiniciar el contenedor a menos que se incluya en la imagen o en un volumen persistente.
6. **NAT y ACLs se pueden perder accidentalmente** al reescribir configuraciones de router por partes. Siempre verificar con `show run | include nat` después de cualquier cambio grande.

---

<div align="center">

**Miguel Ramirez Meli** · Matrícula 2025-1367 · ITLA · Seguridad de Redes

Ver evidencia con capturas: [`EVIDENCIAS/EVIDENCIA.md`](./EVIDENCIAS/EVIDENCIA.md)

</div>
