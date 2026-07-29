# 🖥️ Lab: Configuración del Servicio RDP RemoteApp

![Estado](https://img.shields.io/badge/Estado-Completado-brightgreen)
![Resultado](https://img.shields.io/badge/Resultado-RemoteApp%20Funcional-brightgreen)
![Dominio](https://img.shields.io/badge/Dominio-miguel.local-orange)
![HTTPS](https://img.shields.io/badge/RD%20Web%20Access-HTTPS%20Seguro-success)

> ℹ️ Requisito previo: el dominio `miguel.local` ya debe estar creado, con `WindowsServer2022-1` como DC y `Windows10-1` unido al dominio (ver documento **Creación del Dominio**).

---

## ✅ Checklist rápido

- [ ] Rol RDS instalado
- [ ] Colección de sesión creada
- [ ] Apps publicadas (Notepad, Calc, Word)
- [ ] Certificado SSL emitido y asignado
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
| 7 | **Servicios de rol**: ✅ RD Licensing · ✅ RD Session Host · ✅ RD Connection Broker · ✅ RD Web Access |
| 8 | ✅ Reiniciar automáticamente si es necesario |
| 9 | Instalar → esperar reinicio → Cerrar |

![Servicios de rol RDS](images/servicios_rol.png)

---

## 🔧 2. Crear la colección de sesión

**Administrador del servidor → Servicios de Escritorio remoto → Colecciones**

| Paso | Click |
|---|---|
| 1 | Click derecho en **Colecciones** → **Crear colección de sesiones** |
| 2 | Siguiente (bienvenida) |
| 3 | Nombre de la colección: `ColeccionRemoteApp` → Siguiente |
| 4 | ✅ Seleccionar el servidor RD Session Host (`WINSERVER2022-1`) → Agregar → Siguiente |
| 5 | ✅ **Grupo de usuarios**: agrega `MIGUEL\ronaldrm` (o `MIGUEL\Domain Users`) → Siguiente |
| 6 | Perfiles de usuario: deja predeterminado → Siguiente |
| 7 | Revisar → **Crear** → Cerrar |

---

## 🔧 3. Publicar RemoteApps

**Administrador del servidor → Servicios de Escritorio remoto → Colecciones → `ColeccionRemoteApp`**

| Paso | Click |
|---|---|
| 1 | En **Programas RemoteApp** → **Tareas** → **Publicar programas RemoteApp** |
| 2 | ✅ Marca **Notepad**, **Calculadora**, **Word** → Siguiente |
| 3 | **Publicar** → Cerrar |

---

## 🔒 4. Certificado SSL (para HTTPS sin advertencias)

### Crear el certificado desde la propia consola RDS

**Administrador del servidor → Servicios de Escritorio remoto → Vista general → Tareas → Editar propiedades de implementación**

| Paso | Click |
|---|---|
| 1 | Pestaña **Certificados** |
| 2 | Selecciona el rol **RD Connection Broker - Publicar** → **Crear nuevo certificado** |
| 3 | Nombre del certificado: `miguel.local` |
| 4 | ✅ **Permitir que el certificado se agregue al almacén de raíz de confianza en las computadoras cliente** |
| 5 | Ubicación del archivo: `C:\Certificados\miguel-local.pfx` → Contraseña: `ExportPass123!` → Confirmar |
| 6 | Aceptar → repite los pasos 2–5 para el rol **RD Connection Broker - Autenticación de identidad web único** y para **RD Web Access** |
| 7 | Verifica que la columna **Estado** quede en ✅ para los tres roles |
| 8 | ✅ **Aceptar el certificado para este rol** en cada uno → **Aplicar** → Aceptar |

### Exportar el certificado (sin clave privada) para el cliente

**En WindowsServer2022-1 → Ejecutar (`Win + R`) → `mmc`**

| Paso | Click |
|---|---|
| 1 | Archivo → **Agregar o quitar complementos...** |
| 2 | Selecciona **Certificados** → **Agregar** |
| 3 | ⭕ **Cuenta de equipo** → Siguiente → ⭕ **Equipo local** → Finalizar → Aceptar |
| 4 | Expande **Certificados (Equipo local) → Personal → Certificados** |
| 5 | Click derecho en el certificado `miguel.local` → **Todas las tareas → Exportar...** |
| 6 | Siguiente → ⭕ **No, no exportar la clave privada** → Siguiente |
| 7 | ⭕ **X.509 codificado en Base64 (.CER)** → Siguiente |
| 8 | Nombre de archivo: `C:\Certificados\miguel-local.cer` → Siguiente → **Finalizar** |
| 9 | Copia `miguel-local.cer` a una carpeta compartida o USB para llevarlo a Windows10-1 |

### Confiar en el certificado desde Windows10-1

| Paso | Click |
|---|---|
| 1 | Copia `miguel-local.cer` a Windows10-1 y haz doble click sobre él |
| 2 | **Instalar certificado...** |
| 3 | ⭕ **Equipo local** → Siguiente (aceptar el control de cuentas de usuario) |
| 4 | ⭕ **Colocar todos los certificados en el siguiente almacén** → **Examinar...** |
| 5 | Selecciona **Entidades de certificación raíz de confianza** → Aceptar |
| 6 | Siguiente → **Finalizar** → Aceptar en el mensaje de importación correcta |

---

## 🌐 5. Acceso por `miguel.local` (sin usar la IP)

### Registro DNS

**Server Manager → Herramientas → DNS → `miguel.local` (zona)**

| Paso | Click |
|---|---|
| 1 | Click derecho en la zona `miguel.local` → **Host nuevo (A o AAAA)** |
| 2 | Nombre: (déjalo en blanco, así apunta a la raíz `miguel.local`) |
| 3 | Dirección IP: `20.13.67.10` |
| 4 | **Agregar host** → Aceptar → Listo |

### Forzar HTTPS en IIS (RD Web Access)

**Administrador de Internet Information Services (IIS)**

| Paso | Click |
|---|---|
| 1 | Conexiones → `WINSERVER2022-1` → **Sitios** → **Default Web Site** |
| 2 | Panel derecho → **Enlaces...** |
| 3 | Verifica que exista `https` puerto `443` con el certificado `miguel.local` asignado |
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
- [x] Certificado SSL emitido y asignado
- [x] Registro DNS `miguel.local` apuntando al servidor RDS
- [x] Acceso probado desde `https://miguel.local/rdweb` sin advertencia
