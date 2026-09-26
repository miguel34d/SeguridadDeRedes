#  Acceso a Puertos: Política de Firewall para SSH

![FortiGate](https://img.shields.io/badge/FortiGate-VM64-red?logo=fortinet&style=for-the-badge)
![Security](https://img.shields.io/badge/Security-Firewall_Policy-green?style=for-the-badge)
![SSH](https://img.shields.io/badge/Service-SSH-blue?style=for-the-badge)
![GitHub](https://img.shields.io/badge/GitHub-Repo-black?style=for-the-badge)

##  Descripción
Este documento detalla la configuración de la política de firewall en el FortiGate que permite el acceso SSH desde la PC Host (Cloud/ISP - `10.10.10.1`) hacia los servidores ubicados en la red DMZ (`20.13.67.0/29`) a través del puerto 22.

---

## ️ Configuración mediante GUI

1. Ve a **Policy & Objects** > **Firewall Policy**.
2. Haz clic en **Create New**.
3. Configura los campos así:

| Campo | Valor |
| :--- | :--- |
| **Name** | `PC_to_Servers_SSH` |
| **Incoming Interface** | `WAN-ISP` (port1) |
| **Outgoing Interface** | `LAN-SERVIDORES` (port3) |
| **Source** | `all` |
| **Destination** | `all` |
| **Schedule** | `always` |
| **Service** | `SSH` (puerto 22). ⚠️ No selecciones `ALL` por seguridad. |
| **Action** | `ACCEPT` |
| **NAT** | `DESACTIVADO` (Switch en OFF) |

4. Haz clic en **OK**.

---

##  Configuración mediante CLI

```bash
config firewall policy
    edit 2
        set name "PC_to_Servers_SSH"
        set srcintf "port1"
        set dstintf "port3"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "SSH"
        set nat disable
    next
end
```

---

## ✅ Verificación

Para verificar que la política fue creada correctamente:

**Desde la GUI:**
- Ve a **Policy & Objects** > **Firewall Policy** y confirma que la política `PC_to_Servers_SSH` aparece en la lista con el estado habilitado.

**Desde la CLI:**
```bash
show firewall policy
```

Deberías ver la política con `set service "SSH"` y `set nat disable`.

---

## 🧪 Prueba de Conectividad

Desde tu PC Host, ejecuta:

```bash
ssh miguel@20.13.67.2
```

Si la política está correcta y el servicio SSH está activo en el servidor, deberías obtener acceso remoto.

---

<br>

### 👨‍💻 Realizado por: **Miguel Ramirez Meli**

![Author](https://img.shields.io/badge/Author-Miguel_Ramirez_Meli-orange?style=for-the-badge)
![Lab](https://img.shields.io/badge/Lab-FortiGate_VM64-purple?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)
