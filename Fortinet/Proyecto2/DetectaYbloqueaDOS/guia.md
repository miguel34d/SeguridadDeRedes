# 🚦 Rate Limiting: Protección contra DoS

![FortiGate](https://img.shields.io/badge/FortiGate-VM64_KVM-EE3124?style=for-the-badge)
![DoS](https://img.shields.io/badge/DoS_Policy-Anomaly_Detection-yellow?style=for-the-badge)

## 📋 Descripción

Requisito: implementar rate limiting en el FortiGate para mitigar ataques de denegación de servicio (DoS) hacia el WEB-Server.

En FortiGate, esto se implementa con una **DoS Policy**, un tipo de política independiente de las políticas de firewall normales. Detecta anomalías de tráfico (volumen de paquetes SYN, sesiones concurrentes por IP, etc.) y aplica una acción de mitigación cuando se supera un umbral configurado.

> ⚠️ **La DoS Policy no se aplica dentro de una política de firewall.** Es su propia entrada en `Policy & Objects > DoS Policy`, y se evalúa de forma independiente y **antes** que las políticas de firewall normales, para el tráfico que coincide con su interfaz de entrada, origen y destino.

---

## Paso 1: activar visibilidad de la función

| Campo | Valor |
|---|---|
| System > Feature Visibility | `DoS Policy`: `ON` |

---

## Paso 2: crear el objeto de dirección del WEB-Server (si no existe)

`Policy & Objects > Addresses > Create New > Address`

| Campo | Valor |
|---|---|
| Name | `WEB-Server` |
| Interface | `LAN-SERVIDORES (port3)` |
| Type | `Subnet` |
| IP/Netmask | `20.13.67.2/32` |

---

## Paso 3: crear la política DoS

`Policy & Objects > DoS Policy > Create New`

### Configuración general

| Campo | Valor |
|---|---|
| Name | `DoS-WEB-Server` |
| Incoming Interface | `LAN-USUARIOS (port2)` |
| Source Address | `all` |
| Destination Address | `WEB-Server` (`20.13.67.2/32`) |
| Service | `ALL` |

### L3 Anomalies

| Name | Logging | Action | Threshold |
|---|---|---|---|
| `ip_src_session` | Enable | Block | `5000` |
| `ip_dst_session` | Enable | Block | `5000` |

### L4 Anomalies — TCP

| Name | Logging | Action | Threshold |
|---|---|---|---|
| `tcp_syn_flood` | Enable | Block | `200` |
| `tcp_port_scan` | Enable | Block | `100` |
| `tcp_src_session` | Enable | Block | `100` |
| `tcp_dst_session` | Enable | Block | `300` |

### L4 Anomalies — UDP

| Name | Logging | Action | Threshold |
|---|---|---|---|
| `udp_flood` | Enable | Block | `200` |
| `udp_scan` | Enable | Block | `2000` |
| `udp_src_session` | Enable | Block | `5000` |
| `udp_dst_session` | Enable | Block | `5000` |

### L4 Anomalies — ICMP

| Name | Logging | Action | Threshold |
|---|---|---|---|
| `icmp_flood` | Enable | Block | `100` |
| `icmp_sweep` | Disable | Disable | `100` |
| `icmp_src_session` | Disable | Disable | `300` |
| `icmp_dst_session` | Disable | Disable | `1000` |

Guardar con `OK`, y confirmar que `Enable this policy` quede en `ON`.

---

## Paso 4: verificar que la política quedó activa

`Policy & Objects > DoS Policy` (vista de lista): `DoS-WEB-Server` debe aparecer habilitada, apuntando a `LAN-USUARIOS (port2)` como entrada y `WEB-Server` como destino.

---

## Paso 5: prueba de carga desde Kali (Usuarios)

### SYN Flood con `hping3` (dispara `tcp_syn_flood`)

```bash
sudo hping3 -S -p 443 --flood -c 3000 20.13.67.2
```

- `-S`: envía solo paquetes SYN, sin completar el handshake TCP.
- `--flood`: envía tan rápido como sea posible, sin esperar respuestas.
- `-c 3000`: cantidad de paquetes a enviar.

Detener con `Ctrl+C` si no se corta automáticamente al completar el conteo.

### Carga HTTP concurrente con `ab` (alternativa, dispara `tcp_src_session`)

```bash
sudo apt install apache2-utils -y
ab -n 20000 -c 300 "https://20.13.67.2/buscar.php?q=teclado"
```

> El SYN flood con `hping3` es más confiable para esta prueba: ataca directamente la capa de red sin depender de que Apache procese cada petición HTTP, mientras que `ab` puede saturarse por límites propios del servidor web (`MaxRequestWorkers`) antes de alcanzar el umbral configurado en el FortiGate.

---

## Paso 6: verificar la detección en los logs

`Log & Report > Security Events > Anomaly > Logs`

Filtros útiles:
- `Attack Name == tcp_syn_flood`
- `Action == clear_session`

**Entrada esperada:**

| Campo | Valor |
|---|---|
| Date/Time | fecha y hora del ataque |
| Severity | `Critical` |
| Source | IP del Kali (ej. `10.13.67.11`) |
| Protocol | `6` (TCP) |
| Action | `clear_session` |
| Count | volumen de paquetes detectado (puede llegar a millones si el flood es sostenido) |
| Attack Name | `tcp_syn_flood` |

> **Sobre la acción `clear_session`:** para la anomalía `tcp_syn_flood`, el FortiGate no descarta cada paquete de forma individual; en su lugar, al superar el umbral, **limpia las sesiones y tablas de estado relacionadas** para liberar recursos del dispositivo y cortar el ataque de raíz. Esta es la acción correcta y esperada para este tipo de anomalía, y confirma que el FortiGate reaccionó activamente al ataque, no solo lo registró.

También puede aparecer una entrada adicional de `tcp_port_scan`, como efecto colateral del volumen de tráfico generado por el flood.

---

## Paso 7: confirmar que el servicio se recupera

Después de detener el ataque, esperar unos segundos y probar tráfico normal:

```bash
curl -k "https://20.13.67.2/buscar.php?q=teclado"
```

Debe volver a responder con normalidad una vez que el volumen de tráfico baja por debajo de los umbrales configurados.

---

## ✅ Resumen

| Requisito | Cumplido |
|---|---|
| Política DoS creada y aplicada al WEB-Server | ✅ (`DoS-WEB-Server`) |
| Umbrales de rate limiting configurados | ✅ (`tcp_syn_flood: 200`, `tcp_src_session: 100`, entre otros) |
| Ataque de prueba inyectado | ✅ (`hping3 --flood`) |
| Detección confirmada en logs | ✅ (`Attack Name: tcp_syn_flood`, `Severity: Critical`) |
| Mitigación aplicada | ✅ (`Action: clear_session`) |
| Recuperación del servicio tras el ataque | ✅ |

---

### 👨‍💻 Realizado por: **Miguel Ramirez Meli**

![Lab](https://img.shields.io/badge/Lab-FortiGate_VM64-purple?style=for-the-badge)
![Status](https://img.shields.io/badge/DoS_Policy-Funcionando-brightgreen?style=for-the-badge)
