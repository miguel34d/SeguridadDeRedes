# 🛡️ Implementación de Wazuh en Red Segmentada (Admins / Users)

![Wazuh](https://img.shields.io/badge/Wazuh-4.14-3AA5D5?style=for-the-badge&logo=wazuh&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04_LTS-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Cisco](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Status](https://img.shields.io/badge/Status-Producci%C3%B3n-success?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)
![Autor](https://img.shields.io/badge/Autor-Miguel_Ramirez_Meli-blueviolet?style=for-the-badge)

---

## 📌 Descripción

Guía de implementación de **Wazuh SIEM** sobre una red segmentada en dos VLANs (`Admins` y `Users`), con control de acceso mediante ACL para que **únicamente el equipo ADMINISTRADOR** pueda acceder al portal/API de Wazuh.

## 🗺️ Topología

```
Cloud1 (10.10.10.1)
   │
Router1 (10.10.10.2)
   ├── Gi0/1 → Switch1 (VLAN 10 - Admins) → ADMINISTRADOR / WAZUH
   └── Gi0/2 → Switch2 (VLAN 20 - Users)  → PC-1 / PC-2
```

## 📋 Direccionamiento IP

| Dispositivo | SO | Interfaz | IP | Máscara | VLAN |
|---|---|---|---|---|---|
| Cloud1 | - | ISP | 10.10.10.1 | /24 | - |
| Router1 | Cisco IOS | WAN | 10.10.10.2 | /24 | - |
| Router1 | Cisco IOS | Admins | 10.13.67.1 | /24 | 10 |
| Router1 | Cisco IOS | Users | 20.13.67.1 | /24 | 20 |
| ADMINISTRADOR | Windows | PC | 10.13.67.2 | /24 | 10 |
| **WAZUH (servidor)** | Ubuntu Server | Server | **10.13.67.10** | /24 | 10 |
| PC-1 | Windows | PC | 20.13.67.2 | /24 | 20 |
| PC-2 | Kali Linux | PC | 20.13.67.3 | /24 | 20 |

## ✅ Requisitos previos

- Nodo `WAZUH` agregado a la zona **Admins**, conectado a `Switch1`.
- Acceso administrativo a `Router1`, `Switch1`, `Switch2`.
- Server con Ubuntu 22.04 LTS, mínimo 4 vCPU / 8 GB RAM / 50 GB disco.
- Salida a internet desde el server para descargar paquetes de Wazuh.

## 1️⃣ Configuración de Router1

```
enable
configure terminal
hostname Router1

interface Ethernet0/0
 description WAN-Cloud1
 ip address 10.10.10.2 255.255.255.0
 no shutdown

interface Ethernet0/1
 description LAN-Admins-Switch1
 ip address 10.13.67.1 255.255.255.0
 no shutdown

interface Ethernet0/2
 description LAN-Users-Switch2
 ip address 20.13.67.1 255.255.255.0
 no shutdown

ip route 0.0.0.0 0.0.0.0 10.10.10.1

ip access-list extended ACL_WAZUH
 permit tcp host 10.13.67.2 host 10.13.67.10 eq 443
 permit tcp host 10.13.67.2 host 10.13.67.10 eq 55000
 permit tcp host 10.13.67.2 host 10.13.67.10 eq 1514
 permit tcp host 10.13.67.2 host 10.13.67.10 eq 1515
 deny tcp any host 10.13.67.10 eq 443
 deny tcp any host 10.13.67.10 eq 55000
 permit ip any any

interface Ethernet0/1
 ip access-group ACL_WAZUH out

end
write memory
```

## 2️⃣ Configuración de Switch1 (Admins - VLAN 10)

```
enable
configure terminal
hostname Switch1
vlan 10
 name ADMINS
exit

interface Ethernet0/1
 description ADMINISTRADOR
 switchport mode access
 switchport access vlan 10

interface Ethernet0/2
 description WAZUH-SERVER
 switchport mode access
 switchport access vlan 10

interface Ethernet0/0
 description Uplink-Router1
 switchport mode access
 switchport access vlan 10

end
write memory
```

## 3️⃣ Configuración de Switch2 (Users - VLAN 20)

```
enable
configure terminal
hostname Switch2
vlan 20
 name USERS
exit

interface Ethernet0/1
 description PC-2
 switchport mode access
 switchport access vlan 20

interface Ethernet0/2
 description PC-1
 switchport mode access
 switchport access vlan 20

interface Ethernet0/0
 description Uplink-Router1
 switchport mode access
 switchport access vlan 20

end
write memory
```

## 4️⃣ IP estática permanente por host

Cada equipo tiene un SO distinto, así que el método de IP permanente cambia:

| Host | SO | Método |
|---|---|---|
| WAZUH | Ubuntu Server | `nmcli` — CLI obligatorio (no hay GUI) |
| PC-2 | Kali Linux | `nmcli` — CLI obligatorio (no hay GUI) |
| ADMINISTRADOR | Windows | GUI (Configuración de red) |
| PC-1 | Windows | GUI (Configuración de red) |

### 🔧 WAZUH (Ubuntu Server) — activar NetworkManager primero

Ubuntu Server usa **netplan + systemd-networkd** por defecto, por eso `nmcli` no existe hasta instalarlo y ponerlo como renderer:

```bash
sudo apt update
sudo apt install network-manager -y

# Decirle a netplan que use NetworkManager
sudo tee /etc/netplan/01-network-manager-all.yaml > /dev/null <<EOF
network:
  version: 2
  renderer: NetworkManager
EOF

sudo netplan apply
sudo systemctl enable NetworkManager --now
```

Ahora sí, IP estática permanente. Primero verifica el nombre real de la conexión activa (puede no llamarse igual que la interfaz):

```bash
nmcli con show
```

Con el nombre real (en este caso `ens3`), modifica esa misma conexión en vez de crear una nueva — así evitas perfiles duplicados/huérfanos:

```bash
sudo nmcli con mod ens3 ipv4.method manual
sudo nmcli con mod ens3 ipv4.addresses 10.13.67.10/24
sudo nmcli con mod ens3 ipv4.gateway 10.13.67.1
sudo nmcli con mod ens3 ipv4.dns "8.8.8.8 1.1.1.1"
sudo nmcli con mod ens3 ipv4.ignore-auto-dns yes
sudo nmcli con up ens3
```

### 🔧 PC-2 (Kali Linux)

Kali ya trae NetworkManager activo. Verifica primero el nombre real de la conexión:

```bash
nmcli con show
```

Con el nombre real (ajusta si no es `eth0`), modifica esa conexión (20.13.67.3/24, gw 20.13.67.1):

```bash
sudo nmcli con mod eth0 ipv4.method manual
sudo nmcli con mod eth0 ipv4.addresses 20.13.67.3/24
sudo nmcli con mod eth0 ipv4.gateway 20.13.67.1
sudo nmcli con mod eth0 ipv4.dns "8.8.8.8 1.1.1.1"
sudo nmcli con mod eth0 ipv4.ignore-auto-dns yes
sudo nmcli con up eth0
```

Verificación en WAZUH y PC-2:
```bash
nmcli con show
ip a
```

### 🖱️ ADMINISTRADOR (Windows) — vía GUI

1. Clic derecho en el ícono de red (barra de tareas) → **Abrir configuración de red e Internet**.
2. **Ethernet** → clic en el adaptador conectado → **Editar configuración IP**.
3. Cambiar de "Automática (DHCP)" a **Manual**.
4. Activar **IPv4** y llenar:
   - Dirección IP: `10.13.67.2`
   - Máscara de subred / prefijo: `24` (255.255.255.0)
   - Puerta de enlace: `10.13.67.1`
   - DNS preferido: `8.8.8.8` / DNS alterno: `1.1.1.1`
5. **Guardar**.

### 🖱️ PC-1 (Windows) — vía GUI

Mismos pasos, con estos datos:
   - Dirección IP: `20.13.67.2`
   - Máscara: `24` (255.255.255.0)
   - Puerta de enlace: `20.13.67.1`
   - DNS preferido: `8.8.8.8` / DNS alterno: `1.1.1.1`

> Nota: en Windows la IP queda guardada de forma permanente en cuanto se configura como "Manual" desde esta pantalla — no requiere ningún paso extra ni CLI.

## 5️⃣ Instalación de Wazuh (servidor 10.13.67.10)

```bash
# Descargar el instalador oficial (todo en uno: indexer + manager + dashboard)
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh

# Instalar
sudo bash wazuh-install.sh -a -i
```

Acceder al dashboard en `https://10.13.67.10` con el usuario `admin` y la contraseña generada.

## 6️⃣ Despliegue de agentes (ADMINISTRADOR, PC-1, PC-2)

El orden correcto es: **1) registrar el agente en el Wazuh manager (GUI) → 2) instalar el agente en cada equipo** con los datos que el propio dashboard genera.

