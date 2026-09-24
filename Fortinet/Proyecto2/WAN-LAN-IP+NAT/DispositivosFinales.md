# 📑 Documento de Validación y Configuración de Dispositivos Finales

![FortiGate](https://img.shields.io/badge/FortiGate-VM64-red?logo=fortinet&style=for-the-badge)
![Validation](https://img.shields.io/badge/Validation-Lab_Report-blue?style=for-the-badge)
![Devices](https://img.shields.io/badge/Devices-Win_Ubuntu_Switch-orange?style=for-the-badge)
![GitHub](https://img.shields.io/badge/GitHub-Repo-black?style=for-the-badge)

## 📋 Descripción del Documento
Este documento detalla la configuración de los dispositivos finales (Windows, Servidores Ubuntu y Switch) que forman parte de la topología conectada al FortiGate VM, incluyendo direccionamiento IP, rutas y segmentación por VLANs.

---

## 🖥️ PARTE 1: Configuración de Dispositivos Finales

### 1. Estación de Trabajo (Windows - VLAN 10 Usuarios)
Configuración de la interfaz de red para obtener conectividad a través del FortiGate.

| Campo | Valor Configurado |
| :--- | :--- |
| **Dirección IP** | `10.13.67.10` (o asignada por DHCP) |
| **Máscara de Subred** | `255.255.255.128` (/25) |
| **Puerta de Enlace** | `10.13.67.1` (FortiGate Port2) |
| **Servidor DNS** | `8.8.8.8` |

**Comandos de configuración (CMD como Administrador):**
```cmd
netsh interface ip set address "Ethernet" static 10.13.67.10 255.255.255.128 10.13.67.1
netsh interface ip set dns "Ethernet" static 8.8.8.8
ipconfig /all
```

---

### 2. Servidor Web (Ubuntu - VLAN 20 DMZ)
Configuración estática mediante Netplan.

| Campo | Valor Configurado |
| :--- | :--- |
| **Dirección IP** | `20.13.67.2` |
| **Máscara de Subred** | `255.255.255.248` (/29) |
| **Puerta de Enlace** | `20.13.67.1` (FortiGate Port3) |
| **Servidor DNS** | `8.8.8.8` |

**Archivo de configuración (`/etc/netplan/01-netcfg.yaml`):**
```yaml
network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 20.13.67.2/29
      gateway4: 20.13.67.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

**Aplicar y verificar:**
```bash
sudo netplan apply
ip addr show eth0
ip route show
```

---

### 3. Servidor de Base de Datos (Ubuntu - VLAN 20 DMZ)
Configuración estática mediante Netplan.

| Campo | Valor Configurado |
| :--- | :--- |
| **Dirección IP** | `20.13.67.3` |
| **Máscara de Subred** | `255.255.255.248` (/29) |
| **Puerta de Enlace** | `20.13.67.1` (FortiGate Port3) |
| **Servidor DNS** | `8.8.8.8` |

**Archivo de configuración (`/etc/netplan/01-netcfg.yaml`):**
```yaml
network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 20.13.67.3/29
      gateway4: 20.13.67.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

**Aplicar y verificar:**
```bash
sudo netplan apply
ip addr show eth0
ip route show
```

---

### 4. Switch de Acceso (Switch-DMZ)
Configuración de VLANs y puertos para segmentar el tráfico de servidores.

| Puerto | Modo | VLAN Asignada | Dispositivo Conectado |
| :--- | :--- | :--- | :--- |
| **e0/0** | Trunk | 10, 20 | FortiGate (Port3) |
| **e0/1** | Access | 20 | BaseDeDatos (Ubuntu) |
| **e0/2** | Access | 20 | ServidorWeb (Ubuntu) |

**Configuración CLI (Sintaxis tipo Cisco/GNS3):**
```bash
enable
configure terminal
hostname Switch-DMZ

vlan 20
 name DMZ-SERVIDORES
 exit

interface e0/0
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
 exit

interface e0/1
 switchport mode access
 switchport access vlan 20
 no shutdown
 exit

interface e0/2
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
show running-config
```

---

## ✅ PARTE 2: Resumen de Pruebas de Conectividad

| Prueba | Origen | Destino | Resultado Esperado | Estado |
| :--- | :--- | :--- | :--- | :---: |
| Ping a Gateway LAN | Windows (`10.13.67.10`) | FortiGate (`10.13.67.1`) | Éxito (Reply) | ✅ |
| Ping a Internet | Windows (`10.13.67.10`) | Cloud (`10.10.10.1`) | Éxito (Reply con NAT) | ✅ |
| Ping a Gateway DMZ | Ubuntu Web (`20.13.67.2`) | FortiGate (`20.13.67.1`) | Éxito (Reply) | ✅ |
| Ping entre Servidores | ServidorWeb (`20.13.67.2`) | BaseDeDatos (`20.13.67.3`) | Éxito (misma VLAN 20) | ✅ |
| Trunk Switch-FortiGate | Switch-DMZ | FortiGate Port3 | VLAN 10 y 20 permitidas | ✅ |
| Navegación HTTP | Windows | ServidorWeb | Acceso al servicio web | ✅ |

---

## 📊 Tabla Resumen de Direccionamiento IP

| Dispositivo | Interfaz | IP | Máscara | Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **Cloud/ISP** | - | `10.10.10.1` | `/24` | - |
| **FortiGate** | port1 (WAN) | `10.10.10.2` | `/24` | `10.10.10.1` |
| **FortiGate** | port2 (LAN) | `10.13.67.1` | `/25` | - |
| **FortiGate** | port3 (DMZ) | `20.13.67.1` | `/29` | - |
| **Windows** | NIC1 | `10.13.67.10` | `/25` | `10.13.67.1` |
| **ServidorWeb** | eth0 | `20.13.67.2` | `/29` | `20.13.67.1` |
| **BaseDeDatos** | eth0 | `20.13.67.3` | `/29` | `20.13.67.1` |
| **Switch-DMZ** | e0/0 (Trunk) | - | - | - |
| **Switch-DMZ** | e0/1 (Access V20) | - | - | - |
| **Switch-DMZ** | e0/2 (Access V20) | - | - | - |

---

<br>

### 👨‍ Realizado por: **Miguel Ramirez Meli**

![Author](https://img.shields.io/badge/Author-Miguel_Ramirez_Meli-orange?style=for-the-badge)
![Lab](https://img.shields.io/badge/Lab-FortiGate_VM64-purple?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Validated_and_Completed-brightgreen?style=for-the-badge)
