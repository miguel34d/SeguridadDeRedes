# 📑 Documento de Validación y Configuración de Dispositivos Finales

![FortiGate](https://img.shields.io/badge/FortiGate-VM64-red?logo=fortinet&style=for-the-badge)
![Validation](https://img.shields.io/badge/Validation-Lab_Report-blue?style=for-the-badge)
![Devices](https://img.shields.io/badge/Devices-Win_Ubuntu_Switch-orange?style=for-the-badge)
![GitHub](https://img.shields.io/badge/GitHub-Repo-black?style=for-the-badge)

## 📋 Descripción del Documento
Este documento detalla la configuración de los dispositivos finales (Windows, Servidores Ubuntu y Switch) que forman parte de la topología conectada al FortiGate VM, incluyendo direccionamiento IP, rutas, segmentación por VLANs y configuraciones permanentes.

---

## 🖥️ PARTE 1: Configuración de Dispositivos Finales

### 1. Estación de Trabajo (Windows - VLAN 10 Usuarios)

| Campo | Valor Configurado |
| :--- | :--- |
| **Dirección IP** | `10.13.67.10` (o asignada por DHCP) |
| **Máscara de Subred** | `255.255.255.128` (/25) |
| **Puerta de Enlace** | `10.13.67.1` (FortiGate Port2) |
| **Servidor DNS** | `8.8.8.8` |

**Configuración Permanente mediante CMD (como Administrador):**
```cmd
netsh interface ip set address "Ethernet" static 10.13.67.10 255.255.255.128 10.13.67.1
netsh interface ip set dns "Ethernet" static 8.8.8.8
ipconfig /all
```

**Verificación:**
```cmd
ipconfig
ping 10.13.67.1
ping 10.10.10.1
```

---

### 2. Servidor Web (Ubuntu - VLAN 20 DMZ)

| Campo | Valor Configurado |
| :--- | :--- |
| **Dirección IP** | `20.13.67.2` |
| **Máscara de Subred** | `255.255.255.248` (/29) |
| **Puerta de Enlace** | `20.13.67.1` (FortiGate Port3) |
| **Servidor DNS** | `8.8.8.8` |
| **Interfaz de Red** | `ens3` |

**Configuración Temporal (para pruebas rápidas):**
```bash
sudo ip addr flush dev ens3
sudo ip addr add 20.13.67.2/29 dev ens3
sudo ip link set ens3 up
sudo ip route add default via 20.13.67.1
```

**Configuración Permanente mediante Netplan:**

Editar el archivo de configuración:
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

Contenido del archivo:
```yaml
network:
  version: 2
  ethernets:
    ens3:
      addresses:
        - 20.13.67.2/29
      gateway4: 20.13.67.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
      dhcp4: no
```

Aplicar los cambios:
```bash
sudo netplan apply
sudo netplan generate
```

**Verificación:**
```bash
ip addr show ens3
ip route show
ping 20.13.67.1
ping 20.13.67.3
```

---

### 3. Servidor de Base de Datos (Ubuntu - VLAN 20 DMZ)

| Campo | Valor Configurado |
| :--- | :--- |
| **Dirección IP** | `20.13.67.3` |
| **Máscara de Subred** | `255.255.255.248` (/29) |
| **Puerta de Enlace** | `20.13.67.1` (FortiGate Port3) |
| **Servidor DNS** | `8.8.8.8` |
| **Interfaz de Red** | `ens3` |

**Configuración Temporal (para pruebas rápidas):**
```bash
sudo ip addr flush dev ens3
sudo ip addr add 20.13.67.3/29 dev ens3
sudo ip link set ens3 up
sudo ip route add default via 20.13.67.1
```

**Configuración Permanente mediante Netplan:**

Editar el archivo de configuración:
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

Contenido del archivo:
```yaml
network:
  version: 2
  ethernets:
    ens3:
      addresses:
        - 20.13.67.3/29
      gateway4: 20.13.67.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
      dhcp4: no
```

Aplicar los cambios:
```bash
sudo netplan apply
sudo netplan generate
```

**Verificación:**
```bash
ip addr show ens3
ip route show
ping 20.13.67.1
ping 20.13.67.2
```

---

### 4. Switch de Acceso (Switch-DMZ)

Configuración de VLANs y puertos para segmentar el tráfico de servidores.

| Puerto | Modo | VLAN Asignada | Native VLAN | Dispositivo Conectado |
| :--- | :--- | :--- | :--- | :--- |
| **Et0/0** | Trunk (802.1Q) | 10, 20 | **20** | FortiGate (Port3) |
| **Et0/1** | Access | 20 | - | ServidorWeb (Ubuntu) |
| **Et0/2** | Access | 20 | - | BaseDeDatos (Ubuntu) |

> ⚠️ **NOTA IMPORTANTE:** El **Native VLAN del trunk debe ser 20** para que coincida con la VLAN de los servidores. Si el native VLAN es 1 (por defecto), el tráfico sin tag del FortiGate se asigna a VLAN 1 y no llega a los servidores en VLAN 20.

**Configuración CLI Completa y Corregida (Sintaxis tipo Cisco/GNS3):**

```bash
enable
configure terminal

! Nombre del switch
hostname Switch-DMZ

! Crear VLAN 20 para servidores
vlan 20
 name DMZ-SERVIDORES
 exit

! Configurar puerto Et0/0 como Trunk hacia FortiGate
! IMPORTANTE: Configurar native VLAN 20 para coincidir con la VLAN de los servidores
interface Et0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 20
 switchport trunk allowed vlan 10,20
 no shutdown
 exit

! Configurar puerto Et0/1 para ServidorWeb (VLAN 20)
interface Et0/1
 switchport mode access
 switchport access vlan 20
 no shutdown
 exit

! Configurar puerto Et0/2 para BaseDeDatos (VLAN 20)
interface Et0/2
 switchport mode access
 switchport access vlan 20
 no shutdown
 exit

end
write memory
```

**Verificación en el Switch:**
```bash
show vlan brief
show interfaces trunk
show interfaces Et0/0 status
show interfaces Et0/1 status
show interfaces Et0/2 status
show running-config
```

**Salida esperada de `show interfaces trunk`:**
```
Port        Mode    Encapsulation  Status    Native vlan
Et0/0       on      802.1q         trunking  20

Port        Vlans allowed on trunk
Et0/0       10,20

Port        Vlans allowed and active in management domain
Et0/0       20

Port        Vlans in spanning tree forwarding state and not pruned
Et0/0       20
```

---

## ✅ PARTE 2: Resumen de Pruebas de Conectividad

| Prueba | Origen | Destino | Resultado Esperado | Estado |
| :--- | :--- | :--- | :--- | :---: |
| Ping a Gateway LAN | Windows (`10.13.67.10`) | FortiGate (`10.13.67.1`) | Éxito (Reply) | ✅ |
| Ping a Internet | Windows (`10.13.67.10`) | Cloud (`10.10.10.1`) | Éxito (Reply con NAT) | ✅ |
| Ping a Gateway DMZ | ServidorWeb (`20.13.67.2`) | FortiGate (`20.13.67.1`) | Éxito (Reply) | ✅ |
| Ping entre Servidores | ServidorWeb (`20.13.67.2`) | BaseDeDatos (`20.13.67.3`) | Éxito (misma VLAN 20) | ✅ |
| Ping FortiGate a Servidor | FortiGate (`20.13.67.1`) | ServidorWeb (`20.13.67.2`) | Éxito (Reply) | ✅ |
| Trunk Switch-FortiGate | Switch-DMZ | FortiGate Port3 | VLAN 10 y 20 permitidas, Native VLAN 20 | ✅ |
| Navegación HTTP | Windows | ServidorWeb | Acceso al servicio web | ✅ |

---

##  Tabla Resumen de Direccionamiento IP

| Dispositivo | Interfaz | IP | Máscara | Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **Cloud/ISP** | - | `10.10.10.1` | `/24` | - |
| **FortiGate** | port1 (WAN) | `10.10.10.2` | `/24` | `10.10.10.1` |
| **FortiGate** | port2 (LAN) | `10.13.67.1` | `/25` | - |
| **FortiGate** | port3 (DMZ) | `20.13.67.1` | `/29` | - |
| **Windows** | NIC1 | `10.13.67.10` | `/25` | `10.13.67.1` |
| **ServidorWeb** | ens3 | `20.13.67.2` | `/29` | `20.13.67.1` |
| **BaseDeDatos** | ens3 | `20.13.67.3` | `/29` | `20.13.67.1` |
| **Switch-DMZ** | Et0/0 (Trunk, Native V20) | - | - | - |
| **Switch-DMZ** | Et0/1 (Access V20) | - | - | - |
| **Switch-DMZ** | Et0/2 (Access V20) | - | - | - |

---

## 🔧 Notas de Configuración Permanente

### Windows
- La configuración con `netsh` es permanente y persiste después de reiniciar.

### Ubuntu (Servidores)
- La configuración con `ip addr` es **temporal** (se pierde al reiniciar).
- La configuración con **Netplan** (`/etc/netplan/01-netcfg.yaml`) es **permanente**.
- Después de editar el archivo, siempre ejecutar `sudo netplan apply`.

### Switch-DMZ
- La configuración se guarda con `write memory` o `copy running-config startup-config`.
- Persiste después de reiniciar el switch.

### FortiGate
- Los cambios en la GUI se guardan automáticamente.
- En CLI, los cambios se aplican inmediatamente al ejecutar `end`.

---

<br>

### 👨‍ Realizado por: **Miguel Ramirez Meli**

![Author](https://img.shields.io/badge/Author-Miguel_Ramirez_Meli-orange?style=for-the-badge)
![Lab](https://img.shields.io/badge/Lab-FortiGate_VM64-purple?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Validated_and_Completed-brightgreen?style=for-the-badge)
