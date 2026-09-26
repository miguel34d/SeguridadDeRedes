# 🛡️ IPS: Detección de SQL Injection, Bloqueo y Cuarentena

![FortiGate](https://img.shields.io/badge/FortiGate-VM64_KVM-EE3124?style=for-the-badge)
![IPS](https://img.shields.io/badge/IPS-Custom_Signature-yellow?style=for-the-badge)

## 📋 Descripción

Requisito: crear una regla en el FortiGate que detecte intentos de SQL Injection en el tráfico hacia el WEB-Server, los bloquee, y coloque al atacante en cuarentena. Luego inyectar payloads maliciosos y comprobar el bloqueo en los logs.

> ⚠️ **Por qué firma personalizada y no una de FortiGuard:** el catálogo de firmas de FortiGuard en este FortiGate (5,864 firmas) está compuesto casi en su totalidad por firmas específicas de productos conocidos (CMS, foros, plugins, etc.). Como el `buscar.php` de esta práctica es una aplicación casera, ninguna firma de fábrica coincide con su patrón. Por eso se crea una firma personalizada (`Custom Signature`) que detecta el patrón de ataque en sí, sin depender de qué aplicación esté detrás.

---

## Paso 1: crear la firma IPS personalizada

`Security Profiles > IPS Signatures > Create New`

| Campo | Valor |
|---|---|
| Name | `Custom.SQLi.Lab` |
| Comments | `Detecta SQL Injection hacia WEB-Server` |
| Signature | (ver abajo) |

```
F-SBID( --name "Custom.SQLi.Lab"; --protocol tcp; --service HTTP; --flow from_client; --pcre "/(union(\x20|\+|%20)+select|(\x20|\+|%20)or(\x20|\+|%20)+.{1,12}(=|%3d)|information_schema|sleep(\(|%28)\d)/i"; --context uri; )
```

Esta firma detecta, en la URI de peticiones HTTP: `UNION SELECT`, `OR X=X`, referencias a `information_schema`, y uso de `SLEEP()`, patrones típicos de SQL Injection.

Guardar con `OK`.

---

## Paso 2: crear el sensor IPS

`Security Profiles > Intrusion Prevention > Create New`

| Campo | Valor |
|---|---|
| Name | `IPS-SQLi` |
| Block malicious URLs | `Enable` |
| Botnet C&C | `Disable` (o `Block`, opcional) |

Dentro de `IPS Signatures and Filters > Create New`:

| Campo | Valor | Para qué sirve |
|---|---|---|
| Type | `Signature` | Selecciona firmas específicas, no un filtro dinámico |
| Action | `Quarantine` | Bloquea y aísla al atacante, no solo descarta el paquete |
| Quarantine Duration | `0 days`, `0 hours`, `5 minutes` | Tiempo que la IP queda bloqueada |
| Quarantine method | `Attacker IP address` | Bloquea por IP origen |
| Packet logging | `Enable` | Registra el tráfico bloqueado para revisión posterior |
| Status | `Default` | |

En el buscador de firmas, escribir:
```
Custom.SQLi.Lab
```

Marcar el checkbox y confirmar con `Add Selected`. Guardar el sensor con `OK`.

---

## Paso 3: aplicar el sensor a la política de firewall

`Policy & Objects > Firewall Policy > Usuarios_to_WebServer_HTTPS`

| Sección | Campo | Valor |
|---|---|---|
| Security Profiles | IPS | `IPS-SQLi` |
| Security Profiles | SSL Inspection | `custom-deep-inspection` (con `web-server-cert`, ver `activar-dpi.md`) |
| Logging Options | Log Allowed Traffic | `All Sessions` |

Guardar con `OK`.

> Sin el perfil de SSL Inspection activo, el IPS no puede ver dentro del tráfico HTTPS. El DPI y el IPS trabajan juntos: uno descifra, el otro inspecciona.

---

## Paso 4: inyectar payloads de SQL Injection

Desde la máquina de Usuarios (Windows, PowerShell). Usar `curl.exe`, no el alias `curl` de PowerShell.

```powershell
# Prueba de tráfico legítimo primero
curl.exe -k "https://20.13.67.2/buscar.php?q=teclado"
```

**Respuesta esperada:**
```text
1 - Teclado - 25.50<br>
```

```powershell
# Payload 1: bypass de autenticación clásico
curl.exe -k -G "https://20.13.67.2/buscar.php" --data-urlencode "q=' OR '1'='1"

# Payload 2: extracción de datos vía UNION
curl.exe -k -G "https://20.13.67.2/buscar.php" --data-urlencode "q=zzz' UNION SELECT 1,version(),user()-- -"

# Payload 3: inyección basada en tiempo
curl.exe -k -G "https://20.13.67.2/buscar.php" --data-urlencode "q=x' AND SLEEP(5)-- -"
```

---

## Paso 5: resultado esperado

| Petición | Resultado |
|---|---|
| `q=teclado` (legítima, antes del ataque) | `1 - Teclado - 25.50` |
| `q=' OR '1'='1` (primer payload) | `curl: (56) Recv failure: Connection was reset` — el IPS corta la conexión en el momento |
| `q=zzz' UNION SELECT...` (segundo payload) | `curl: (28) Failed to connect... Could not connect to server` — la IP ya está en cuarentena |
| `q=x' AND SLEEP(5)...` (tercer payload) | Mismo error — confirma que la cuarentena sigue activa |
| `q=teclado` repetido durante la cuarentena | También falla, aunque sea tráfico legítimo — la cuarentena bloquea por IP, no por contenido |

---

## Paso 6: verificar en los logs del FortiGate

`Log & Report > Security Events > Intrusion Prevention`

Filtros útiles:
- `Attack Name == Custom.SQLi.Lab`
- `Action == dropped`

**Ejemplo de entrada esperada:**

| Campo | Valor |
|---|---|
| Date/Time | fecha y hora del ataque |
| Severity | `Critical` |
| Source | IP del equipo de Usuarios (ej. `10.13.67.10`) |
| Protocol | `6` (TCP) |
| Action | `dropped` |
| Attack Name | `Custom.SQLi.Lab` |

---

## Paso 7: verificar la cuarentena activa (opcional)

Buscar en el menú del FortiGate el monitor de cuarentena (`Dashboard` o `Monitor`, según la versión). Ahí debe listarse la IP del atacante con el tiempo restante de bloqueo.

---

## ✅ Resumen

| Requisito | Cumplido |
|---|---|
| DPI activo sobre tráfico hacia WEB-Server | ✅ |
| Regla que detecta SQL Injection | ✅ (`Custom.SQLi.Lab`) |
| Bloqueo del ataque | ✅ (`Action: dropped`) |
| Cuarentena del atacante | ✅ (`Quarantine`, 5 minutos, por IP) |
| Payloads inyectados y verificados en logs | ✅ |

---

### 👨‍💻 Realizado por: **Miguel Ramirez Meli**

![Lab](https://img.shields.io/badge/Lab-FortiGate_VM64-purple?style=for-the-badge)
![Status](https://img.shields.io/badge/IPS_SQLi-Funcionando-brightgreen?style=for-the-badge)
