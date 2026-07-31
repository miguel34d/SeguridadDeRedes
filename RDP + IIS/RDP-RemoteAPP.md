# 🖥️ Lab: Configuración del Servicio RDP RemoteApp

![Estado](https://img.shields.io/badge/Estado-Completado-brightgreen)
![Resultado](https://img.shields.io/badge/Resultado-RemoteApp%20Funcional-brightgreen)
![Dominio](https://img.shields.io/badge/Dominio-miguel.local-orange)
![HTTPS](https://img.shields.io/badge/RD%20Web%20Access-HTTPS%20Seguro-success)
![Certificado](https://img.shields.io/badge/Certificado-Miguel.localssl-blueviolet)

> ℹ️ Requisito previo: el dominio `miguel.local` ya debe estar creado, con `WindowsServer2022-1` como DC y `Windows10-1` unido al dominio (ver documento **Creación del Dominio**).

> 🔑 **Nota importante:** este lab usa un único certificado llamado **`Miguel.localssl`**, creado por PowerShell con el Key Usage correcto (`DigitalSignature,KeyEncipherment`). Este mismo certificado se reutiliza sin cambios en los documentos **RD RemoteApp Web Client** e **IIS**, así que solo se crea **una vez, aquí**.

---

## ✅ Checklist rápido

- [ ] Rol RDS instalado
- [ ] Colección de sesión creada
- [ ] Apps publicadas (Notepad, Calc, Word)
- [ ] Certificado `Miguel.localssl` creado por PowerShell
- [ ] Certificado asignado a los roles RDS (Agente de conexión x2 + Acceso web)
- [ ] Certificado confiado en Windows10-1
- [ ] Registro DNS `miguel.local` apuntando al servidor RDS
- [ ] Acceso probado desde `https://miguel.local/rdweb` sin advertencia

---

## 🔧 1. Instalar rol RDS (Server Manager)

**Administrador del servidor → Administrar → Agregar roles y características**

| Paso | Click |
|---|---|
| 1 | Siguiente (bienvenida) |
| 2 | ⭕ Instalación basada en características o roles → Siguiente |
| 3 | Seleccionar servidor → Siguiente |
| 4 | ✅ Servicios de Escritorio remoto → Agregar características → Siguiente |
| 5 | Características → Siguiente (sin marcar nada) |
| 6 | Pantalla informativa RDS → Siguiente |
| 7 | **Servicios de rol**, marca las 4 casillas: ✅ **Acceso web a Escritorio remoto** · ✅ **Administración de licencias de Escritorio remoto** · ✅ **Agente de conexión a Escritorio remoto** · ✅ **Host de sesión de Escritorio remoto** |
| 8 | Si aparece un cuadro pidiendo características adicionales → **Agregar características** → Siguiente |
| 9 | Rol de servidor web (IIS) → Siguiente → Servicios de rol (IIS) → Siguiente (dejar lo predeterminado) |
| 10 | Confirmación → ✅ Reiniciar automáticamente el servidor de destino si es necesario |
| 11 | **Instalar** → esperar reinicio → Cerrar |

> ⚠️ **Host de sesión de Escritorio remoto** es fácil de pasar por alto, pero es obligatorio: es el rol que realmente ejecuta las RemoteApps (Notepad, Calculadora, Word). Sin él no vas a poder crear la colección de sesión más adelante.

![Servicios de rol RDS](images/servicios_rol.png)

---

## 🔧 1.1 Completar la implementación de Escritorio remoto (primera vez)

Al entrar por primera vez a **Servicios de Escritorio remoto → Vista general**, Server Manager lanza automáticamente el asistente de implementación (si aún no existe ninguna). Si no aparece solo, click en **Servicios de Escritorio remoto** en el panel izquierdo.

| Paso | Click |
|---|---|
| 1 | **Seleccionar tipo de implementación** → ⭕ **Implementación estándar** → Siguiente |
| 2 | **Seleccionar escenario de implementación** → ⭕ **Implementación de escritorio basada en sesión** → Siguiente |
| 3 | Servicios de rol (pantalla informativa) → Siguiente |
| 4 | **Agente de conexión a Escritorio remoto** → selecciona `WIN-3RVTQIDV70S.miguel.local` → **Agregar** → Siguiente |
| 5 | **Acceso web de RD** → selecciona `WIN-3RVTQIDV70S.miguel.local` → **Agregar** → Siguiente |
| 6 | **Host de sesión de Escritorio remoto** → selecciona `WIN-3RVTQIDV70S.miguel.local` → **Agregar** → Siguiente |
| 7 | **Confirmación** → ✅ **Reiniciar automáticamente el servidor de destino si es necesario** |
| 8 | **Implementar** → esperar → **Cerrar** |

> ⚠️ Este asistente es el que realmente crea la implementación de RDS. Sin completarlo, las opciones de **Colecciones** y **Editar propiedades de implementación** (usadas más adelante para el certificado) no van a estar disponibles.

---

## 🔧 2. Crear la colección de sesión

**Administrador del servidor → Servicios de Escritorio remoto → Información general**

| Paso | Click |
|---|---|
| 1 | En la sección **"Configurar una implementación de Servicios de Escritorio remoto"**, click en el link **"3. Crear colecciones de sesiones"** |
| 2 | Siguiente (bienvenida) |
| 3 | Nombre de la colección: `ColeccionRemoteApp` → Siguiente |
| 4 | ✅ Seleccionar el servidor `WIN-3RVTQIDV70S.MIGUEL.LOCAL` → **Agregar** → Siguiente |
| 5 | ✅ **Grupo de usuarios**: agrega `MIGUEL\ronaldrm` (o `MIGUEL\Domain Users`) → Siguiente |
| 6 | **Discos de perfil de usuario** → **desmarca** ✅ **Habilitar discos de perfil de usuario** → Siguiente |
| 7 | Revisar → **Crear** → Cerrar |

> ℹ️ Los discos de perfil de usuario necesitan una carpeta compartida (ruta UNC) creada de antemano. Para este lab se desactivan y los perfiles quedan guardados localmente en el servidor.

---

## 🔧 3. Publicar RemoteApps

**Administrador del servidor → Servicios de Escritorio remoto → Colecciones → `ColeccionRemoteApp`**

| Paso | Click |
|---|---|
| 1 | En **Programas RemoteApp** → **Tareas** → **Publicar programas RemoteApp** |
| 2 | ✅ Marca **Notepad**, **Calculadora**, **Word** → Siguiente |
| 3 | **Publicar** → Cerrar |

---

## 🔒 4. Certificado SSL único: `Miguel.localssl`

Este es el **único punto del lab donde se crea el certificado**. En los otros documentos (Web Client / IIS) solo se **selecciona el mismo archivo**, nunca se vuelve a crear uno nuevo.

### 4.1 Crear el certificado por PowerShell (Key Usage correcto)

**En WindowsServer2022-1 → PowerShell como administrador:**

```powershell
New-SelfSignedCertificate -DnsName "miguel.local" -CertStoreLocation "Cert:\LocalMachine\My" -KeyUsage DigitalSignature,KeyEncipherment -FriendlyName "Miguel.localssl"
```

La salida debe mostrar un **Thumbprint** y `Subject: CN=miguel.local`. Anota el thumbprint, porque será el mismo certificado que vas a reutilizar en RD Gateway, Web Client e IIS.

> ⚠️ Se usa PowerShell (no el asistente gráfico "Crear nuevo certificado..." de la consola RDS) porque ese asistente solo asigna el Key Usage `Key Encipherment`, y los navegadores modernos (Chrome/Edge) también exigen `Digital Signature`. Si lo creas por el asistente gráfico vas a terminar con `ERR_SSL_KEY_USAGE_INCOMPATIBLE` más adelante.

### 4.2 Exportar el certificado con clave privada (.pfx)

**Ejecutar (`Win + R`) → `mmc`**

| Paso | Click |
|---|---|
| 1 | Archivo → **Agregar o quitar complementos...** → **Certificados** → **Agregar** |
| 2 | ⭕ **Cuenta de equipo** → Siguiente → ⭕ **Equipo local** → Finalizar → Aceptar |
| 3 | Expande **Certificados (Equipo local) → Personal → Certificados** |
| 4 | Busca el certificado `CN=miguel.local` con nombre descriptivo **Miguel.localssl** |
| 5 | Click derecho sobre él → **Todas las tareas → Exportar...** |
| 6 | Siguiente → ⭕ **Sí, exportar la clave privada** → Siguiente |
| 7 | ⭕ **Intercambio de información personal - PKCS #12 (.PFX)** → ✅ Incluir todos los certificados en la ruta de acceso → Siguiente |
| 8 | ✅ Contraseña: `Segura2121...` → confirmar contraseña → Siguiente |
| 9 | Nombre de archivo: `C:\Certificados\Miguel.localssl.pfx` → Siguiente → **Finalizar** |
| 10 | Debe salir: **"La exportación se realizó correctamente"** → Aceptar |

### 4.3 Asignar el certificado a los roles RDS

**Información general → Tareas → Editar propiedades de la implementación → pestaña Certificados**

En la tabla **"Administrar certificados"** verás 4 filas: 2x **Agente de conexión a Escritorio remoto**, **Acceso web a Escritorio remoto** y **Puerta de enlace de Escritorio remoto** (esta última en gris por ahora, no aplica todavía porque no instalamos ese rol en este documento).

| Paso | Click |
|---|---|
| 1 | Selecciona la primera fila **Agente de conexión a Escritorio remoto** (la de "Publicación") |
| 2 | **Seleccionar certificado existente...** |
| 3 | Ruta: `C:\Certificados\Miguel.localssl.pfx` → Contraseña: `Segura2121...` |
| 4 | ✅ **Permitir agregar el certificado al almacén de certificados Entidades de certificación raíz de confianza en los equipos de destino** |
| 5 | Aceptar → **Aplicar** |
| 6 | Repite pasos 2–4 en la fila **Agente de conexión a Escritorio remoto** (la de "Autenticación de identidad web único") → **Aplicar** |
| 7 | Repite pasos 2–4 en la fila **Acceso web a Escritorio remoto** → **Aplicar** |
| 8 | Verifica que las 3 filas queden con **Estado: Correcto** |

### 4.4 Exportar el certificado (sin clave privada) para el cliente

**En WindowsServer2022-1 → mmc → Certificados (Equipo local) → Personal → Certificados**

| Paso | Click |
|---|---|
| 1 | Click derecho en `CN=miguel.local` (nombre descriptivo **Miguel.localssl**) → **Todas las tareas → Exportar...** |
| 2 | Siguiente → ⭕ **No, no exportar la clave privada** → Siguiente |
| 3 | ⭕ **X.509 codificado en Base64 (.CER)** → Siguiente |
| 4 | Nombre de archivo: `C:\Certificados\Miguel.localssl.cer` → Siguiente → **Finalizar** |
| 5 | Copia `Miguel.localssl.cer` a una carpeta compartida o USB para llevarlo a Windows10-1 |

### Crear la carpeta compartida en WindowsServer2022-1

| Paso | Click |
|---|---|
| 1 | Explorador de archivos → crea una carpeta nueva, ej. `C:\Compartido` |
| 2 | Copia dentro el archivo `Miguel.localssl.cer` |
| 3 | Click derecho en `C:\Compartido` → **Propiedades** → pestaña **Compartir** → **Uso compartido avanzado...** |
| 4 | ✅ **Compartir esta carpeta** → nombre del recurso: `Compartido` → **Permisos** |
| 5 | Selecciona **Todos** → ✅ **Permitir: Lectura** → Aceptar |
| 6 | Aceptar → **Cerrar** |
| 7 | Pestaña **Seguridad** → **Editar...** → **Agregar...** → escribe `Todos` → **Comprobar nombres** → Aceptar |
| 8 | Selecciona **Todos** → ✅ **Lectura y ejecución** → Aceptar → Aceptar |
| 9 | Anota la ruta: `\\WIN-3RVTQIDV70S\Compartido` |

> ⚠️ Si al entrar desde Windows10-1 aparece **"Windows no puede obtener acceso a \\...\Compartido"**, casi siempre falta el permiso de **Seguridad (NTFS)** del paso 7–8 (el de "Compartir" no es suficiente por sí solo). Si persiste, borra credenciales guardadas en Windows10-1 desde **Panel de control → Cuentas de usuario → Administrador de credenciales** y vuelve a intentar escribiendo explícitamente `MIGUEL\ronaldrm`.

### 4.5 Confiar en el certificado desde Windows10-1

| Paso | Click |
|---|---|
| 1 | Explorador de archivos → barra de direcciones → `\\WIN-3RVTQIDV70S\Compartido` → Enter |
| 2 | Si pide credenciales: usuario `MIGUEL\ronaldrm` → contraseña `Segura2121...` |
| 3 | Copia `Miguel.localssl.cer` al **Escritorio** de Windows10-1 |
| 4 | Doble click sobre el archivo → **Instalar certificado...** |
| 5 | ⭕ **Equipo local** → Siguiente (aceptar el control de cuentas de usuario) |
| 6 | ⭕ **Colocar todos los certificados en el siguiente almacén** → **Examinar...** |
| 7 | Selecciona **Entidades de certificación raíz de confianza** → Aceptar |
| 8 | Siguiente → **Finalizar** → Aceptar en el mensaje de importación correcta |

> ⚠️ **Si aparece "El administrador del sistema bloqueó esta aplicación"**: es una GPO de AppLocker / Directivas de restricción de software del dominio bloqueando el asistente de certificados.
>
> - **Opción 1 (rápida):** cierra sesión e inicia con una cuenta de **Administrador** (local o `MIGUEL\Administrador`) en vez del usuario estándar `ronaldrm`, y repite la importación.
> - **Opción 2 (de fondo):** en WindowsServer2022-1, quita al usuario o al equipo `Windows10-1` del **grupo de seguridad de dominio** al que está filtrada esa GPO (Ámbito → Filtrado de seguridad, en `gpmc.msc`), para que la política deje de aplicarse. Si prefieres editar la regla en vez de quitar el grupo:
>   1. `Win + R` → `gpmc.msc`
>   2. Busca la GPO que se aplica a Windows10-1 (Default Domain Policy u otra)
>   3. Click derecho → **Editar**
>   4. **Configuración del equipo → Directivas → Configuración de Windows → Configuración de seguridad → Directivas de restricción de software** (o **Directivas de control de aplicaciones → AppLocker**)
>   5. Si hay una regla que bloquea ejecutables, cámbiala a **No configurada** o agrega una regla de excepción
>   6. En Windows10-1: `gpupdate /force` y reinicia

### 🆘 Opción de respaldo: ¿no tienes el certificado `Miguel.localssl` creado?

Si por cualquier motivo no existe todavía (lo borraste, cambiaste de servidor, etc.), créalo de nuevo con el mismo comando y **el mismo FriendlyName**, para que todos los documentos sigan siendo válidos sin renombrar nada:

```powershell
New-SelfSignedCertificate -DnsName "miguel.local" -CertStoreLocation "Cert:\LocalMachine\My" -KeyUsage DigitalSignature,KeyEncipherment -FriendlyName "Miguel.localssl"
```

Luego repite los pasos 4.2 a 4.5 de esta misma sección (exportar `.pfx`, asignar a los roles, exportar `.cer`, confiar en Windows10-1).

---

## 🌐 5. Acceso por `miguel.local` (sin usar la IP)

### Registro DNS

**Server Manager → Herramientas → DNS**

| Paso | Click |
|---|---|
| 1 | En el árbol izquierdo, click en la flechita ▸ junto a `WIN-3RVTQIDV70S` para expandirlo |
| 2 | Click en la flechita ▸ junto a **Zonas de búsqueda directa** para expandirla |
| 3 | Verás la zona **`miguel.local`** → click derecho sobre ella → **Host nuevo (A o AAAA)...** |
| 4 | Nombre: (déjalo en blanco, así apunta a la raíz `miguel.local`) |
| 5 | Dirección IP: `20.13.67.10` |
| 6 | **Agregar host** → Aceptar → Listo |

### Forzar HTTPS en IIS (RD Web Access)

**Administrador de Internet Information Services (IIS)**

| Paso | Click |
|---|---|
| 1 | Conexiones → `WINSERVER2022-1` → **Sitios** → **Default Web Site** |
| 2 | Panel derecho → **Enlaces...** |
| 3 | Verifica que exista `https` puerto `443` con el certificado `Miguel.localssl` asignado |
| 4 | Selecciona el enlace `http` puerto `80` → **Quitar** → Sí |
| 5 | Cerrar |

---

## 🧪 6. Prueba final desde Windows10-1

| Paso | Click |
|---|---|
| 1 | Abre el navegador en Windows10-1 |
| 2 | Escribe `https://miguel.local/rdweb` |
| 3 | Verifica el candado 🔒 (sin advertencia de certificado) |
| 4 | Inicia sesión con `ronaldrm` / `Segura2121...` |
| 5 | Click en las apps publicadas (**Notepad**, **Calculadora**, **Word**) para abrirlas en RemoteApp |

---

## ✅ Checklist final

- [x] Rol RDS instalado
- [x] Colección de sesión creada
- [x] Apps publicadas (Notepad, Calc, Word)
- [x] Certificado `Miguel.localssl` creado por PowerShell
- [x] Certificado asignado a los roles RDS (Agente de conexión x2 + Acceso web)
- [x] Certificado confiado en Windows10-1
- [x] Registro DNS `miguel.local` apuntando al servidor RDS
- [x] Acceso probado desde `https://miguel.local/rdweb` sin advertencia
