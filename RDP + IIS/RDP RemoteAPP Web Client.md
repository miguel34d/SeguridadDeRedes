# 🖥️ Lab: Configuración del Servicio RDP RemoteApp — Web Client

![Estado](https://img.shields.io/badge/Estado-Completado-brightgreen)
![Resultado](https://img.shields.io/badge/Resultado-RD%20Web%20Client%20Funcional-brightgreen)
![Dominio](https://img.shields.io/badge/Dominio-miguel.local-orange)
![HTTPS](https://img.shields.io/badge/Acceso-HTTPS%20Seguro-success)

---

## ✅ Checklist rápido

- [ ] Certificado `miguel.local` creado por PowerShell con Key Usage correcto
- [ ] Certificado asignado a los 3 roles RDS + Gateway
- [ ] Certificado confiado en Windows10-1
- [ ] RD Gateway instalado y configurado en la implementación
- [ ] Módulo `RDWebClientManagement` instalado
- [ ] Paquete RD Web Client instalado
- [ ] Certificado del Broker importado al Web Client
- [ ] Paquete publicado
- [ ] Acceso probado desde `https://miguel.local/RDWeb/webclient/` sin errores de certificado

---

## 🔒 Crear el certificado por PowerShell (Key Usage correcto)

**En WindowsServer2022-1 → PowerShell como administrador:**

```powershell
New-SelfSignedCertificate -DnsName "miguel.local" -CertStoreLocation "Cert:\LocalMachine\My" -KeyUsage DigitalSignature,KeyEncipherment -FriendlyName "miguel.local-v2"
```

La salida debe mostrar un **Thumbprint** y `Subject: CN=miguel.local`. Anota o recuerda el thumbprint por si hay varios certificados parecidos más adelante.

---

## 🔒 Exportar el certificado con clave privada (.pfx)

**Ejecutar (`Win + R`) → `mmc`**

| Paso | Click |
|---|---|
| 1 | Archivo → **Agregar o quitar complementos...** → **Certificados** → **Agregar** |
| 2 | ⭕ **Cuenta de equipo** → Siguiente → ⭕ **Equipo local** → Finalizar → Aceptar |
| 3 | Expande **Certificados (Equipo local) → Personal → Certificados** |
| 4 | Busca el certificado `CN=miguel.local` con nombre descriptivo **miguel.local-v2** (identifícalo por el thumbprint si hay varios parecidos) |
| 5 | Click derecho sobre él → **Todas las tareas → Exportar...** |
| 6 | Siguiente → ⭕ **Sí, exportar la clave privada** → Siguiente |
| 7 | ⭕ **Intercambio de información personal - PKCS #12 (.PFX)** → ✅ Incluir todos los certificados en la ruta de acceso → Siguiente |
| 8 | ✅ Contraseña: `Segura2121...` → confirmar contraseña → Siguiente |
| 9 | Nombre de archivo: `C:\Certificados\miguel-local-v2.pfx` → Siguiente → **Finalizar** |
| 10 | Debe salir: **"La exportación se realizó correctamente"** → Aceptar |

---

## 🔒 Asignar el certificado a los 3 roles RDS

**Consola RDS → Información general → Tareas → Editar propiedades de la implementación → pestaña Certificados**

| Paso | Click |
|---|---|
| 1 | Selecciona la fila **Agente de conexión a Escritorio remoto - Habilitar inicio de sesión único** |
| 2 | **Seleccionar certificado existente...** |
| 3 | Ruta: `C:\Certificados\miguel-local-v2.pfx` → Contraseña: `Segura2121...` |
| 4 | ✅ **Permitir agregar el certificado al almacén de certificados Entidades de certificación raíz de confianza en los equipos de destino** |
| 5 | Aceptar → **Aplicar** |
| 6 | Selecciona la fila **Agente de conexión a Escritorio remoto - Publicación** → repite pasos 2-4 → Aceptar → **Aplicar** |
| 7 | Selecciona la fila **Acceso web a Escritorio remoto** → repite pasos 2-4 → Aceptar → **Aplicar** |

Al final las 4 filas (contando el Gateway de la siguiente sección) deben mostrar **Estado: Correcto**.

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

## 🔒 Asignar el certificado al Gateway

**Información general → Tareas → Editar propiedades de la implementación → pestaña Certificados**

| Paso | Click |
|---|---|
| 1 | Selecciona la fila **Puerta de enlace de Escritorio remoto** |
| 2 | **Seleccionar certificado existente...** |
| 3 | Ruta: `C:\Certificados\miguel-local-v2.pfx` → Contraseña: `Segura2121...` |
| 4 | ✅ **Permitir agregar el certificado al almacén de raíz de confianza en equipos de destino** |
| 5 | Aceptar → **Aplicar** → Aceptar |

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

## 🔒 Confiar en el certificado desde Windows10-1

| Paso | Click |
|---|---|
| 1 | En WindowsServer2022-1, exporta el certificado **sin clave privada**: en `mmc → Certificados (Equipo local) → Personal → Certificados`, click derecho en `CN=miguel.local` (mismo thumbprint del `.pfx`) → **Todas las tareas → Exportar...** |
| 2 | Siguiente → ⭕ **No, no exportar la clave privada** → Siguiente |
| 3 | ⭕ **X.509 codificado en Base64 (.CER)** → Siguiente |
| 4 | Nombre de archivo: `C:\Certificados\miguel-local-v2.cer` → Siguiente → **Finalizar** |
| 5 | Copia `miguel-local-v2.cer` a Windows10-1 (por la carpeta compartida `\\WIN-3RVTQIDV70S\Compartido`) |
| 6 | En Windows10-1, doble click sobre el archivo → **Instalar certificado...** → ⭕ **Equipo local** → Siguiente |
| 7 | ⭕ **Colocar todos los certificados en el siguiente almacén** → **Examinar...** → **Entidades de certificación raíz de confianza** → Aceptar |
| 8 | Siguiente → **Finalizar** → debe salir **"La importación se realizó correctamente"** |
| 9 | *(Opcional, recomendado)* `Win + R` → `certmgr.msc` → **Entidades de certificación raíz de confianza → Certificados** → busca cualquier `miguel.local` **viejo** (de intentos anteriores) → click derecho → **Eliminar**, para que no quede conviviendo con el nuevo y genere confusión |

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
Install-Module -Name RDWebClientManagement -Force
```

Si pregunta por confiar en el repositorio NuGet o PSGallery, responde:

```powershell
Y
```

---

## 🔧 2. Instalar el paquete del RD Web Client

```powershell
Install-RDWebClientPackage
```

Espera a que termine (no muestra barra de progreso, solo tarda unos segundos).

---

## 🔒 Conectar el certificado con el Web Client

En la misma consola de PowerShell (administrador), pega:

```powershell
Import-RDWebClientBrokerCert -Path "C:\Certificados\miguel-local-v2.cer"
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
| 3 | Verifica el candado 🔒 sin advertencia y **sin** `ERR_SSL_KEY_USAGE_INCOMPATIBLE` (mismo certificado ya confiado) |
| 4 | Inicia sesión con `ronaldrm` / `Segura2121...` |
| 5 | Click en las apps publicadas (**Notepad**, **Calculadora**, **Word**) — se abren dentro del navegador, sin descargar el `.rdp` |

---

## 🩺 Solución de problemas

<details>
<summary>ERR_SSL_KEY_USAGE_INCOMPATIBLE al entrar a https://miguel.local/RDWeb/webclient/</summary>

**Causa:** el certificado activo fue creado con el asistente gráfico de RDS (`Crear nuevo certificado...`), que solo asigna el key usage "Key Encipherment". Chrome/Edge actualizados exigen también "Digital Signature".

**Solución:** repetir todo el flujo de este documento desde **"Crear el certificado por PowerShell"**, usando `New-SelfSignedCertificate` con `-KeyUsage DigitalSignature,KeyEncipherment`, y reasignar el `.pfx`/`.cer` resultante a los 3 roles + Gateway + Web Client + confianza en el cliente.

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

- [x] Certificado `miguel.local` creado por PowerShell con Key Usage correcto
- [x] Certificado asignado a los 3 roles RDS + Gateway
- [x] Certificado confiado en Windows10-1
- [x] RD Gateway instalado y configurado en la implementación
- [x] Módulo `RDWebClientManagement` instalado
- [x] Paquete RD Web Client instalado
- [x] Certificado del Broker importado al Web Client
- [x] Paquete publicado
- [x] Acceso probado desde `https://miguel.local/RDWeb/webclient/` sin errores de certificado
