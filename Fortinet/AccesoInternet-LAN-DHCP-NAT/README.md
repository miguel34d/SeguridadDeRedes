# 🌐 Acceso a Internet — Demostración

![Status](https://img.shields.io/badge/Estado-Funcional-success?style=flat-square)
![Autor](https://img.shields.io/badge/Autor-Miguel_Ramirez_Meli-informational?style=flat-square)
![Matricula](https://img.shields.io/badge/Matr%C3%ADcula-20251367-informational?style=flat-square)
![Docente](https://img.shields.io/badge/Docente-Jonathan_Rond%C3%B3n-lightgrey?style=flat-square)
![Materia](https://img.shields.io/badge/Materia-Seguridad_de_Redes-yellow?style=flat-square)

## Topología

![Topología](./images/00-topologia.png)

## IP en interfaces

Resumen de interfaces con las IPs aplicadas en Port1, Port2 y Port3.

![Resumen de interfaces](./images/01-interfaces-overview.png)

IP aplicada en Lan-Servers (port3): 20.13.67.1/24.

![Lan-Servers (port3)](./images/02-interface-lan-servidores.png)

IP aplicada en Lan-Users (port2): 10.13.67.1/24.

![Lan-Users (port2)](./images/03-interface-lan-usuarios-dhcp.png)

IP aplicada en Salida-Internet (port1): 200.13.67.2/30.

![Salida-Internet (port1)](./images/04-interface-salida-internet.png)

## DHCP en LAN de usuarios

DHCP habilitado en port2, rango 10.13.67.2 - 10.13.67.254.

![DHCP config](./images/03-interface-lan-usuarios-dhcp.png)

Cliente Windows recibe IP por DHCP correctamente.

![ipconfig cliente](./images/05-ping-ipconfig-usuarios.png)

## Ruta por defecto

Ruta estática 0.0.0.0/0 aplicada por port1 hacia el gateway 200.13.67.1.

![Ruta estática](./images/06-ruta-default.png)

## NAT

Política de NAT activa entre LAN de usuarios y salida a Internet.

![Política NAT - lista](./images/07-politica-nat-lista.png)

Detalle de la política con NAT habilitado (Use Outgoing Interface Address).

![Política NAT - detalle](./images/08-politica-nat-detalle.png)

## Conectividad a Internet

Ping exitoso desde el cliente hacia 8.8.8.8 y google.com, confirmando salida a Internet.

![Ping 8.8.8.8 y google.com](./images/09-ping-internet-dns.png)

