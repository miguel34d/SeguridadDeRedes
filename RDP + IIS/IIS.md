# 🌐 Lab: Configuración de una Página Personalizada en IIS (HTTPS/443)

![Estado](https://img.shields.io/badge/Estado-Completado-brightgreen)
![Resultado](https://img.shields.io/badge/Resultado-Sitio%20Web%20Funcional-brightgreen)
![Dominio](https://img.shields.io/badge/Dominio-miguel.local-orange)
![HTTPS](https://img.shields.io/badge/Acceso-HTTPS%20Seguro-success)

---

## ✅ Checklist rápido

- [ ] Rol **Servidor Web (IIS)** instalado
- [ ] Carpeta del sitio creada con la página personalizada
- [ ] Documento predeterminado configurado
- [ ] Certificado `miguel.local` creado por PowerShell con Key Usage correcto
- [ ] Sitio creado en IIS Manager con enlace HTTPS en el puerto 443
- [ ] Certificado confiado en Windows10-1
- [ ] Regla de Firewall habilitada para HTTPS (puerto 443)
- [ ] Acceso probado desde `https://miguel.local/` sin errores de certificado

---

## 🔧 1. Instalar el rol Servidor Web (IIS)

**En WindowsServer2022-1 → Administrador del servidor → Administrar → Agregar roles y características**

| Paso | Click |
|---|---|
| 1 | ⭕ **Instalación basada en características o en roles** → Siguiente |
| 2 | Seleccionar servidor → Siguiente |
| 3 | Roles de servidor → ✅ **Servidor Web (IIS)** |
| 4 | **Agregar características** (para agregar las herramientas de administración) → Siguiente |
| 5 | Características → Siguiente |
| 6 | Servidor Web (IIS) → Siguiente |
| 7 | Servicios de rol → asegúrate de incluir ✅ **Seguridad → Seguridad de contenido estático** y ✅ **Rendimiento en el desarrollo de aplicaciones → según lo necesites** (deja marcados los predeterminados) → Siguiente |
| 8 | Confirmación → **Instalar** → esperar → **Cerrar** |

---

## 📁 2. Crear la carpeta del sitio y la página personalizada

**Explorador de archivos en WindowsServer2022-1:**

| Paso | Click |
|---|---|
| 1 | Crea la carpeta `C:\inetpub\miguel-site` |
| 2 | Dentro de esa carpeta, crea un archivo de texto y renómbralo a `index.html` |
| 3 | Ábrelo con el Bloc de notas y pega el contenido HTML de la página (ver ejemplo abajo) |
| 4 | Guarda el archivo con codificación **UTF-8** |

**Contenido de ejemplo para `index.html`:**

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>miguel.local</title>
    <style>
        body {
            background-color: #1E1E1E;
            color: #F0F0F0;
            font-family: Segoe UI, Arial, sans-serif;
            text-align: center;
            padding-top: 80px;
        }
        h1 {
            color: #4EC9B0;
        }
    </style>
</head>
<body>
    <h1>Bienvenido a miguel.local</h1>
    <p>Página personalizada servida por IIS en WindowsServer2022-1 vía HTTPS.</p>
</body>
</html>
```

---

## 🔒 3. Crear el certificado por PowerShell (Key Usage correcto)

**En WindowsServer2022-1 → PowerShell como administrador:**

```powershell
New-SelfSignedCertificate -DnsName "miguel.local" -CertStoreLocation "Cert:\LocalMachine\My" -KeyUsage DigitalSignature,KeyEncipherment -FriendlyName "miguel.local-iis"
```

La salida debe mostrar un **Thumbprint** y `Subject: CN=miguel.local`. Anota o recuerda el thumbprint por si hay varios certificados parecidos más adelante.

---

## 🔧 4. Crear el sitio web en IIS Manager con enlace HTTPS

**Herramientas → Administrador de Internet Information Services (IIS)**

| Paso | Click |
|---|---|
| 1 | En el panel izquierdo, expande el servidor → click derecho en **Sitios** → **Agregar sitio web...** |
| 2 | Nombre del sitio: `miguel-site` |
| 3 | Ruta de acceso física: `C:\inetpub\miguel-site` |
| 4 | Tipo: **https** — Dirección IP: **Todas sin asignar** — Puerto: **443** |
| 5 | Nombre de host: `miguel.local` |
| 6 | Certificado SSL: selecciona **miguel.local-iis** (el certificado creado en el paso 3) del menú desplegable |
| 7 | Aceptar |

---

## 📄 5. Configurar el documento predeterminado

**En IIS Manager, con el sitio `miguel-site` seleccionado:**

| Paso | Click |
|---|---|
| 1 | Doble click en **Documento predeterminado** (panel central) |
| 2 | Verifica que `index.html` esté en la lista |
| 3 | Si no está, click en **Agregar...** (panel derecho) → escribe `index.html` → Aceptar |
| 4 | Selecciónalo y usa **Subir** (panel derecho) para que quede de primero en la lista |

---

## 🔥 6. Habilitar el Firewall para HTTPS

**En WindowsServer2022-1 → PowerShell como administrador:**

```powershell
New-NetFirewallRule -DisplayName "Permitir HTTPS (Puerto 443)" -Direction Inbound -Protocol TCP -LocalPort 443 -Action Allow
```

---

## 🔒 7. Confiar en el certificado desde Windows10-1

**En WindowsServer2022-1:**

| Paso | Click |
|---|---|
| 1 | `Win + R` → `mmc` → Archivo → **Agregar o quitar complementos...** → **Certificados** → **Agregar** |
| 2 | ⭕ **Cuenta de equipo** → Siguiente → ⭕ **Equipo local** → Finalizar → Aceptar |
| 3 | Expande **Certificados (Equipo local) → Personal → Certificados** |
| 4 | Click derecho en `CN=miguel.local` (nombre descriptivo **miguel.local-iis**) → **Todas las tareas → Exportar...** |
| 5 | Siguiente → ⭕ **No, no exportar la clave privada** → Siguiente |
| 6 | ⭕ **X.509 codificado en Base64 (.CER)** → Siguiente |
| 7 | Nombre de archivo: `C:\Certificados\miguel-local-iis.cer` → Siguiente → **Finalizar** |
| 8 | Copia `miguel-local-iis.cer` a Windows10-1 (por la carpeta compartida `\\WIN-3RVTQIDV70S\Compartido`) |

**En Windows10-1:**

| Paso | Click |
|---|---|
| 9 | Doble click sobre el archivo → **Instalar certificado...** → ⭕ **Equipo local** → Siguiente |
| 10 | ⭕ **Colocar todos los certificados en el siguiente almacén** → **Examinar...** → **Entidades de certificación raíz de confianza** → Aceptar |
| 11 | Siguiente → **Finalizar** → debe salir **"La importación se realizó correctamente"** |

---

## 🧪 8. Probar el acceso desde Windows10-1

| Paso | Click |
|---|---|
| 1 | Cierra el navegador por completo y ábrelo de nuevo en Windows10-1 |
| 2 | Escribe `https://miguel.local/` |
| 3 | Verifica el candado 🔒 sin advertencia de certificado |
| 4 | Confirma que cargue la página personalizada con el mensaje de bienvenida |

---

## 🩺 Solución de problemas

<details>
<summary>Advertencia de certificado no confiable en el navegador</summary>

**Causa:** el certificado no se importó en el almacén **Entidades de certificación raíz de confianza** de Windows10-1, o se importó con un thumbprint distinto al que quedó enlazado en IIS.

**Solución:** repite el paso 7 verificando que el `.cer` copiado corresponda al mismo certificado seleccionado en el enlace HTTPS del sitio (paso 4.6).

</details>

<details>
<summary>ERR_SSL_KEY_USAGE_INCOMPATIBLE</summary>

**Causa:** el certificado fue creado sin el key usage "Digital Signature" (por ejemplo, con un asistente gráfico que solo asigna "Key Encipherment"). Chrome/Edge actualizados lo rechazan.

**Solución:** vuelve a crear el certificado con `New-SelfSignedCertificate` incluyendo `-KeyUsage DigitalSignature,KeyEncipherment` (paso 3) y reasígnalo al enlace HTTPS del sitio.

</details>

<details>
<summary>La página no carga / "No se puede acceder a este sitio"</summary>

**Causas comunes:**
- El Firewall en WindowsServer2022-1 no tiene la regla del puerto 443 habilitada — revisa el paso 6.
- El nombre `miguel.local` no resuelve en Windows10-1 — verifica el registro correspondiente en DNS.
- El enlace en IIS Manager quedó con un nombre de host distinto al que se está escribiendo en el navegador — revisa **Enlaces...** en el sitio `miguel-site`.

</details>

---

## 🔁 Comandos de mantenimiento (opcional)

<details>
<summary>Ver comandos útiles</summary>

```powershell
# Ver el estado del sitio
Get-Website -Name "miguel-site"

# Ver los enlaces (bindings) del sitio
Get-WebBinding -Name "miguel-site"

# Reiniciar el sitio
Restart-WebItem "IIS:\Sites\miguel-site"

# Ver las reglas de firewall relacionadas a HTTPS
Get-NetFirewallRule -DisplayName "*HTTPS*"

# Ver los certificados instalados en el almacén personal
Get-ChildItem -Path Cert:\LocalMachine\My
```
</details>

---

## ✅ Checklist final

- [x] Rol **Servidor Web (IIS)** instalado
- [x] Carpeta del sitio creada con la página personalizada
- [x] Documento predeterminado configurado
- [x] Certificado `miguel.local` creado por PowerShell con Key Usage correcto
- [x] Sitio creado en IIS Manager con enlace HTTPS en el puerto 443
- [x] Certificado confiado en Windows10-1
- [x] Regla de Firewall habilitada para HTTPS (puerto 443)
- [x] Acceso probado desde `https://miguel.local/` sin errores de certificado
