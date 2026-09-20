# 🌐 Acceso a Internet — FortiGate

![FortiGate](https://img.shields.io/badge/FortiGate-Firewall-red?style=flat-square&logo=fortinet&logoColor=white)
![GNS3](https://img.shields.io/badge/GNS3-Lab-0089D6?style=flat-square)
![GUI](https://img.shields.io/badge/GUI-Incluido-blue?style=flat-square)
![CLI](https://img.shields.io/badge/CLI-Incluido-brightgreen?style=flat-square)
![Status](https://img.shields.io/badge/Estado-Funcional-success?style=flat-square)
![Autor](https://img.shields.io/badge/Autor-Miguel_Ramirez_Meli-informational?style=flat-square)
![Matricula](https://img.shields.io/badge/Matr%C3%ADcula-20251367-informational?style=flat-square)
![Docente](https://img.shields.io/badge/Docente-Jonathan_Rond%C3%B3n-lightgrey?style=flat-square)
![Materia](https://img.shields.io/badge/Materia-Seguridad_de_Redes-yellow?style=flat-square)

Guía paso a paso para habilitar el acceso a Internet desde las LAN internas a través del FortiGate, incluyendo direccionamiento IP, DHCP, ruta por defecto y NAT.

## 📌 Topología de referencia

```
Internet — Router1 (e0/0) —— Port1 [FortiGate] Port2 —— Switch1 —— Windows10-1
                                              Port3 —— Switch2 —— WEB-1
```

| Segmento              | Red              | Puerto FortiGate |
|-----------------------|------------------|-------------------|
| WAN (hacia Router1)   | 200.13.67.0/30   | Port1             |
| LAN Usuarios          | 10.13.67.0/25    | Port2             |
| LAN Servidores        | 20.13.67.0/28    | Port3             |

![Topología](./images/topologia.png)

---

## 📑 Contenido

1. [IP en interfaces](#1-ip-en-interfaces)
2. [DHCP en LAN de usuarios](#2-dhcp-en-lan-de-usuarios)
3. [Ruta por defecto](#3-ruta-por-defecto)
4. [NAT](#4-nat)
5. [Verificar traducción de IPs (NAT) en los logs](#5-verificar-traducción-de-ips-nat-en-los-logs)

---

## 1. IP en interfaces

### 🖱️ GUI

**Port2 — LAN Usuarios (10.13.67.0/25)**
1. `Network > Interfaces > Edit port2`
2. Role: `LAN`
3. IP/Netmask: `10.13.67.1/255.255.255.128`
4. Administrative Access: `HTTPS`, `PING`
5. `OK`

**Port3 — LAN Servidores (20.13.67.0/28)**
1. `Network > Interfaces > Edit port3`
2. Role: `LAN`
3. IP/Netmask: `20.13.67.1/255.255.255.240`
4. `OK`

**Port1 — WAN (hacia Router1)**
1. `Network > Interfaces > Edit port1`
2. Role: `WAN`
3. IP/Netmask: `200.13.67.2/255.255.255.252`
4. Administrative Access: `PING`
5. `OK`

![Configuración de interfaces](./images/interfaces.png)

### ⌨️ CLI

```bash
config system interface
    edit "port2"
        set alias "LAN-USUARIOS"
        set role lan
        set ip 10.13.67.1 255.255.255.128
        set allowaccess ping https http
    next
    edit "port3"
        set alias "LAN-SERVIDORES"
        set role lan
        set ip 20.13.67.1 255.255.255.240
        set allowaccess ping
    next
    edit "port1"
        set alias "WAN"
        set role wan
        set ip 200.13.67.2 255.255.255.252
        set allowaccess ping
    next
end
```

---

## 2. DHCP en LAN de usuarios

### 🖱️ GUI

1. `Network > Interfaces > Edit port2`
2. Activar **DHCP Server**
3. Address Range: `10.13.67.2 – 10.13.67.126`
4. Netmask: `255.255.255.128`
5. Default Gateway: `Same as Interface IP` (`10.13.67.1`)
6. DNS Server: `Same as System DNS` (o `8.8.8.8`)
7. `OK`

![DHCP Server](./images/dhcp.png)

### ⌨️ CLI

```bash
config system dhcp server
    edit 0
        set interface "port2"
        set default-gateway 10.13.67.1
        set netmask 255.255.255.128
        set dns-service default
        config ip-range
            edit 1
                set start-ip 10.13.67.2
                set end-ip 10.13.67.126
            next
        end
    next
end
```

---

## 3. Ruta por defecto

### 🖱️ GUI

1. `Network > Static Routes > Create New`
2. Destination: `0.0.0.0/0.0.0.0`
3. Interface: `port1`
4. Gateway: `200.13.67.1`
5. `OK`

![Ruta estática](./images/ruta-default.png)

### ⌨️ CLI

```bash
config router static
    edit 0
        set dst 0.0.0.0 0.0.0.0
        set gateway 200.13.67.1
        set device "port1"
    next
end
```

---

## 4. NAT

### 🖱️ GUI

**Policy: LAN Usuarios → Internet**
1. `Policy & Objects > Firewall Policy > Create New`
2. Name: `LAN-Usuarios-to-WAN`
3. Incoming Interface: `port2`
4. Outgoing Interface: `port1`
5. Source: `all` — Destination: `all`
6. Schedule: `always` — Service: `ALL`
7. Action: `ACCEPT`
8. NAT: **ON** → `Use Outgoing Interface Address`
9. `OK`

**Policy: LAN Servidores → Internet**
1. `Create New`
2. Name: `LAN-Servidores-to-WAN`
3. Incoming Interface: `port3`
4. Outgoing Interface: `port1`
5. Source: `all` — Destination: `all`
6. Schedule: `always` — Service: `ALL`
7. Action: `ACCEPT`
8. NAT: **ON** → `Use Outgoing Interface Address`
9. `OK`

![Políticas NAT](./images/nat-policies.png)

### ⌨️ CLI

```bash
config firewall policy
    edit 0
        set name "LAN-Usuarios-to-WAN"
        set srcintf "port2"
        set dstintf "port1"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set nat enable
    next
    edit 0
        set name "LAN-Servidores-to-WAN"
        set srcintf "port3"
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

## 5. Verificar traducción de IPs (NAT) en los logs

Por defecto el log de tráfico local puede venir limitado y no mostrar las sesiones permitidas. Para ver la IP origen (LAN) y su traducción hacia la WAN en `Forward Traffic`, hay que habilitar el log completo primero.

### 🖱️ GUI

1. `Log & Report > Log Settings`
2. En **Local traffic logging**, selecciona `All` (o si usas `Customize`, activa el toggle **"Log allowed traffic"**)
3. Ve a `Log & Report > Forward Traffic`
4. Cambia el rango de tiempo (arriba a la derecha) de `5 minutes` a `1 hour`
5. Genera tráfico desde el cliente (ej. `ping 8.8.8.8` desde Windows10-1) y refresca
6. Haz clic en la sesión y luego en `Details` para ver **Source IP** (IP real de la LAN) vs **NAT'd Source IP / SNAT IP** (IP traducida hacia la WAN)

### ⌨️ CLI

```bash
config log setting
    set local-traffic enable
end
```

Ver sesiones NAT activas en tiempo real:

```bash
diagnose sys session filter src 10.13.67.0 255.255.255.0
diagnose sys session list
```

---

## ✅ Verificación

| Prueba | Comando / Acción | Resultado esperado |
|--------|-------------------|---------------------|
| DHCP en Windows10-1 | `ipconfig` | IP en rango `10.13.67.2–126`, gateway `10.13.67.1` |
| Conectividad usuarios | `ping 8.8.8.8` / `ping google.com` | Respuesta exitosa |
| Conectividad servidores | Desde WEB-1: `ping 8.8.8.8` | Respuesta exitosa |
| NAT activo | `Log & Report > Forward Traffic` | Sesiones saliendo por `port1` |
| Traducción de IP visible | `Forward Traffic > Details` de una sesión | Source IP (LAN) distinta a NAT'd Source IP (WAN) |
| Ruta por defecto | `get router info routing-table all` | `0.0.0.0/0` vía `port1` |
