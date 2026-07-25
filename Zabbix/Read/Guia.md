# Monitoreo de Red con SNMP y Zabbix — GNS3

![Estado](https://img.shields.io/badge/Estado-Completado-brightgreen)
![Resultado](https://img.shields.io/badge/Resultado-SNMP%20%2B%20Zabbix%20Funcional-brightgreen)
![Modelo](https://img.shields.io/badge/Modelo-Red%20Plana%20%2F24-blue)
![Simulador](https://img.shields.io/badge/Simulador-GNS3-informational)
![Dispositivos](https://img.shields.io/badge/Dispositivos-Router1%20%7C%20Switch1%20%7C%20Windows10--1%20%7C%20Zabbix--1-lightgrey)
![SNMP](https://img.shields.io/badge/SNMP-Comunidad%20RO-orange)

**Estudiante:** Miguel Ramirez Meli
**Matrícula:** 2025-1367
**Asignatura:** Seguridad de Redes
**Práctica:** Tarea Semana 11 — Monitoreo con SNMP y Zabbix

---

## Topología

```
Router1 (f0/0) ── Switch1 ── Windows10-1
                       │
                    Zabbix-1
```

| Equipo | Interfaz | IP |
|---|---|---|
| Router1 | f0/0 | `10.13.67.1/24` |
| Switch1 | VLAN1 | `10.13.67.2/24` |
| Zabbix-1 | eth0 | DHCP |
| Windows10-1 | NIC1 | DHCP |

Comunidad SNMP de solo lectura: **`20251367`**

---

## 1. Router1

Entra a la consola de Router1 y ejecuta:

```
enable
configure terminal
hostname Router1

interface f0/0
 ip address 10.13.67.1 255.255.255.0
 no shutdown
exit

ip dhcp excluded-address 10.13.67.1 10.13.67.3
ip dhcp pool LAN
 network 10.13.67.0 255.255.255.0
 default-router 10.13.67.1
 dns-server 8.8.8.8
exit

snmp-server community 20251367 RO
snmp-server location Laboratorio_GNS3
snmp-server contact Miguel_Ramirez_Meli

end
write memory
```

---

## 2. Switch1

Entra a la consola de Switch1 y ejecuta:

```
enable
configure terminal
hostname Switch1

interface vlan1
 ip address 10.13.67.2 255.255.255.0
 no shutdown
exit

ip default-gateway 10.13.67.1

snmp-server community 20251367 RO
snmp-server location Laboratorio_GNS3
snmp-server contact Miguel_Ramirez_Meli

end
write memory
```

---

## 3. Windows10-1

1. Clic derecho en el ícono de red → **Abrir configuración de red e Internet** → **Cambiar opciones del adaptador**
2. Clic derecho en el adaptador → **Propiedades** → **Protocolo de Internet versión 4 (TCP/IPv4)** → **Propiedades**
3. Selecciona **"Obtener una dirección IP automáticamente"** → **Aceptar**
4. Abre `cmd` y ejecuta:

```
ipconfig /release
ipconfig /renew
ipconfig /all
```

5. Instala **iReasoning MIB Browser**
6. En el campo **Address**, escribe `10.13.67.1`
7. Clic en **Advanced...** → coloca en **Read Community**: `20251367` → versión `V2` → puerto `161` → **OK**
8. En el campo **OID**, escribe `.1.3.6.1.2.1`
9. Cambia **Operations** a **Walk**
10. Clic en **Go**
11. Repite los pasos 6-10 cambiando el Address a `10.13.67.2`

---

## 4. Zabbix-1

1. Enciende la VM (usuario `root` / contraseña `zabbix`)
2. Verifica su IP con:
```bash
ip addr show eth0
```
3. Desde Windows10-1, abre el navegador y entra a:
```
http://<IP-de-Zabbix>/
```
4. Login:
   - Usuario: `Admin`
   - Contraseña: `zabbix`

### Crear host Router1
1. **Data collection → Hosts → Create host**
2. **Host name:** `Router1`
3. **Templates:** busca y selecciona `Network Generic Device by SNMP`
4. **Host groups:** escribe `Red GNS3` y créalo
5. En **Interfaces**, clic en **Add** → selecciona **SNMP**
6. **IP address:** `10.13.67.1`
7. **SNMP version:** `SNMPv2`
8. **SNMP community:** `20251367`
9. Clic en **Add** para guardar

### Crear host Switch1
1. **Data collection → Hosts → Create host**
2. **Host name:** `Switch1`
3. **Templates:** `Network Generic Device by SNMP`
4. **Host groups:** `Red GNS3` (selecciónalo, ya existe)
5. **Interfaces → Add → SNMP**
6. **IP address:** `10.13.67.2`
7. **SNMP version:** `SNMPv2`
8. **SNMP community:** `20251367`
9. Clic en **Add** para guardar

---

## Verificación

### Datos SNMP (MIB Browser)

| Campo | Router1 | Switch1 |
|---|---|---|
| sysDescr.0 | Cisco IOS Software | Cisco IOS Software |
| sysContact.0 | Miguel_Ramirez_Meli | Miguel_Ramirez_Meli |
| sysLocation.0 | Laboratorio_GNS3 | Laboratorio_GNS3 |
| sysName.0 | Router1 | Switch1 |

### Datos en Zabbix (Monitoring → Latest data)

Filtra por host `Router1` y `Switch1`, confirma valores en tiempo real:
- `SNMP agent availability` → `available (1)`
- `Interface Fa0/0(): Operational status` → `up (1)`
- `System description`, `System contact`, `System location`, `System name`
- `Uptime (network)`

### Eventos (Monitoring → Problems)

Filtra por host `Router1` y `Switch1`, confirma que aparecen eventos reales como:
- `Router1 has been restarted (uptime < 10m)`
- `Interface Fa0/0(): In half-duplex mode`
- `Switch1 has been restarted (uptime < 10m)`
- `Interface Et0/3(): Link down` → `RESOLVED`

---

## Resumen de cumplimiento

| Requisito | Estado |
|---|---|
| Direccionamiento IP en Router y Switch | ✅ |
| Servidor DHCP para la red LAN | ✅ |
| Comunidad SNMP de solo lectura | ✅ |
| Cliente DHCP en PC | ✅ |
| Verificación de datos SNMP | ✅ |
| Servidor Zabbix configurado | ✅ |
| Verificación de eventos del Router y Switch | ✅ |
