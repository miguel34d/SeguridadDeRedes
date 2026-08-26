# 📸 Evidencia de Implementación — Wazuh SIEM

![Wazuh](https://img.shields.io/badge/Wazuh-4.14.7-3AA5D5?style=for-the-badge&logo=wazuh&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-Server-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Kali](https://img.shields.io/badge/Kali-Linux-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)
![Windows](https://img.shields.io/badge/Windows-10-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![Cisco](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Status](https://img.shields.io/badge/Resultado-Validado-success?style=for-the-badge)
![Autor](https://img.shields.io/badge/Autor-Miguel_Ramirez_Meli-blueviolet?style=for-the-badge)

---

## 1. Topología final (interfaces reales)

![Topología](./evidencias/01_topologia_interfaces.png)

Router1 (e0/0 → Cloud1, e0/1 → Switch1/Admins, e0/2 → Switch2/Users), Switch1 y Switch2 con sus respectivos hosts.

## 2. Descarga del instalador de Wazuh

![Descarga](./evidencias/02_descarga_wazuh_install.png)

`curl -O https://packages.wazuh.com/4.14/wazuh-install.sh` ejecutado en el servidor WAZUH (10.13.67.10).

## 3. Instalación en progreso (indexer + manager + dashboard)

![Instalación](./evidencias/03_instalacion_en_progreso.png)

`sudo bash wazuh-install.sh -a -i` — Wazuh 4.14.7 instalando los tres componentes de forma automática.

## 4. Instalación finalizada con credenciales generadas

![Password](./evidencias/04_instalacion_finalizada_password.png)

Instalación completada, contraseña inicial del usuario `admin` generada por el instalador.

## 5. Conectividad validada desde ADMINISTRADOR

![Ping y TCP test](./evidencias/05_administrador_ping_tcptest_ok.png)
<img width="1026" height="763" alt="image" src="https://github.com/user-attachments/assets/edab45f5-6cf2-407a-9952-b0291d86ee26" />

Desde ADMINISTRADOR (10.13.67.2): `ping` exitoso y `Test-NetConnection -Port 443` con `TcpTestSucceeded: True`.

## 6. Acceso al portal Wazuh desde ADMINISTRADOR

![Login portal](./evidencias/06_portal_wazuh_login_administrador.png)

Portal cargando correctamente en `https://10.13.67.10` desde el equipo autorizado por la ACL.

## 7. Instalación del agente en PC-1 (Windows)

![Agente PC-1](./evidencias/07_pc1_agente_instalado_powershell.png)

Comando generado por el dashboard ejecutado en PowerShell, con `WAZUH_MANAGER='10.13.67.10'` corregido, y servicio iniciado con `NET START Wazuh`.

## 8. PC1-WINDOWS activo en el dashboard

![PC1 activo](./evidencias/08_pc1_agente_activo_dashboard.png)

Agente `001 - PC1-WINDOWS` reportando como **Active**.

## 9. Instalación del agente en PC-2 (Kali)

![Agente PC-2](./evidencias/09_pc2_kali_agente_instalado.png)

Descarga e instalación del agente vía `wget` + `dpkg`, con `WAZUH_MANAGER` corregido a `10.13.67.10`.

## 10. Agentes activos (PC1 y PC2)

![Agentes activos](./evidencias/10_agentes_activos_dashboard.png)

Dashboard mostrando `Agents (2)`: PC1-WINDOWS y PC2-Kali, ambos **Active**.

## 11. ACL bloqueando el portal desde PC-2 (puerto 443)

![ACL bloqueo 443](./evidencias/11_acl_bloquea_pc2_puerto443.png)

`curl` desde PC-2 (VLAN Users) contra `10.13.67.10:443` → `No route to host`, confirmando que la ACL del router bloquea el acceso al portal.

## 12. Puerto SSH (22) expuesto intencionalmente

![Puerto 22 abierto](./evidencias/12_puerto22_abierto_pc2.png)

`curl` contra `10.13.67.10:22` desde PC-2 conecta sin problema (banner OpenSSH visible) — puerto usado luego para la simulación de ataque.

## 13. ACL bloqueando el portal desde PC-1 (navegador)

![ACL bloqueo PC1](./evidencias/13_acl_bloquea_pc1_navegador.png)

Desde PC-1 (Windows, VLAN Users), `https://10.13.67.10` → `ERR_CONNECTION_TIMED_OUT`, confirmando el bloqueo también por navegador.

## 14. Lista de agentes en el manager

![agent_control](./evidencias/14_agent_control_lista_agentes.png)

`agent_control -l` en el servidor WAZUH: agente `000` (el propio manager), `001` (PC1-WINDOWS) y `002` (PC2-Kali), todos **Active**.

## 15. Simulación de ataque (fuerza bruta SSH)

![Ataque hydra](./evidencias/15_ataque_hydra_ejecutando.png)

`hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://10.13.67.10` ejecutado desde PC-2 (Kali) contra el servidor WAZUH.

## 16. Alerta detectada en el dashboard (severidad alta)

![Alerta roja](./evidencias/16_dashboard_alerta_fuerza_bruta.png)

647 eventos capturados, nivel de regla **10** (alto), incluyendo:
- **5551** — PAM: múltiples logins fallidos
- **5712** — sshd: fuerza bruta detectada
- **2502** — intentos individuales de contraseña incorrecta

---

## ✅ Resumen de validación

| Prueba | Resultado |
|---|---|
| Instalación de Wazuh (indexer + manager + dashboard) | ✅ Exitosa |
| Acceso al portal desde ADMINISTRADOR | ✅ Permitido |
| Acceso al portal desde PC-1 (Windows) | ⛔ Bloqueado por ACL |
| Acceso al portal desde PC-2 (Kali) | ⛔ Bloqueado por ACL |
| Agentes PC1-WINDOWS y PC2-Kali | ✅ Activos |
| Ataque simulado (fuerza bruta SSH) | ✅ Detectado (nivel 10) |

---

**Autor:** Miguel Ramírez Meli
