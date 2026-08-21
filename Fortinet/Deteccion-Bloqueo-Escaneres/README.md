# 🛡️ Detección y Bloqueo de Escáneres de Red — Demostración

![Status](https://img.shields.io/badge/Estado-Funcional-success?style=flat-square)
![Autor](https://img.shields.io/badge/Autor-Miguel_Ramirez_Meli-informational?style=flat-square)
![Matricula](https://img.shields.io/badge/Matr%C3%ADcula-20251367-informational?style=flat-square)
![Docente](https://img.shields.io/badge/Docente-Jonathan_Rond%C3%B3n-lightgrey?style=flat-square)
![Materia](https://img.shields.io/badge/Materia-Seguridad_de_Redes-yellow?style=flat-square)

## Topología

Kali agregado a la LAN de Usuarios como origen del escaneo.

![Topología](./images/00-topologia.png)

## DoS Policy

Nombre e interfaz de entrada configurados.

![DoS Policy - nombre e interfaz](./images/01-dos-policy-nombre-interfaz.png)

Anomalías activas: `tcp_port_scan` (threshold 3), `tcp_src_session`, `tcp_dst_session`, `udp_scan`, `icmp_sweep`, todas en Block con logging.

![DoS Policy - anomalías](./images/02-dos-policy-anomalias.png)

## IPS Sensor

Filtro por Vulnerability Type `Anomaly` aplicado con Action Block y Quarantine sobre el atacante.

![IPS Sensor - filtro Anomaly](./images/03-ips-sensor-filtro-anomaly.png)

Perfil IPS asignado a la política `LAN-Usuarios-to-WAN`.

![Política con IPS asignado](./images/04-politica-ips-asignado.png)

## Detección confirmada

Evento `tcp_port_scan` capturado en Security Events, severidad Critical, origen Kali (10.13.67.3).

![Evento tcp_port_scan detectado](./images/05-security-events-tcp-port-scan.png)

## Bloqueo total por Quarantine

Con la IP de Kali en cuarentena, un nuevo escaneo contra cualquier destino queda completamente filtrado, no solo la sesión que disparó la alerta.

![Escaneo bloqueado por completo](./images/06-nmap-bloqueado-total-quarantine.png)
