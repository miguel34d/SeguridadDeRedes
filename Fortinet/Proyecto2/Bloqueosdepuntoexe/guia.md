# 🚫 File Filter: Bloqueo de descargas .exe desde web

![FortiGate](https://img.shields.io/badge/FortiGate-VM64_KVM-EE3124?style=for-the-badge)
![FileFilter](https://img.shields.io/badge/File_Filter-EXE_Blocked-yellow?style=for-the-badge)

## 📋 Descripción

Requisito: agregar filtrado que bloquee descargas de archivos `.exe` desde web, tanto hacia el WEB-Server del laboratorio como en la navegación general de Usuarios hacia Internet.

En FortiGate, el bloqueo por tipo de archivo se implementa con el perfil **File Filter** (no con Application Control, que identifica aplicaciones como Facebook o YouTube, no tipos de archivo).

---

## Paso 1: activar visibilidad de la función

| Campo | Valor |
|---|---|
| System > Feature Visibility | `File Filter`: `ON` |

---

## Paso 2: crear el perfil File Filter

`Security Profiles > File Filter > Create New`

| Campo | Valor |
|---|---|
| Name | `FF-NoEXE` |
| Comments | `Bloquea descargas .exe desde WEB-Server e Internet` |
| Feature set | `Flow-based` |
| Log | `Enable` |

Dentro de `Rules > Create New`:

| Campo | Valor |
|---|---|
| Name | `HTTP` |
| Protocols | `HTTP` |
| Traffic | `Incoming` |
| Password-protected only | `Disabled` |
| File types | `exe` |
| Action | `Block` |

Guardar la regla y el perfil con `OK`.

---

## Paso 3: aplicar el perfil a las políticas correspondientes

El requisito pide bloquear `.exe` "desde web" en general, así que el perfil se aplica en **dos** políticas: la del WEB-Server del laboratorio y la de salida a Internet de Usuarios.

### 3.1 Política Usuarios → WEB-Server

`Policy & Objects > Firewall Policy > Usuarios_to_WebServer_HTTPS`

| Sección | Campo | Valor |
|---|---|---|
| Security Profiles | File Filter | `FF-NoEXE` |
| Security Profiles | SSL Inspection | `custom-deep-inspection` (ya configurado con `web-server-cert`) |

### 3.2 Política Usuarios/Servidores → Internet

`Policy & Objects > Firewall Policy > LAN_to_WAN_NAT`

| Sección | Campo | Valor |
|---|---|---|
| Security Profiles | File Filter | `FF-NoEXE` |
| Security Profiles | SSL Inspection | `deep-inspection` |

> ⚠️ **No usar `custom-deep-inspection` en esta política.** Ese perfil está en modo `Protecting SSL Server`, apuntando al certificado de `20.13.67.2`. Usarlo en tráfico hacia Internet rompería la navegación, porque el FortiGate intentaría presentar el certificado del WEB-Server para cualquier sitio externo. `deep-inspection` está en modo "Multiple Clients Connecting to Multiple Servers", correcto para tráfico saliente hacia múltiples destinos.
>
> Con `deep-inspection` activo, los navegadores de Usuarios mostrarán advertencia de certificado en cada sitio HTTPS visitado (la CA de Fortinet no está instalada como confiable). Es un comportamiento esperado en un laboratorio; se acepta el aviso del navegador para continuar.

Guardar ambas políticas con `OK`.

---

## Paso 4: crear archivos de prueba en el ServidorWeb

```bash
sudo mkdir -p /var/www/html/descargas
echo "archivo de prueba, no es un ejecutable real" | sudo tee /var/www/html/descargas/prueba.exe
echo "contenido normal" | sudo tee /var/www/html/descargas/prueba.txt
ls -la /var/www/html/descargas/
```

> ⚠️ **Importante:** las pruebas de descarga contra sitios externos "generadores de archivos" (como byterivet.com, file-examples.com y similares) **no sirven para validar este filtro**. Esos sitios crean el archivo con JavaScript dentro del propio navegador (`blob:`), sin que ningún byte viaje por la red. El File Filter solo puede inspeccionar archivos que se transfieren realmente como respuesta HTTP de un servidor, como los que se crean en este paso.

---

## Paso 5: probar el bloqueo desde Windows (Usuarios)

Desde PowerShell:

```powershell
cd C:\Users\Miguel

# Debe bloquearse
curl.exe -k -O "https://20.13.67.2/descargas/prueba.exe"

# Debe descargar normal, sin problema
curl.exe -k -O "https://20.13.67.2/descargas/prueba.txt"
dir prueba.txt
type prueba.txt
```

### Resultado esperado

| Archivo | Resultado |
|---|---|
| `prueba.exe` | `curl: (56) Recv failure: Connection was reset` — la conexión se corta, el archivo nunca llega a crearse en disco |
| `prueba.txt` | Se descarga completo, con su contenido íntegro (`contenido normal`) |

Este contraste demuestra que el bloqueo es específico al tipo de archivo `.exe`, y no una restricción general de descargas.

---

## Paso 6: verificar en los logs del FortiGate

`Log & Report > Security Events`, en la sección de `File Filter` (o dentro de `Forward Traffic`, filtrando por la hora de la prueba y destino `20.13.67.2`).

Se debe registrar:

| Campo | Valor |
|---|---|
| Filename | `prueba.exe` |
| Action | `blocked` |
| Source | IP del equipo de Usuarios |
| Destination | `20.13.67.2` |

---

## ✅ Resumen

| Requisito | Cumplido |
|---|---|
| Perfil File Filter creado (`FF-NoEXE`) | ✅ |
| Bloqueo de `.exe` vía HTTP | ✅ |
| Aplicado hacia el WEB-Server del laboratorio | ✅ |
| Aplicado hacia navegación general a Internet | ✅ |
| Verificado con archivo de control (`.txt` sí pasa) | ✅ |
| Verificado en logs del FortiGate | ✅ |

---

### 👨‍💻 Realizado por: **Miguel Ramirez Meli**

![Lab](https://img.shields.io/badge/Lab-FortiGate_VM64-purple?style=for-the-badge)
![Status](https://img.shields.io/badge/FileFilter-Funcionando-brightgreen?style=for-the-badge)
