# ️ Guía de Configuración FortiGate: Topología WAN/LAN con NAT

![FortiGate](https://img.shields.io/badge/FortiGate-VM64-red?logo=fortinet&style=for-the-badge)
![Network](https://img.shields.io/badge/Network-Topology-blue?style=for-the-badge)
![Security](https://img.shields.io/badge/Security-Firewall-green?style=for-the-badge)
![GitHub](https://img.shields.io/badge/GitHub-Repo-black?style=for-the-badge)

## 📋 Descripción de la Topología

Esta guía detalla la configuración paso a paso de un FortiGate VM conectando una red de Usuarios (VLAN 10) y Servidores (VLAN 20) hacia Internet (Cloud/ISP), aplicando NAT, DHCP y políticas de seguridad.

| Interfaz | Alias | VLAN / Rol | Dirección IP | Máscara |
| :--- | :--- | :--- | :--- | :--- |
| **port1** | WAN-ISP | WAN | `10.10.10.2` | `/24` (255.255.255.0) |
| **port2** | LAN-USUARIOS | LAN (VLAN 10) | `10.13.67.1` | `/25` (255.255.255.128) |
| **port3** | LAN-SERVIDORES | LAN (VLAN 20) | `20.13.67.1` | `/29` (255.255.255.248) |

---

## 🟢 PARTE 1: Configuración WAN (Port1) y Ruta por Defecto

### 🖥️ Configuración mediante GUI

1. Ve a **Network** > **Interfaces**.
2. Haz doble clic en **port1**.
3. Configura:
   - **Alias:** `WAN-ISP`
   - **Role:** `WAN`
   - **Addressing mode:** `Manual`
   - **IP/Network Mask:** `10.10.10.2 / 255.255.255.0`
   - **Administrative Access:** Marca `PING`, `HTTPS`, `SSH`.
4. Haz clic en **OK**.
5. Ve a **Network** > **Static Routes** > **Create New**.
   - **Destination IP/Mask:** `0.0.0.0 / 0.0.0.0`
   - **Device:** `port1`
   - **Gateway:** `10.10.10.1`
6. Haz clic en **OK**.

### 💻 Configuración mediante CLI

```bash
config system interface
    edit "port1"
        set mode static
        set ip 10.10.10.2 255.255.255.0
        set allowaccess ping https ssh
        set role wan
    next
end

config router static
    edit 1
        set dst 0.0.0.0 0.0.0.0
        set gateway 10.10.10.1
        set device "port1"
    next
end
```

---

## 🔵 PARTE 2: Configuración LAN (Port2 y Port3)

### 🖥️ Configuración mediante GUI

#### Paso 1 — Configurar Port2 (LAN-USUARIOS)

1. Ve a **Network** > **Interfaces**.
2. Haz doble clic en **port2**:
   - **Alias:** `LAN-USUARIOS`
   - **Role:** `LAN`
   - **Addressing mode:** `Manual`
   - **IP/Network Mask:** `10.13.67.1 / 255.255.255.128`
   - **Administrative Access:** Marca `PING`, `HTTPS`, `SSH`.
3. Haz clic en **OK**.

#### Paso 2 — Configurar DHCP Server en Port2 (LAN-USUARIOS)

1. Ve a **Network** > **Interfaces**.
2. Haz doble clic en **port2** (LAN-USUARIOS).
3. Desplázate hasta la sección **DHCP Server** y configúralo así:

| Campo | Valor |
| :--- | :--- |
| **DHCP status** | `Enabled` |
| **Address range** | `10.13.67.10 - 10.13.67.126` |
| **Netmask** | `255.255.255.128` |
| **Default gateway** | `Same as Interface IP` (10.13.67.1) |
| **DNS server** | `Same as Interface IP` |
| **Lease time** | `604800` seconds (7 días) |

4. Haz clic en **OK**.

#### Paso 3 — Configurar Port3 (LAN-SERVIDORES)

1. Ve a **Network** > **Interfaces**.
2. Haz doble clic en **port3**:
   - **Alias:** `LAN-SERVIDORES`
   - **Role:** `LAN`
   - **Addressing mode:** `Manual`
   - **IP/Network Mask:** `20.13.67.1 / 255.255.255.248`
   - **Administrative Access:** Marca `PING`, `HTTPS`, `SSH`.
   - **DHCP Server:** `Disabled` (los servidores tienen IPs estáticas).
3. Haz clic en **OK**.

###  Configuración mediante CLI

```bash
config system interface
    edit "port2"
        set mode static
        set ip 10.13.67.1 255.255.255.128
        set allowaccess ping https ssh
        set role lan
    next
end

config system dhcp server
    edit 1
        set interface "port2"
        set status enable
        set netmask 255.255.255.128
        set start-ip 10.13.67.10
        set end-ip 10.13.67.126
        set default-gateway 10.13.67.1
        set dns-service default
        set lease-time 604800
    next
end

config system interface
    edit "port3"
        set mode static
        set ip 20.13.67.1 255.255.255.248
        set allowaccess ping https ssh
        set role lan
    next
end
```

---

## 🟠 PARTE 3: Políticas de Firewall y NAT (Salida a Internet)

Para que los usuarios y servidores puedan navegar, debemos crear una política que permita el tráfico desde las LAN hacia la WAN, activando **NAT**.

### ️ Configuración mediante GUI

1. Ve a **Policy & Objects** > **Firewall Policy**.
2. Haz clic en **Create New**.
3. Configura:
   - **Name:** `LAN_to_WAN_NAT`
   - **Incoming Interface:** Selecciona `LAN-USUARIOS` y `LAN-SERVIDORES` (mantén presionado Ctrl para seleccionar ambas).
   - **Outgoing Interface:** Selecciona `WAN-ISP`.
   - **Source:** `all`
   - **Destination:** `all`
   - **Schedule:** `always`
   - **Service:** `ALL`
   - **Action:** `ACCEPT`
   - **NAT:** **ACTIVADO** (Switch en ON).
4. Haz clic en **OK**.

### 💻 Configuración mediante CLI

```bash
config firewall policy
    edit 1
        set name "LAN_to_WAN_NAT"
        set srcintf "port2" "port3"
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

## ✅ Validación Final

Para verificar que todo esté correcto, ejecuta los siguientes comandos en la CLI:

```bash
# Verificar interfaces y IPs
get system interface

# Verificar configuración DHCP
show system dhcp server

# Verificar ruta por defecto
get router info routing-table all

# Probar conectividad hacia el ISP
execute ping 10.10.10.1
```

---

<br>

### 👨‍ Realizado por: **Miguel Ramirez Meli**

![Author](https://img.shields.io/badge/Author-Miguel_Ramirez_Meli-orange?style=for-the-badge)
![Lab](https://img.shields.io/badge/Lab-FortiGate_VM64-purple?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)
