# Monitoreo de Red con SNMP y Zabbix — Demostración

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

## 1. Topología

Router1 conectado a Switch1, con Windows10-1 como cliente DHCP y Zabbix-1 como servidor de monitoreo.

![Topología](assets/01_topologia.png)

| Equipo | Interfaz | IP |
|---|---|---|
| Router1 | f0/0 | `10.13.67.1/24` |
| Switch1 | VLAN1 | `10.13.67.2/24` |
| Zabbix-1 | eth0 | DHCP |
| Windows10-1 | NIC1 | DHCP |

Comunidad SNMP de solo lectura: **`20251367`**

---

## 2. Direccionamiento IP y Comunidad SNMP en Router1 y Switch1

Configuración aplicada en ambos dispositivos:

![Configuración SNMP](assets/04_snmp_config_dispositivos.png)

Verificación del servicio SNMP activo en Switch1, con la comunidad `20251367` registrada como `active` y estadísticas reales de paquetes SNMP procesados:

![Show SNMP Switch1](assets/05_show_snmp_switch1.png)

---

## 3. Servidor DHCP para la red LAN — Cliente en Windows10-1

Windows10-1 recibiendo dirección IP automáticamente desde el pool DHCP configurado en Router1:

![DHCP Windows10-1](assets/02_dhcp_windows.png)

---

## 4. Verificación de datos SNMP desde el PC Cliente

Consulta SNMP (Walk) realizada desde iReasoning MIB Browser hacia Router1, mostrando datos reales como `sysDescr`, `sysContact`, `sysLocation`, `sysName` y `sysUpTime`:

![MIB Browser Router1](assets/03_mib_browser_router1.png)

---

## 5. Configuración del Servidor Zabbix

Acceso al frontend web de Zabbix:

![Login Zabbix](assets/06_zabbix_login.png)

### 5.1 Creación del host Router1

Host `Router1` configurado con su grupo y template `Network Generic Device by SNMP`:

![Host Router1 creado](assets/07_host_router1_creado.png)

### 5.2 Interfaz SNMP configurada

Interfaz de tipo SNMP asignada a Router1, con versión `SNMPv2` y comunidad `20251367`:

![Interfaz SNMP Router1](assets/08_interfaz_snmp_router1.png)

---

## 6. Datos en tiempo real (Monitoring → Latest data)

### 6.1 Switch1 — ítems recolectados por SNMP

Zabbix recolectando datos activos del Switch1 (165 ítems con datos, incluyendo interfaces, componentes de red y sistema):

![Latest data Switch1](assets/09_host_switch1_latestdata.png)

### 6.2 Router1 — ítems recolectados por SNMP

Zabbix recolectando datos activos del Router1 (22 ítems con datos, interfaz Fa0/0 y variables de sistema):

![Latest data Router1](assets/10_latestdata_router1.png)

---

## 7. Verificación de eventos del Router y el Switch (Monitoring → Problems)

Eventos reales generados y detectados por Zabbix para ambos dispositivos:

![Eventos Router1 y Switch1](assets/11_problems_eventos.png)

- `Router1: has been restarted (uptime < 10m)`
- `Switch1: has been restarted (uptime < 10m)`
- `Router1: Interface Fa0/0(): In half-duplex mode`

---

## 8. Resumen de cumplimiento

| Requisito | Estado |
|---|---|
| Direccionamiento IP en Router y Switch | ✅ |
| Servidor DHCP para la red LAN | ✅ |
| Comunidad SNMP de solo lectura | ✅ |
| Cliente DHCP en PC | ✅ |
| Verificación de datos SNMP desde el PC | ✅ |
| Servidor Zabbix configurado | ✅ |
| Verificación de eventos del Router y Switch en Zabbix | ✅ |
