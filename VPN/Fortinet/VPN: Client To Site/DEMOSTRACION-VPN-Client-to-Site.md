# 🔐 Demostración: VPN Client-to-Site (IKEv2 / IPsec) — FortiGate

![Estado](https://img.shields.io/badge/Estado-Completado-brightgreen)
![Resultado](https://img.shields.io/badge/Resultado-VPN%20Client--to--Site%20funcional-brightgreen)
![Modelo](https://img.shields.io/badge/Modelo-Red%20VPN%20%2F24-blue)
![Simulador](https://img.shields.io/badge/Simulador-GNS3-lightgrey)
![IKE](https://img.shields.io/badge/IKE-IKEv2%20%2F%20DES--SHA256-orange)
![Dispositivos](https://img.shields.io/badge/Dispositivos-Router1%20%7C%20Switch1%20%7C%20Switch2%20%7C%20FortiGate%20%7C%20Windows10--1%20%7C%20Windows10--2-lightgrey)

---

### 📋 Datos del estudiante

| Campo | Valor |
|---|---|
| **Nombre** | Miguel Ramírez Mella |
| **Matrícula** | 20251367 |
| **Institución** | ITLA |
| **Profesor** | Jonathan Rondón |
| **Práctica** | Configuración de VPN Client to Site (FortiGate + GNS3) |

> ⚠️ Este documento es una **demostración de resultados**: recopila la evidencia visual que confirma que la VPN Client‑to‑Site fue implementada correctamente y se encuentra 100% funcional. No es una guía paso a paso de configuración.

---

## 🗺️ Topología implementada

![Topología VPN Client to Site](evidencias/image01.png)

La red simula un escenario típico de acceso remoto: dos usuarios (**Windows10‑1** y un segundo cliente FortiClient) se conectan por IPsec a través de Internet (`Cloud1` / `Router-ISP`) hasta el **FortiGate**, obteniendo una IP del pool `30.13.67.10 – 30.13.67.254 /24` y accediendo de forma segura a la LAN interna `20.13.67.0/24`, donde residen **Windows10‑2** y **PC1 (VPCS)**.

---

## ✅ Evidencia 1 — Interfaces del FortiGate configuradas

Se muestran las interfaces **port1 (WAN)** con IP `200.13.67.2/30` y **port2 → LAN-INTERNA** con IP `20.13.67.1/24` y servidor DHCP activo (rango `20.13.67.20 – 20.13.67.254`), tal como fueron definidas para soportar el túnel.

![Interfaces FortiGate](evidencias/image02.png)

---

## ✅ Evidencia 2 — Ruta estática hacia Internet

Ruta por defecto `0.0.0.0/0` apuntando al gateway `200.13.67.1` a través de `port1`, en estado **Enabled**.

![Ruta estática](evidencias/image03.png)

---

## ✅ Evidencia 3 — Usuarios VPN creados

Los usuarios locales `uservpn1` y `uservpn2` fueron creados y habilitados correctamente para la autenticación de los clientes remotos.

![Usuarios VPN](evidencias/image04.png)

---

## ✅ Evidencia 4 — Grupo de usuarios VPN-USERS

El grupo **VPN-USERS** agrupa a `uservpn1` y `uservpn2`, quedando listo para ser referenciado en la política de autenticación del túnel.

![Grupo VPN-USERS](evidencias/image05.png)

---

## ✅ Evidencia 5 — Túnel IPsec "Client-To-Site" configurado

Detalle del túnel creado por el wizard y ajustado manualmente: autenticación por **Pre-shared Key**, **IKEv2**, algoritmos **DES-SHA256**, grupo Diffie-Hellman **14**, y rango de direcciones para clientes `30.13.67.10 – 30.13.67.254 /24`.

![Detalle túnel IPsec](evidencias/image07.png)

---

## ✅ Evidencia 6 — Políticas de firewall activas

Las políticas necesarias quedaron creadas y habilitadas:
- **vpn_Client-To-Site_remote_0** (Túnel → LAN-INTERNA)
- **LAN-TO-VPN** (LAN-INTERNA → Túnel)
- **LAN-to-WAN** (LAN-INTERNA → port1, con NAT activado — tráfico de salida a Internet con 276.59 KB ya cursados)

![Políticas de firewall](evidencias/image08.png)

---

## ✅ Evidencia 7 — Clientes conectados exitosamente vía FortiClient

Dos clientes remotos se conectaron al túnel **Client-To-Site**, recibiendo IPs del pool configurado:

**Cliente 1 — IP asignada `30.13.67.10`**

![FortiClient conectado - cliente 1](evidencias/image09.png)

**Cliente 2 — IP asignada `30.13.67.11`**

![FortiClient conectado - cliente 2](evidencias/image10.png)

---

## ✅ Evidencia 8 — Monitor IPsec del FortiGate

El FortiGate confirma **2 conexiones dialup activas** sobre el túnel `Client-To-Site`, validando que ambos clientes están efectivamente levantados del lado del servidor.

![IPsec Monitor](evidencias/image06.png)

---

## ✅ Evidencia 9 — Conectividad desde la LAN interna hacia los clientes VPN

Desde **PC1 (VPCS)**, tras obtener IP por DHCP (`20.13.67.21/24`), se realizan pings exitosos hacia ambos clientes remotos a través del túnel:

- `ping 30.13.67.10` → 5/5 paquetes exitosos
- `ping 30.13.67.11` → 5/5 paquetes exitosos

![Ping desde PC1 a clientes VPN](evidencias/image11.png)

---

## ✅ Evidencia 10 — Conectividad cruzada entre clientes VPN

Pruebas de ping exitosas realizadas desde `cmd` de Windows entre los clientes conectados al túnel (0% de pérdida en ambos sentidos):

**Ping hacia `30.13.67.11`**

![Ping cmd hacia 30.13.67.11](evidencias/image12.png)

**Ping hacia `30.13.67.10`**

![Ping cmd hacia 30.13.67.10](evidencias/image13.png)

---

## ✅ Evidencia 11 — Verificación de configuración IP y conectividad final (`ipconfig` + `ping`)

Se confirma la asignación de las direcciones del túnel VPN sobre el adaptador virtual de cada equipo, junto con la conectividad exitosa entre ambos extremos:

**Cliente con IP de túnel `30.13.67.10`**

![ipconfig + ping cliente 1](evidencias/image14.png)

**Cliente con IP de túnel `30.13.67.11`**

![ipconfig + ping cliente 2](evidencias/image15.png)

---

## 🏁 Conclusión

Con base en la evidencia recopilada, se confirma que la **VPN Client-to-Site (IKEv2, PSK, DES-SHA256)** implementada sobre el FortiGate quedó **completamente funcional**:

| Criterio | Resultado |
|---|---|
| Interfaces y enrutamiento del FortiGate | ✅ Correcto |
| Usuarios y grupo VPN | ✅ Creados y habilitados |
| Túnel IPsec (Fase 1 / Fase 2) | ✅ Levantado (IKEv2, DES-SHA256, DH14) |
| Políticas de firewall (Túnel↔LAN, LAN↔WAN) | ✅ Activas |
| Conexión de clientes FortiClient | ✅ 2 clientes conectados (30.13.67.10 / .11) |
| Monitor IPsec del FortiGate | ✅ 2 dialup connection(s) activas |
| Conectividad LAN interna ↔ clientes VPN | ✅ 0% pérdida |
| Conectividad entre clientes VPN | ✅ 0% pérdida |

**Resultado final: VPN Client-to-Site funcional al 100%.** ✔️

---

*Documento generado como evidencia de demostración — ITLA — Miguel Ramírez Mella (20251367) — Prof. Jonathan Rondón*
