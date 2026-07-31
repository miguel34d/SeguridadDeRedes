# 🌐 Lab: Configuración de una Página Personalizada en IIS (HTTPS/443)

![Estado](https://img.shields.io/badge/Estado-Completado-brightgreen)
![Resultado](https://img.shields.io/badge/Resultado-Sitio%20Web%20Funcional-brightgreen)
![Dominio](https://img.shields.io/badge/Dominio-miguel.local-orange)
![HTTPS](https://img.shields.io/badge/Acceso-HTTPS%20Seguro-success)
![Certificado](https://img.shields.io/badge/Certificado-Miguel.localssl-blueviolet)

> 🔑 **Nota importante:** este documento **no crea un certificado nuevo**. Se reutiliza el mismo `Miguel.localssl` creado en el documento **RDP-RemoteAPP.md**. Solo hace falta enlazarlo al sitio de IIS.

---

## ✅ Checklist rápido

- [ ] Rol **Servidor Web (IIS)** instalado
- [ ] Carpeta del sitio creada con la página personalizada
- [ ] Documento predeterminado configurado
- [ ] Certificado `Miguel.localssl` ya creado (documento anterior) — reutilizado aquí
- [ ] Sitio creado en IIS Manager con enlace HTTPS en el puerto 443 usando `Miguel.localssl`
- [ ] Certificado confiado en Windows10-1 (ya lo estaba)
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

## 🔒 3. Reutilizar el certificado `Miguel.localssl`

No hace falta crear un certificado nuevo para IIS. El mismo `Miguel.localssl` creado por PowerShell en el documento **RDP-RemoteAPP.md** ya está en el almacén `Cert:\LocalMachine\My` de `WindowsServer2022-1` con el Key Usage correcto (`DigitalSignature,KeyEncipherment`), así que aparecerá directamente en el desplegable de certificados al crear el sitio en IIS (paso 4.6 más abajo).

### 🆘 Opción de respaldo: ¿no tienes el certificado `Miguel.localssl`?

Si no hiciste los documentos anteriores o el certificado ya no existe, créalo con el mismo comando y el mismo nombre:

```powershell
New-SelfSignedCertificate -DnsName "miguel.local" -CertStoreLocation "Cert:\LocalMachine\My" -KeyUsage DigitalSignature,KeyEncipherment -FriendlyName "Miguel.localssl"
```

Con esto ya aparecerá disponible en el desplegable de IIS Manager en el paso 4.6. Recuerda repetir también los pasos de exportar y confiar en Windows10-1 (sección 7) si Windows10-1 tampoco lo tiene todavía.

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
| 6 | Certificado SSL: selecciona **Miguel.localssl** (el mismo certificado ya creado y usado en RDS/Gateway) del menú desplegable |
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

## 🔒 7. Confirmar que Windows10-1 ya confía en el certificado

Como el certificado es el mismo `Miguel.localssl` confiado en los documentos anteriores, **normalmente no hace falta volver a importarlo**. Solo verifica:

| Paso | Click |
|---|---|
| 1 | En Windows10-1: `Win + R` → `certmgr.msc` |
| 2 | Expande **Entidades de certificación raíz de confianza → Certificados** |
| 3 | Confirma que aparece `miguel.local` en la lista |

Si no aparece (por ejemplo, porque este es el primer documento que haces del lab), expórtalo sin clave privada y trúalo en Windows10-1:

**En WindowsServer2022-1:**

| Paso | Click |
|---|---|
| 1 | `Win + R` → `mmc` → Archivo → **Agregar o quitar complementos...** → **Certificados** → **Agregar** |
| 2 | ⭕ **Cuenta de equipo** → Siguiente → ⭕ **Equipo local** → Finalizar → Aceptar |
| 3 | Expande **Certificados (Equipo local) → Personal → Certificados** |
| 4 | Click derecho en `CN=miguel.local` (nombre descriptivo **Miguel.localssl**) → **Todas las tareas → Exportar...** |
| 5 | Siguiente → ⭕ **No, no exportar la clave privada** → Siguiente |
| 6 | ⭕ **X.509 codificado en Base64 (.CER)** → Siguiente |
| 7 | Nombre de archivo: `C:\Certificados\Miguel.localssl.cer` → Siguiente → **Finalizar** |
| 8 | Copia `Miguel.localssl.cer` a Windows10-1 (por la carpeta compartida `\\WIN-3RVTQIDV70S\Compartido`) |

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

**Causa:** el certificado no se importó en el almacén **Entidades de certificación raíz de confianza** de Windows10-1, o Windows10-1 tiene un `miguel.local` distinto (de un intento anterior) al que quedó enlazado en IIS.

**Solución:** repite la sección 7 verificando que el `.cer` copiado corresponda al mismo certificado **Miguel.localssl** seleccionado en el enlace HTTPS del sitio (paso 4.6). Si hay certificados `miguel.local` viejos en `certmgr.msc`, elimínalos para evitar confusión.

</details>

<details>
<summary>ERR_SSL_KEY_USAGE_INCOMPATIBLE</summary>

**Causa:** el certificado enlazado no tiene el Key Usage "Digital Signature" (por ejemplo, si se usó un asistente gráfico distinto en vez de reutilizar `Miguel.localssl`). Chrome/Edge actualizados lo rechazan.

**Solución:** verifica en `mmc → Certificados (Equipo local) → Personal` que el certificado enlazado en el sitio sea exactamente `Miguel.localssl` (creado con `-KeyUsage DigitalSignature,KeyEncipherment`). Si no lo es, sigue la opción de respaldo de la sección 3 para recrearlo con ese nombre y reasignarlo al enlace HTTPS del sitio.

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
- [x] Certificado `Miguel.localssl` ya creado (documento anterior) — reutilizado aquí
- [x] Sitio creado en IIS Manager con enlace HTTPS en el puerto 443 usando `Miguel.localssl`
- [x] Certificado confiado en Windows10-1 (ya lo estaba)
- [x] Regla de Firewall habilitada para HTTPS (puerto 443)
- [x] Acceso probado desde `https://miguel.local/` sin errores de certificado