### Paso 1 — Registrar el agente en Wazuh (GUI, en el dashboard `https://10.13.67.10`)

1. Ir a **Agents** (menú lateral) → botón **Deploy new agent**.
2. Seleccionar el **sistema operativo** del equipo a registrar (Linux/deb o Windows).
3. Confirmar que el campo **Wazuh server address** ya muestre `10.13.67.10`.
4. (Opcional) Asignar un **nombre de agente** y un **grupo** (p. ej. `admins` o `users`).
5. El dashboard genera automáticamente el **comando de instalación** con la clave de enrolamiento ya incluida — copiarlo.
6. Repetir este paso una vez por cada equipo (ADMINISTRADOR, PC-1, PC-2), ya que cada uno recibe su propio comando/clave.

### Paso 2 — Instalar el agente en cada equipo con el comando generado

**PC-2 (Kali Linux) — CLI, obligatorio:**

1. Copiar el comando que generó el dashboard en el Paso 1 y ejecutarlo:
```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.7-1_amd64.deb && sudo WAZUH_MANAGER='10.13.67.10' WAZUH_AGENT_NAME='PC2-Kali' dpkg -i ./wazuh-agent_4.14.7-1_amd64.deb
```
> ⚠️ Verifica que `WAZUH_MANAGER` sea **10.13.67.10** (la IP del servidor WAZUH). Si el dashboard genera otra IP (por ejemplo la propia IP de PC-2), corrígela manualmente antes de ejecutar el comando.

