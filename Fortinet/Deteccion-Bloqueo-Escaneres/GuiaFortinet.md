# 🛡️ Detección y Bloqueo de Escáneres de Red — FortiGate

![FortiGate](https://img.shields.io/badge/FortiGate-Firewall-red?style=flat-square&logo=fortinet&logoColor=white)
![GUI](https://img.shields.io/badge/GUI-Incluido-blue?style=flat-square)
![CLI](https://img.shields.io/badge/CLI-Incluido-brightgreen?style=flat-square)
![Status](https://img.shields.io/badge/Estado-Funcional-success?style=flat-square)
![Autor](https://img.shields.io/badge/Autor-Miguel_Ramirez_Meli-informational?style=flat-square)
![Matricula](https://img.shields.io/badge/Matr%C3%ADcula-20251367-informational?style=flat-square)
![Docente](https://img.shields.io/badge/Docente-Jonathan_Rond%C3%B3n-lightgrey?style=flat-square)
![Materia](https://img.shields.io/badge/Materia-Seguridad_de_Redes-yellow?style=flat-square)

## 📑 Contenido

1. [DoS Policy (anomalías de escaneo)](#1-dos-policy-anomalías-de-escaneo)
2. [IPS Sensor (firmas de tipo Anomaly)](#2-ips-sensor-firmas-de-tipo-anomaly)
3. [Aplicar IPS a la política de tráfico](#3-aplicar-ips-a-la-política-de-tráfico)
4. [Quarantine (bloqueo total de la IP atacante)](#4-quarantine-bloqueo-total-de-la-ip-atacante)

---

## 1. DoS Policy (anomalías de escaneo)

### 🖱️ GUI

1. `Policy & Objects > DoS Policy > Create New`
2. Name: `Deteccion-Escaneo-Usuarios`
3. Incoming Interface: `port2` (Lan-Users)
4. Source/Destination: `all`
5. Service: `ALL`
6. Activar y configurar en **L4 Anomalies**:

| Anomalía | Action | Threshold |
|----------|--------|-----------|
| `tcp_port_scan` | Block | 3 |
| `tcp_src_session` | Block | 5000 |
| `tcp_dst_session` | Block | 5000 |
| `udp_scan` | Block | 1000 |
| `icmp_sweep` | Block | 20 |

7. Logging: habilitado en cada anomalía activada
8. `OK`

> ℹ️ El threshold de `tcp_port_scan` se dejó en `3` (detección casi inmediata, apenas 3 puertos distintos) y `icmp_sweep` en `20`, para forzar la detección en un entorno de laboratorio con poco tráfico de fondo. En producción se ajustan más altos según el tráfico normal de la red.

### ⌨️ CLI

```bash
config firewall DoS-policy
    edit 0
        set interface "port2"
        set srcaddr "all"
        set dstaddr "all"
        set service "ALL"
        config anomaly
            edit "tcp_port_scan"
                set status enable
                set action block
                set log enable
                set threshold 3
            next
            edit "tcp_src_session"
                set status enable
                set action block
                set log enable
                set threshold 5000
            next
            edit "tcp_dst_session"
                set status enable
                set action block
                set log enable
                set threshold 5000
            next
            edit "udp_scan"
                set status enable
                set action block
                set log enable
                set threshold 1000
            next
            edit "icmp_sweep"
                set status enable
                set action block
                set log enable
                set threshold 20
            next
        end
    next
end
```

---

## 2. IPS Sensor (firmas de tipo Anomaly)

### 🖱️ GUI

1. `Security Profiles > Intrusion Prevention > Create New`
2. Name: `IPS-Deteccion-Scanners`
3. `IPS Signatures and Filters > Create New`
4. Type: `Filter`
5. Filter: `+` → **Vulnerability Type** → `Anomaly`
6. Action: `Block`
7. Packet logging: `Enable`
8. Status: `Enable`
9. Aplicar el filtro (queda como grupo de firmas en la tabla)
10. `OK` al sensor

### ⌨️ CLI

```bash
config ips sensor
    edit "IPS-Deteccion-Scanners"
        config entries
            edit 1
                set status enable
                set log enable
                set action block
                set vuln-type anomaly
            next
        end
    next
end
```

---

## 3. Aplicar IPS a la política de tráfico

### 🖱️ GUI

1. `Policy & Objects > Firewall Policy`
2. Edita `LAN-Usuarios-to-WAN`
3. Security Profiles → `IPS`: activa y selecciona `IPS-Deteccion-Scanners`
4. `OK`

### ⌨️ CLI

```bash
config firewall policy
    edit <ID-LAN-Usuarios-to-WAN>
        set ips-sensor "IPS-Deteccion-Scanners"
    next
end
```

---

## 4. Quarantine (bloqueo total de la IP atacante)

Sin esto, el IPS solo corta la sesión puntual que disparó la alerta (`clear_session`) — el atacante puede seguir probando otros puertos/destinos de inmediato. Con Quarantine, en cuanto se detecta el patrón, se banea la IP origen completa por un tiempo, bloqueando todo su tráfico sin importar el destino.

### 🖱️ GUI

1. `Security Profiles > Intrusion Prevention > IPS-Deteccion-Scanners`
2. Edita el filtro `Anomaly` ya creado
3. Campo **Quarantine** → `Attacker`
4. Duration → `5m`
5. Guarda el filtro y `OK` al sensor

### ⌨️ CLI

```bash
config ips sensor
    edit "IPS-Deteccion-Scanners"
        config entries
            edit 1
                set status enable
                set log enable
                set action block
                set vuln-type 12
                set quarantine attacker
                set quarantine-expiry 5m
                set quarantine-log enable
            next
        end
    next
end
```

Ver la IP en cuarentena en tiempo real:

```bash
diagnose user quarantine list
```

---

## ✅ Verificación

Desde Kali (o cualquier host con Nmap), escaneo agresivo que dispare el threshold:

```bash
nmap -sS -Pn -p 1-65535 --min-rate 5000 -T5 20.13.67.10
```

| Prueba | Dónde revisar | Resultado esperado |
|--------|----------------|---------------------|
| Detección de port scan | `Log & Report > Security Events > Logs`, filtro `Anomaly` | Evento `tcp_port_scan`, Severity `Critical`, Action `clear_session` |
| Origen identificado | Columna `Source` en el mismo log | IP real del atacante (Kali) |
| Superficie de ataque reducida | Resultado del propio `nmap` | Solo puertos permitidos por política (ej. 80/tcp) aparecen abiertos, el resto `filtered` |
| Bloqueo total por Quarantine | Repetir `nmap` contra otro destino distinto, sin esperar | Todos los puertos aparecen `filtered` — la IP completa quedó bloqueada, no solo el destino escaneado |
| Cuarentena activa | `diagnose user quarantine list` | IP de Kali listada con tiempo restante |

> Nota: si el escaneo intenta primero un host-discovery (ping/TCP/HTTPS probe) y la política de tráfico ya bloquea todo excepto el servicio permitido, Nmap puede reportar "Host seems down" — usar `-Pn` para saltar esa verificación y forzar el escaneo real de puertos.
