# 🖧 Configuración de Router y Switches

![Cisco IOS](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=flat-square&logo=cisco&logoColor=white)
![Router](https://img.shields.io/badge/Router1-Sin_NAT-orange?style=flat-square)
![VLAN10](https://img.shields.io/badge/VLAN-10_Usuarios-blueviolet?style=flat-square)
![VLAN20](https://img.shields.io/badge/VLAN-20_Servidores-9cf?style=flat-square)
![Status](https://img.shields.io/badge/Estado-Funcional-success?style=flat-square)

Configuración completa de **Router1** (enrutamiento puro, sin NAT — el NAT lo realiza el FortiGate) y de **Switch1 / Switch2**, cada uno dedicado a una VLAN de acceso.

## 📌 Topología de referencia

```
Cloud1 (ISP) —— e0/1 [Router1] e0/0 —— Port1 [FortiGate] Port2 —— e0/0 [Switch1] e0/1 —— Windows10-1
                                                          Port3 —— e0/0 [Switch2] e0/1 —— WEB-1
```

| Dispositivo | Interfaz | Red / VLAN            |
|-------------|----------|------------------------|
| Router1     | e0/1     | 10.10.10.0/24 (hacia ISP) |
| Router1     | e0/0     | 200.13.67.0/30 (hacia FortiGate) |
| Switch1     | e0/0, e0/1 | VLAN 10 — Usuarios   |
| Switch2     | e0/0, e0/1 | VLAN 20 — Servidores |

![Topología completa](./images/topologia-router-switches.png)

---

## 📑 Contenido

1. [Router1](#1-router1)
2. [Switch1 — VLAN 10 (Usuarios)](#2-switch1--vlan-10-usuarios)
3. [Switch2 — VLAN 20 (Servidores)](#3-switch2--vlan-20-servidores)

---

## 1. Router1

> ⚠️ **Importante:** Router1 solo enruta. No se configura `ip nat inside`, `ip nat outside` ni `ip nat inside source` — el NAT lo hace el FortiGate.

```bash
enable
configure terminal
hostname Router1

! Interfaz hacia el ISP (Cloud1 / PC host 10.10.10.1)
interface e0/1
 ip address 10.10.10.2 255.255.255.0
 no shutdown
exit

! Interfaz hacia el FortiGate (Port1)
interface e0/0
 ip address 200.13.67.1 255.255.255.252
 no shutdown
exit

! Rutas hacia las LAN internas (detrás del FortiGate)
ip route 10.13.67.0 255.255.255.128 200.13.67.2
ip route 20.13.67.0 255.255.255.240 200.13.67.2

! Ruta por defecto hacia el ISP
ip route 0.0.0.0 0.0.0.0 10.10.10.1

end
write memory
```

**Verificación:**
```bash
show ip route
ping 200.13.67.2
```

![Router1 CLI](./images/router1-cli.png)

---

## 2. Switch1 — VLAN 10 (Usuarios)

```bash
enable
configure terminal
hostname Switch1

! Crear VLAN 10
vlan 10
 name USUARIOS
exit

! Puerto hacia FortiGate (Port2)
interface e0/0
 switchport mode access
 switchport access vlan 10
exit

! Puerto hacia Windows10-1
interface e0/1
 switchport mode access
 switchport access vlan 10
exit

end
write memory
```

**Verificación:**
```bash
show vlan brief
show interfaces status
```

![Switch1 CLI](./images/switch1-cli.png)

---

## 3. Switch2 — VLAN 20 (Servidores)

```bash
enable
configure terminal
hostname Switch2

! Crear VLAN 20
vlan 20
 name SERVIDORES
exit

! Puerto hacia FortiGate (Port3)
interface e0/0
 switchport mode access
 switchport access vlan 20
exit

! Puerto hacia WEB-1
interface e0/1
 switchport mode access
 switchport access vlan 20
exit

end
write memory
```

**Verificación:**
```bash
show vlan brief
show interfaces status
```

![Switch2 CLI](./images/switch2-cli.png)

---

## ✅ Resumen de verificación

| Dispositivo | Comando | Resultado esperado |
|-------------|---------|---------------------|
| Router1 | `show run \| include nat` | Sin salida (ningún `ip nat` configurado) |
| Router1 | `show ip route` | Rutas a `10.13.67.0/25` y `20.13.67.0/28` vía `200.13.67.2` |
| Switch1 | `show vlan brief` | e0/0 y e0/1 en VLAN 10 |
| Switch2 | `show vlan brief` | e0/0 y e0/1 en VLAN 20 |
