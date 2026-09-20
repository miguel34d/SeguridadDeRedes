# 🔒 Restricción HTTP (Usuarios → Servidores) + DNS Local — Demostración

![Status](https://img.shields.io/badge/Estado-Funcional-success?style=flat-square)
![Autor](https://img.shields.io/badge/Autor-Miguel_Ramirez_Meli-informational?style=flat-square)
![Matricula](https://img.shields.io/badge/Matr%C3%ADcula-20251367-informational?style=flat-square)
![Docente](https://img.shields.io/badge/Docente-Jonathan_Rond%C3%B3n-lightgrey?style=flat-square)
![Materia](https://img.shields.io/badge/Materia-Seguridad_de_Redes-yellow?style=flat-square)

## Topología

![Topología](./images/00-topologia.png)

## Políticas de firewall

Lista de políticas: `Usuarios-to-Servidores-HTTP` (ACCEPT, sin NAT) por encima de `Usuarios-to-Servidores-DENY` (DENY).

![Lista de políticas](./images/07-politicas-lista.png)

Detalle de la política HTTP, con Service limitado a `HTTP` únicamente.

![Detalle política HTTP](./images/08-politica-http-detalle.png)

Detalle de la política DENY, bloqueando todo lo demás entre las dos LAN.

![Detalle política DENY](./images/09-politica-deny-detalle.png)

## DNS local

DNS Service on Interface en modo Recursive sobre port2, y DNS Database con la zona `miguel.edu.do` creada.

![DNS Service y DNS Database](./images/10-dns-service-database.png)

Detalle de la zona DNS `miguel.edu.do`.

![Detalle zona DNS](./images/11-dns-zone-detalle.png)

Registro A: `miguel.edu.do` → `20.13.67.10`.

![Registro DNS tipo A](./images/12-dns-entry-a-record.png)

DHCP de port2 con el FortiGate (10.13.67.1) como DNS server 1.

![Orden DNS en DHCP](./images/13-dhcp-dns-order.png)

## Verificación desde el cliente

`nslookup miguel.edu.do` resolviendo correctamente a `20.13.67.10`.

![nslookup](./images/02-nslookup-dns-local.png)

Ping bloqueado hacia el servidor — 100% de paquetes perdidos.

![Ping bloqueado](./images/01-ping-bloqueado.png)

Acceso HTTP permitido a `http://miguel.edu.do`.

![HTTP permitido](./images/04-http-permitido-navegador.png)

Acceso HTTPS bloqueado a `https://miguel.edu.do`.

![HTTPS bloqueado](./images/03-https-bloqueado-navegador.png)

## Logs de Forward Traffic

Sesión de ping bloqueada por la política DENY.

![Log ping deny](./images/05-log-ping-deny.png)

Sesión HTTPS bloqueada por la política DENY.

![Log https deny](./images/06-log-https-deny.png)
