# 🖥️ Lab: Configuración del Servicio RDP RemoteApp — Web Client

![Estado](https://img.shields.io/badge/Estado-Completado-brightgreen)
![Resultado](https://img.shields.io/badge/Resultado-RD%20Web%20Client%20Funcional-brightgreen)
![Dominio](https://img.shields.io/badge/Dominio-miguel.local-orange)
![HTTPS](https://img.shields.io/badge/Acceso-HTTPS%20Seguro-success)
![Certificado](https://img.shields.io/badge/Certificado-Miguel.localssl-blueviolet)

> 🔑 **Nota importante:** este documento **no crea un certificado nuevo**. Se reutiliza el mismo `Miguel.localssl` creado en el documento **RDP-RemoteAPP.md** (sección 4), tanto para el Gateway como para el Web Client. Solo hace falta asignarlo a los roles nuevos que se agregan aquí (RD Gateway).

---

## ✅ Checklist rápido

- [ ] Certificado `Miguel.localssl` ya creado (documento anterior) — reutilizado aquí
- [ ] RD Gateway instalado
- [ ] Certificado `Miguel.localssl` asignado también al Gateway
- [ ] Certificado del Gateway confiado en Windows10-1 (si no lo estaba ya)
- [ ] Módulo `RDWebClientManagement` instalado
- [ ] Paquete RD Web Client instalado
- [ ] Certificado del Broker importado al Web Client
- [ ] Paquete publicado
- [ ] Acceso probado desde `https://miguel.local/RDWeb/webclient/` sin errores de certificado

---

## 🔒 Reutilizar el certificado `Miguel.localssl`

No hace falta volver a crear el certificado por PowerShell. Si ya hiciste el documento **RDP-RemoteAPP.md**, en `WindowsServer2022-1` ya existe:

- El certificado en el almacén: `Cert:\LocalMachine\My` con FriendlyName **Miguel.localssl**
- El archivo exportado con clave privada: `C:\Certificados\Miguel.localssl.pfx` (contraseña `Segura2121...`)
- El archivo exportado sin clave privada: `C:\Certificados\Miguel.localssl.cer`

Y ya está asignado a los 3 roles de RDS (Agente de conexión x2 + Acceso web) y confiado en Windows10-1. En este documento solo falta **asignarlo también al Gateway** y usarlo en el **Web Client**.

### 🆘 Opción de respaldo: ¿no tienes el certificado `Miguel.localssl`?

Si no hiciste el documento anterior o el certificado ya no existe en el almacén, créalo ahora con el mismo comando y el mismo nombre, para no romper la compatibilidad con el resto de la guía:

```powershell
New-SelfSignedCertificate -DnsName "miguel.local" -CertStoreLocation "Cert:\LocalMachine\My" -KeyUsage DigitalSignature,KeyEncipherment -FriendlyName "Miguel.localssl"
```

Luego expórtalo con clave privada a `C:\Certificados\Miguel.localssl.pfx` (contraseña `Segura2121...`) usando `mmc → Certificados (Equipo local) → Personal → Certificados → Exportar...` (ver el detalle paso a paso en la sección 4.2 del documento **RDP-RemoteAPP.md**), y asígnalo también a los 3 roles de RDS antes de continuar aquí.

---

## 🔧 Instalar RD Gateway

**Administrador del servidor → Administrar → Agregar roles y características**

| Paso | Click |
|---|---|
| 1 | ⭕ **Instalación basada en características o en roles** → Siguiente |
| 2 | Seleccionar servidor → Siguiente |
| 3 | Expande **Servicios de Escritorio remoto** → ✅ **Puerta de enlace de Escritorio remoto** |
| 4 | **Agregar características** → Siguiente |
| 5 | Características → Siguiente |
| 6 | Servicios de acceso y directivas de redes → Siguiente |
| 7 | Nombre de dominio SSL: `miguel.local` |
| 8 | ⭕ **Elegir un certificado para SSL más tarde** → Siguiente |
| 9 | Confirmación → **Instalar** → esperar → **Cerrar** |

## 🔒 Asignar el mismo certificado `Miguel.localssl` al Gateway

**Información general → Tareas → Editar propiedades de la implementación → pestaña Certificados**

| Paso | Click |
|---|---|
| 1 | Selecciona la fila **Puerta de enlace de Escritorio remoto** |
| 2 | **Seleccionar certificado existente...** |
| 3 | Ruta: `C:\Certificados\Miguel.localssl.pfx` → Contraseña: `Segura2121...` |
| 4 | ✅ **Permitir agregar el certificado al almacén de raíz de confianza en equipos de destino** |
| 5 | Aceptar → **Aplicar** → Aceptar |

Al final, las 4 filas (2x Agente de conexión, Acceso web y Gateway) deben mostrar el mismo certificado **Miguel.localssl** con **Estado: Correcto**.

## 🔧 Configurar el Gateway en la implementación

**Editar propiedades de la implementación → pestaña Puerta de enlace de Escritorio remoto**

| Paso | Click |
|---|---|
| 1 | ⭕ **Usar estos valores de configuración de servidor de Puerta de enlace de RD** |
| 2 | Nombre del servidor: `miguel.local` |
| 3 | Método de inicio de sesión: **Autenticación de contraseña** |
| 4 | Desmarca ✅ **No usar el servidor de puerta de enlace de Escritorio remoto para direcciones locales** (déjala **sin marcar**) |
| 5 | Deja marcada solo ✅ **Usar las credenciales de Puerta de enlace de Escritorio remoto para los equipos remotos** |
| 6 | **Aplicar** → Aceptar |

---

## 🔒 Confirmar que Windows10-1 ya confía en el certificado

Como el certificado es el mismo `Miguel.localssl` que ya se confió en el documento anterior, **no hace falta volver a importarlo** en Windows10-1. Solo verifica que siga ahí:

| Paso | Click |
|---|---|
| 1 | En Windows10-1: `Win + R` → `certmgr.msc` |
| 2 | Expande **Entidades de certificación raíz de confianza → Certificados** |
| 3 | Busca `miguel.local` y confirma que aparece un único certificado (no varios "viejos" de intentos anteriores) |
| 4 | Si hay certificados viejos de pruebas anteriores, selecciónalos y **Eliminar**, dejando solo el vigente |

Si por algún motivo no aparece ninguno, repite la exportación sin clave privada y la importación descritas en la sección 4.4–4.5 del documento **RDP-RemoteAPP.md**, usando el archivo `Miguel.localssl.cer`.

---

## 🔧 1. Instalar el módulo de administración

**En WindowsServer2022-1:**

| Paso | Click |
|---|---|
| 1 | Click en el ícono de **Windows** (Inicio) en la barra de tareas |
| 2 | Escribe `PowerShell` |
| 3 | Click derecho en **Windows PowerShell** → **Ejecutar como administrador** |
| 4 | Click en **Sí** en el control de cuentas de usuario |

Con la consola abierta, pega este comando y presiona `Enter`:

```powershell
# 1. Instalar/actualizar el proveedor NuGet
Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force

# 2. Actualizar PowerShellGet
Install-Module -Name PowerShellGet -Force -AllowClobber -Scope AllUsers
```
Cierra la consola y abre una nueva 


## 🔧 2. Instalar el paquete del RD Web Client

```powershell
# 3. Confirmar versión nueva
Get-Module PowerShellGet -ListAvailable

# 4. Reinstalar el módulo RDS (con aceptación de licencia)
Install-Module -Name RDWebClientManagement -Force -AcceptLicense

# 5. Importarlo explícitamente
Import-Module RDWebClientManagement

# 6. Ahora sí
Install-RDWebClientPackage
```

Espera a que termine (no muestra barra de progreso, solo tarda unos segundos).

---

## 🔒 Conectar el certificado `Miguel.localssl` con el Web Client

En la misma consola de PowerShell (administrador), pega:

```powershell
Import-RDWebClientBrokerCert -Path "C:\Certificados\Miguel.localssl.cer"
```

Si sale un mensaje pidiendo confirmar el reemplazo de un certificado existente, responde:

```powershell
Y
```

---

## 🔧 4. Publicar el paquete

```powershell
Publish-RDWebClientPackage -Type Production -Latest
```

---

## 🧪 5. Prueba final desde Windows10-1

| Paso | Click |
|---|---|
| 1 | Cierra el navegador por completo y ábrelo de nuevo en Windows10-1 |
| 2 | Escribe `https://miguel.local/RDWeb/webclient/` |
| 3 | Verifica el candado 🔒 sin advertencia y **sin** `ERR_SSL_KEY_USAGE_INCOMPATIBLE` (mismo certificado `Miguel.localssl` ya confiado) |
| 4 | Inicia sesión con `ronaldrm` / `Segura2121...` |
| 5 | Click en las apps publicadas (**Notepad**, **Calculadora**, **Word**) — se abren dentro del navegador, sin descargar el `.rdp` |

---

## 🩺 Solución de problemas

<details>
<summary>ERR_SSL_KEY_USAGE_INCOMPATIBLE al entrar a https://miguel.local/RDWeb/webclient/</summary>

**Causa:** el certificado activo no tiene el Key Usage `Digital Signature` (por ejemplo, si en algún momento se volvió a usar el asistente gráfico "Crear nuevo certificado..." de la consola RDS en vez del `Miguel.localssl` creado por PowerShell).

**Solución:** verifica en `mmc → Certificados (Equipo local) → Personal` que el certificado enlazado en los 4 roles (Agente x2, Acceso web, Gateway) y en el Web Client sea el mismo `Miguel.localssl`. Si no lo es, sigue la opción de respaldo al inicio de este documento para recrearlo con `-KeyUsage DigitalSignature,KeyEncipherment` y reasignarlo a todos los roles + Web Client + confianza en el cliente.

**Errores de tipeo comunes al escribir el comando a mano:**
- `New-SelfSignedCertificaten` (con "n" de más) → cmdlet no reconocido
- `KeyEnciphermanent` (con "an" de más) → parámetro inválido

Para evitarlos, copia el comando con `Ctrl+C` y pégalo con click derecho o `Ctrl+Shift+V`, no lo retipees a mano.

</details>

---

## 🔁 Comandos de mantenimiento (opcional)

<details>
<summary>Ver comandos útiles</summary>

```powershell
# Ver la versión del Web Client instalada
Get-RDWebClientPackage

# Quitar el paquete (para reinstalar limpio)
Remove-RDWebClientPackage

# Actualizar a la última versión disponible
Uninstall-Module -Name RDWebClientManagement
Install-Module -Name RDWebClientManagement -Force
Install-RDWebClientPackage
Publish-RDWebClientPackage -Type Production -Latest
```
</details>

---

## ✅ Checklist final

- [x] Certificado `Miguel.localssl` ya creado (documento anterior) — reutilizado aquí
- [x] RD Gateway instalado
- [x] Certificado `Miguel.localssl` asignado también al Gateway
- [x] Certificado del Gateway confiado en Windows10-1 (ya lo estaba)
- [x] Módulo `RDWebClientManagement` instalado
- [x] Paquete RD Web Client instalado
- [x] Certificado del Broker importado al Web Client
- [x] Paquete publicado
- [x] Acceso probado desde `https://miguel.local/RDWeb/webclient/` sin errores de certificado
