![Autor](https://img.shields.io/badge/Autor-Miguel_Ramirez_Meli-blue)
![Matricula](https://img.shields.io/badge/Matr%C3%ADcula-20251367-informational)
![Docente](https://img.shields.io/badge/Docente-Jonathan_Rond%C3%B3n-green)
![Materia](https://img.shields.io/badge/Materia-Seguridad_de_Redes-orange)
![Institucion](https://img.shields.io/badge/Instituci%C3%B3n-ITLA-lightgrey)
![Estado](https://img.shields.io/badge/Estado-Completado-success)

# Aplicar WAF (Web Application Firewall) al Servidor Web

## Objetivo

Proteger el servidor **WEB-1** (`20.13.67.10`) contra ataques web comunes (SQL Injection, Cross Site Scripting, escaneo con herramientas automatizadas) mediante un perfil **WAF** aplicado en modo **Proxy-based** sobre la política de firewall que rige el tráfico HTTP desde **Lan-Users** hacia **Lan-Servers** en FortiGate.

## Topología

Cloud1 (ISP) — Router1 — FortiGate (Port1=WAN, Port2=Lan-Users, Port3=Lan-Servers) — Switch1 (Windows10-1 + Kali) — Switch2 (WEB-1)

| Red | Rango |
|---|---|
| LAN Usuarios | `10.13.67.0/24` |
| LAN Servidores | `20.13.67.0/24` |
| WAN Router-FortiGate | `200.13.67.0/30` |

Política base sobre la que se trabaja: **Usuarios-to-Servidores-HTTP** (Lan-Users → Lan-Servers, servicio HTTP).

---

## 1. Habilitar el modo Proxy (requisito para usar WAF)

Los perfiles WAF en FortiOS solo están disponibles en políticas con **modo de inspección Proxy-based**. Por defecto, las políticas nuevas quedan en Flow-based.

### GUI

En **FortiOS moderno, el modo de inspección se cambia editando la política**: `Policy & Objects > Firewall Policy > Usuarios-to-Servidores-HTTP`, y seleccionando **Inspection Mode: Proxy-based** en la parte superior del formulario. Antes de que esta opción esté disponible, se debe habilitar `gui-proxy-inspection` por CLI (no existe equivalente en GUI para este paso inicial).

### CLI

Habilitar la opción en la interfaz gráfica:

```
config system settings
    set gui-proxy-inspection enable
end
```

Cambiar el modo de inspección directamente en la política (en FortiOS 6.4+ ya no existe un modo global `inspection-mode` en `system settings`; se define por política):

```
config firewall policy
    edit 2
        set inspection-mode proxy
    next
end
```

> **Nota:** el ID `2` corresponde a la política `Usuarios-to-Servidores-HTTP` en este laboratorio. Verificar el ID real con `show firewall policy` antes de editar.

---

## 2. Habilitar Web Application Firewall en Feature Visibility

### GUI

1. Ir a **System > Feature Visibility**.
2. En la sección de Security Features, localizar **Web Application Firewall**.
3. Activar el toggle (ya debería estar disponible tras el paso 1).
4. Clic en **Apply**.

### CLI

```
config system settings
    set gui-waf-profile enable
end
```

---

## 3. Crear el perfil WAF

### GUI

1. Ir a **Security Profiles > Web Application Firewall**.
2. Clic en **Create New**.
3. **Name:** `WAF-WebServer-Protection`.
4. Configurar **Signatures**:

| Signature | Status | Action |
|---|---|---|
| Cross Site Scripting | Enable | Block |
| Cross Site Scripting (Extended) | Enable | Block |
| SQL Injection | Enable | Block |
| SQL Injection (Extended) | Enable | Block |
| Generic Attacks | Enable | Block |
| Generic Attacks (Extended) | Enable | Block |
| Trojans | Enable | Block |
| Known Exploits | Enable | Block |
| Information Disclosure | Disable | — |
| Bad Robot | Disable | — |

5. Configurar **Constraints** (todos en Enable / Block):

| Constraint |
|---|
| Illegal Host Name |
| Illegal HTTP Version |
| Illegal HTTP Request Method |
| Content Length |
| Header Length |
| Header Line Length |
| Number of Header Lines in Request |
| Total URL and Body Parameters Length |
| Total URL Parameters Length |
| Number of URL Parameters |
| Number of Cookies in Request |
| Number of Ranges in Range Header |
| Malformed Request |

6. Clic en **OK** para guardar.

### CLI

```
config waf profile
    edit "WAF-WebServer-Protection"
        set extended-log enable
        config constraint
            config hostname
                set status enable
                set action block
                set severity high
            end
            config version
                set status enable
                set action block
                set severity high
            end
            config method
                set status enable
                set action block
                set severity high
            end
            config content-length
                set status enable
                set action block
                set severity high
            end
            config header-length
                set status enable
                set action block
                set severity high
            end
            config line-length
                set status enable
                set action block
                set severity high
            end
            config max-header-line
                set status enable
                set action block
                set severity high
            end
            config param-length
                set status enable
                set action block
                set severity high
            end
            config url-param-length
                set status enable
                set action block
                set severity high
            end
            config malformed
                set status enable
                set action block
                set severity high
            end
        end
    next
end
```

> Las **Signatures** (SQL Injection, XSS, Generic Attacks, Trojans, Known Exploits) se configuraron desde la GUI, ya que la sintaxis CLI de `config signature > config main-class` requiere IDs internos de `main-class` que varían entre builds de FortiOS y no siempre son fáciles de listar de forma consistente por CLI. Se puede verificar el resultado final combinado (GUI + CLI) con:

```
show waf profile "WAF-WebServer-Protection"
```

---

## 4. Aplicar el perfil WAF a la política de firewall

### GUI

1. Ir a **Policy & Objects > Firewall Policy**.
2. Editar `Usuarios-to-Servidores-HTTP`.
3. Confirmar **Inspection mode: Proxy-based**.
4. En **Security Profiles**, activar **Web application firewall** y seleccionar `WAF-WebServer-Protection`.
5. Dejar **SSL inspection** en `certificate-inspection` (se ajusta automáticamente al cambiar a modo proxy).
6. Clic en **OK**.

### CLI

```
config firewall policy
    edit 2
        set waf-profile "WAF-WebServer-Protection"
    next
end
```

Verificación de la política final:

```
show firewall policy 2
```

Salida esperada (resumida):

```
set inspection-mode proxy
set ssl-ssh-profile "certificate-inspection"
set waf-profile "WAF-WebServer-Protection"
set logtraffic all
```

---

## 5. Pruebas de verificación

### 5.1 Tráfico legítimo (control)

Desde un navegador en **Windows10-1**, acceder a `http://20.13.67.10/` — la página debe cargar con normalidad (HTTP 200).

### 5.2 Tráfico de herramienta automatizada (curl) sin payload

```bash
curl -s -o /dev/null -w "%{http_code}\n" "http://20.13.67.10/"
```

Con el perfil final (Bad Robot deshabilitada), este tráfico responde `200`.

### 5.3 Intento de SQL Injection

```bash
curl -s -o /dev/null -w "%{http_code}\n" -G "http://20.13.67.10/" --data-urlencode "id=1' OR '1'='1"
```

Respuesta esperada: `403` (bloqueado por firma **SQL Injection**).

### 5.4 Intento de Cross Site Scripting (XSS)

```bash
curl -s -o /dev/null -w "%{http_code}\n" "http://20.13.67.10/?search=<script>alert(1)</script>"
```

Respuesta esperada: `403` (bloqueado por firma **Cross Site Scripting**).

### 5.5 Verificar en logs (GUI)

**Log & Report > Security Events > Web Application Firewall**, filtrando por `Source: 10.13.67.3` (Kali) y `Action: blocked`. El detalle de cada evento muestra el campo **Message** con el nombre exacto de la firma disparada (`SQL Injection`, `Cross Site Scripting`).

### CLI (referencia)

```
execute log filter category 5
execute log display
```

> En este laboratorio la consulta por CLI con `category 5` no devolvió resultados (posible diferencia de índice de categoría según build); la verificación se realizó desde **Log & Report** en la GUI, donde sí se confirmaron los bloqueos con el detalle completo de cada firma.

---

## Resultado final

| Prueba | Resultado | Firma / Evento |
|---|---|---|
| Navegador legítimo (Windows10-1) | ✅ 200 OK | — |
| curl sin payload | ✅ 200 OK | — |
| SQL Injection (`' OR '1'='1`) | ⛔ 403 Bloqueado | SQL Injection |
| XSS (`<script>alert(1)</script>`) | ⛔ 403 Bloqueado | Cross Site Scripting |

El WAF protege activamente a WEB-1 contra SQL Injection y XSS sin afectar el tráfico legítimo de usuarios ni de herramientas de administración.
