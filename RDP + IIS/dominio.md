# 🏛️ Lab: Creación del Dominio `miguel.local`

![Estado](https://img.shields.io/badge/Estado-Completado-brightgreen)
![Resultado](https://img.shields.io/badge/Resultado-Dominio%20Funcional-brightgreen)
![Modelo](https://img.shields.io/badge/Modelo-VLANs%20Segmentadas%20%2F24-blue)
![Simulador](https://img.shields.io/badge/Simulador-GNS3-blue)
![Dispositivos](https://img.shields.io/badge/Dispositivos-Router1%20%7C%20Switch1%20%7C%20Switch2%20%7C%20Win10--1%20%7C%20WinServer2022--1-lightgrey)
![Dominio](https://img.shields.io/badge/Dominio-miguel.local-orange)

---

## 🌐 Topología

![Topología de red](images/topologia.png)

| Dispositivo | Interfaz | IP | VLAN | Rol |
|---|---|---|---|---|
| Router1 | e0/2 → Cloud1 (tap0) | 10.10.10.2/24 | - | NAT/Gateway |
| Router1 | e0/0 → Switch1 | 10.13.67.1/24 | 10 | Gateway VLAN10 |
| Router1 | e0/1 → Switch2 | 20.13.67.1/24 | 20 | Gateway VLAN20 |
| Windows10-1 | NIC1 (DHCP) | 10.13.67.x/24 | 10 | Cliente del dominio |
| WindowsServer2022-1 | NIC1 (estática) | 20.13.67.10/24 | 20 | DC / RDS |

---

## ⚙️ Configuración de dispositivos

### Router1

```
enable
configure terminal
hostname Router1

interface e0/2
 ip address 10.10.10.2 255.255.255.0
 ip nat outside
 no shutdown

interface e0/0
 ip address 10.13.67.1 255.255.255.0
 ip nat inside
 no shutdown

interface e0/1
 ip address 20.13.67.1 255.255.255.0
 ip nat inside
 no shutdown

ip route 0.0.0.0 0.0.0.0 10.10.10.1

access-list 1 permit 10.13.67.0 0.0.0.255
access-list 1 permit 20.13.67.0 0.0.0.255
ip nat inside source list 1 interface e0/2 overload

ip dhcp excluded-address 10.13.67.1
ip dhcp pool VLAN10-POOL
 network 10.13.67.0 255.255.255.0
 default-router 10.13.67.1
 dns-server 20.13.67.10
 domain-name miguel.local

end
write memory
```

### Switch1 (VLAN 10 — Windows10-1)

```
enable
configure terminal
hostname Switch1

vlan 10
 name VLAN_WINDOWS10
exit

interface e0/0
 switchport mode access
 switchport access vlan 10
 no shutdown

interface e0/1
 switchport mode access
 switchport access vlan 10
 no shutdown

end
write memory
```

### Switch2 (VLAN 20 — WindowsServer2022-1)

```
enable
configure terminal
hostname Switch2

vlan 20
 name VLAN_SERVER
exit

interface e0/0
 switchport mode access
 switchport access vlan 20
 no shutdown

interface e0/1
 switchport mode access
 switchport access vlan 20
 no shutdown

end
write memory
```

### Host físico (NAT hacia Internet — Linux)

```bash
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o <tu_interfaz_real> -j MASQUERADE
```

> Si tu host es Windows, habilita "Compartir conexión a Internet" (ICS) desde tu adaptador real hacia el adaptador usado por Cloud1.

---

## 🏛️ Creación del dominio `miguel.local` (con clicks)

### WindowsServer2022-1 — IP estática

1. Panel de control → Centro de redes y recursos compartidos → **Cambiar configuración del adaptador**
2. Click derecho en la NIC → **Propiedades**
3. Selecciona **Protocolo de Internet versión 4 (TCP/IPv4)** → **Propiedades**
4. ⭕ **Usar la siguiente dirección IP**:
   - IP: `20.13.67.10` · Máscara: `255.255.255.0` · Gateway: `20.13.67.1`
5. ⭕ **Usar las siguientes direcciones de servidor DNS**: `127.0.0.1`
6. **Aceptar** en ambas ventanas

### Instalar rol AD DS

1. **Administrador del servidor → Administrar → Agregar roles y características**
2. Siguiente → ⭕ Instalación basada en características o roles → Siguiente
3. Seleccionar servidor → Siguiente
4. ✅ **Servicios de dominio de Active Directory** → Agregar características → Siguiente
5. Siguiente (Características) → Siguiente (pantalla informativa) → **Instalar**
6. Esperar → **Cerrar** (no reinicies)

### Promover a Controlador de Dominio

1. En Server Manager aparece una bandera 🚩 amarilla → click en ella
2. Click **"Promover este servidor a controlador de dominio"**
3. ⭕ **Agregar un nuevo bosque** → Nombre del dominio raíz: `miguel.local` → Siguiente
4. Nivel funcional: deja el predeterminado
5. ✅ DNS Server (ya viene marcado) → Contraseña DSRM: `TuPasswordSeguro123!` → Siguiente
6. Siguiente (delegación DNS, ignora la advertencia) → Siguiente (nombre NetBIOS: `MIGUEL`)
7. Siguiente (rutas de base de datos, dejar predeterminadas)
8. Revisar opciones → Siguiente → **Verificar requisitos previos** → **Instalar**
9. El servidor reinicia solo

### Windows10-1 — unir al dominio

1. `ipconfig /release` → `ipconfig /renew` (confirma que reciba IP `10.13.67.x` y DNS `20.13.67.10`)
2. Panel de control → Sistema → **Cambiar configuración** → pestaña **Nombre de equipo** → **Cambiar...**
3. ⭕ **Dominio**: escribe `miguel.local` → Aceptar
4. Ingresa credenciales de administrador del dominio → Aceptar
5. Mensaje de bienvenida al dominio → **Aceptar** → **Reiniciar ahora**

### Crear usuario de dominio (Ronald Ramirez)

**Server Manager → Herramientas → Usuarios y equipos de Active Directory**

1. Click derecho en `miguel.local` (o una OU) → **Nuevo → Usuario**
2. Nombre: `Ronald` · Apellidos: `Ramirez` · Nombre de inicio de sesión: `ronaldrm` → Siguiente
3. Contraseña: `Segura2121...` → confirmar
4. Desmarca ✅ **"El usuario debe cambiar la contraseña en el siguiente inicio de sesión"**
5. Siguiente → **Finalizar**

---

## ✅ Checklist rápido

- [x] Router1 / Switch1 / Switch2 configurados
- [x] WindowsServer2022-1 promovido a DC (`miguel.local`)
- [x] Windows10-1 unido al dominio
- [x] Usuario `ronaldrm` creado