2. Habilitar e iniciar el servicio:
```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

**ADMINISTRADOR (Windows) — vía GUI:**
1. Descargar el instalador `.msi` desde el enlace que dio el dashboard en el Paso 1 (abrir en el navegador).
2. Ejecutar el `.msi` con doble clic → seguir el asistente (ya trae precargada la IP `10.13.67.10` y la clave de enrolamiento de ADMINISTRADOR).
3. Finalizar instalación → el servicio "Wazuh" queda activo automáticamente (verificar en **Servicios de Windows**).

**PC-1 (Windows) — vía GUI/PowerShell:**
1. Copiar el comando que generó el dashboard en el Paso 1 y ejecutarlo en PowerShell **como administrador**:
```powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.7-1.msi -OutFile $env:tmp\wazuh-agent; msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER='10.13.67.10' WAZUH_AGENT_NAME='PC1-WINDOWS'
```
> ⚠️ Verifica que `WAZUH_MANAGER` apunte a **10.13.67.10** (la IP del servidor WAZUH). Si el dashboard te genera otra IP (por ejemplo la propia IP de PC-1), corrígela manualmente antes de correr el comando — si no, el agente nunca conectará.

2. Iniciar el servicio del agente:
```powershell
NET START Wazuh
```
3. Verificar en **Servicios de Windows** que "Wazuh" quede en estado "En ejecución".

### Paso 3 — Confirmar en el dashboard

En **Agents**, el equipo debe pasar de estado "Pending" a **"Active"** una vez que reporta correctamente.

## 7️⃣ Verificación de la ACL

Desde **ADMINISTRADOR** (10.13.67.2) debe abrir sin problema:
```
https://10.13.67.10
```

Desde **PC-1** o **PC-2** (VLAN 20) el acceso al puerto 443/55000 debe ser rechazado por la ACL del router.

## 8️⃣ Simulación de ataque (para generar alerta roja)

Desde **PC-2 (Kali)**, que está en VLAN Users y bloqueado por la ACL para el puerto 443/55000, se ataca por SSH (puerto 22, no filtrado) al propio servidor **WAZUH** (10.13.67.10), que tiene el agente local monitoreándose a sí mismo:

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://10.13.67.10
```

Los múltiples intentos fallidos de login disparan las reglas de fuerza bruta SSH de Wazuh (nivel de severidad alto), visibles en rojo en el módulo **Security events** del dashboard — mientras que el acceso de PC-2 al propio portal (443) sigue bloqueado por la ACL.

## 🧾 Checklist final

- [ ] Nodo WAZUH agregado y con IP 10.13.67.10/24
- [ ] IPs estáticas permanentes aplicadas (nmcli en WAZUH/PC-2, netsh/PowerShell en ADMINISTRADOR/PC-1)
- [ ] Router1 con interfaces y ACL aplicados
- [ ] Switch1 y Switch2 con VLANs correctas
- [ ] Wazuh instalado y dashboard accesible
- [ ] Agente registrado desde el dashboard (Deploy new agent) antes de instalar en cada equipo
- [ ] Agentes reportando en ADMINISTRADOR, PC-1 y PC-2 (estado "Active")
- [ ] ACL verificada (solo ADMINISTRADOR entra al portal)
- [ ] Ataque simulado y alerta visible en rojo

---

**Autor:** Miguel Ramírez Meli
