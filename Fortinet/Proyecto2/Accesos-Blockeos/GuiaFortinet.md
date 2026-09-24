# 🛡️ Políticas de Firewall: Control de Acceso entre VLANs

![FortiGate](https://img.shields.io/badge/FortiGate-VM64-red?logo=fortinet&style=for-the-badge)
![Security](https://img.shields.io/badge/Security-Firewall_Policies-green?style=for-the-badge)
![VLAN](https://img.shields.io/badge/VLAN-10_to_20-blue?style=for-the-badge)
![GitHub](https://img.shields.io/badge/GitHub-Repo-black?style=for-the-badge)

## 📋 Descripción
Este documento detalla la configuración de dos políticas de firewall en el FortiGate para controlar y segmentar el tráfico entre la **VLAN 10 (Usuarios)** y la **VLAN 20 (Servidores/DMZ)**:

- ✅ **Política 1:** Permitir acceso HTTPS (puerto 443) desde Usuarios hacia el ServidorWeb.
- ❌ **Política 2:** Bloquear acceso MySQL (puerto 3306) desde Usuarios hacia la BaseDeDatos.

---

##  POLÍTICA 1: Permitir Usuarios → ServidorWeb (HTTPS / 443)

### 🖥️ Configuración mediante GUI

1. Ve a **Policy & Objects** > **Firewall Policy**.
2. Haz clic en **Create New**.
3. Configura los campos así:

| Campo | Valor |
| :--- | :--- |
| **Name** | `Usuarios_to_WebServer_HTTPS` |
| **Incoming Interface** | `LAN-USUARIOS` (port2) |
| **Outgoing Interface** | `LAN-SERVIDORES` (port3) |
| **Source** | `all` |
| **Destination** | `all` |
| **Schedule** | `always` |
| **Service** | `HTTPS` (puerto 443). ⚠️ No selecciones `ALL`. |
| **Action** | `ACCEPT` |
| **NAT** | `DESACTIVADO` (Switch en OFF) |

4. Haz clic en **OK**.

### 💻 Configuración mediante CLI

> **Nota:** Ajusta el número del `edit` (en este ejemplo es `3`) según el siguiente ID disponible en tu lista de políticas.

```bash
config firewall policy
    edit 3
        set name "Usuarios_to_WebServer_HTTPS"
        set srcintf "port2"
        set dstintf "port3"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "HTTPS"
        set nat disable
    next
end
```

---

## 🔴 POLÍTICA 2: Bloquear Usuarios → BaseDeDatos (MySQL / 3306)

### 🖥️ Configuración mediante GUI

1. Ve a **Policy & Objects** > **Firewall Policy**.
2. Haz clic en **Create New**.
3. Configura los campos así:

| Campo | Valor |
| :--- | :--- |
| **Name** | `Block_Usuarios_to_DB_MySQL` |
| **Incoming Interface** | `LAN-USUARIOS` (port2) |
| **Outgoing Interface** | `LAN-SERVIDORES` (port3) |
| **Source** | `all` |
| **Destination** | `all` |
| **Schedule** | `always` |
| **Service** | `MYSQL` (puerto 3306). ⚠️ No selecciones `ALL`. |
| **Action** | `DENY` |
| **NAT** | `DESACTIVADO` (Switch en OFF) |
| **Log Traffic** | `ENABLED` (Recomendado para auditar intentos de acceso) |

4. Haz clic en **OK**.

### 💻 Configuración mediante CLI

> **Nota:** Ajusta el número del `edit` (en este ejemplo es `4`) según el siguiente ID disponible en tu lista de políticas.

```bash
config firewall policy
    edit 4
        set name "Block_Usuarios_to_DB_MySQL"
        set srcintf "port2"
        set dstintf "port3"
        set srcaddr "all"
        set dstaddr "all"
        set action deny
        set schedule "always"
        set service "MYSQL"
        set logtraffic all
        set nat disable
    next
end
```

---

## ⚠️ IMPORTANTE: Orden de las Políticas

El FortiGate evalúa las políticas de **arriba hacia abajo** y se detiene en la primera coincidencia. 

Si existe una política previa que permita tráfico `ALL` entre el `port2` y el `port3`, la política de bloqueo (DENY) **no funcionará** si está ubicada debajo de ella.

**Solución:** En la GUI (**Policy & Objects > Firewall Policy**), arrastra la política `Block_Usuarios_to_DB_MySQL` para que quede ubicada **por encima** de cualquier política permisiva general.

---

##  Pruebas de Conectividad

### Prueba 1: Acceso HTTPS al ServidorWeb (Debe funcionar ✅)
Desde un equipo en la VLAN de Usuarios, abre un navegador o usa la terminal:
```bash
curl -k https://20.13.67.2
```

### Prueba 2: Acceso MySQL a la BaseDeDatos (Debe ser bloqueado ❌)
Desde un equipo en la VLAN de Usuarios, intenta conectarte al puerto:
```bash
telnet 20.13.67.3 3306
```
*El resultado debe ser conexión rechazada o timeout.*

---

## 📊 Resumen de Políticas Configuradas

| # | Nombre de Política | Origen (Src) | Destino (Dst) | Servicio | Acción | NAT |
| :---: | :--- | :--- | :--- | :--- | :---: | :---: |
| 1 | `Usuarios_to_WebServer_HTTPS` | VLAN 10 (port2) | ServidorWeb (20.13.67.2) | HTTPS (443) | ✅ ACCEPT | OFF |
| 2 | `Block_Usuarios_to_DB_MySQL` | VLAN 10 (port2) | BaseDeDatos (20.13.67.3) | MYSQL (3306) | ❌ DENY | OFF |

---

<br>

### 👨‍💻 Realizado por: **Miguel Ramirez Meli**

![Author](https://img.shields.io/badge/Author-Miguel_Ramirez_Meli-orange?style=for-the-badge)
![Lab](https://img.shields.io/badge/Lab-FortiGate_VM64-purple?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Policies_Configured-brightgreen?style=for-the-badge)
