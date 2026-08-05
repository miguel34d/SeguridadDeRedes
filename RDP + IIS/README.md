# 🖥️ Evidencia: RemoteApp · RemoteApp Web Client · Página Personalizada IIS

![Estado](https://img.shields.io/badge/Estado-Completado-brightgreen)
![RemoteApp](https://img.shields.io/badge/RemoteApp-Funcional-brightgreen)
![Web Client](https://img.shields.io/badge/Web%20Client-Funcional-brightgreen)
![IIS](https://img.shields.io/badge/P%C3%A1gina%20IIS-Funcional-brightgreen)
![Dominio](https://img.shields.io/badge/Dominio-miguel.local-orange)
![Simulador](https://img.shields.io/badge/Simulador-GNS3-blue)
![Estudiante](https://img.shields.io/badge/Estudiante-Miguel%20Ramirez%20%7C%202025--1367-9cf)

> Documento demostrativo — evidencia de que los tres servicios fueron configurados y probados exitosamente sobre el dominio `miguel.local`. No es una guía de instalación; muestra la configuración final en el servidor y el resultado real desde el cliente **Windows10-1**.

---

## 📋 Resumen

| Servicio | Estado | Evidencia |
|---|---|---|
| RDP RemoteApp | ✅ Funcional | [Ir a sección](#1-rdp-remoteapp) |
| RDP RemoteApp Web Client | ✅ Funcional | [Ir a sección](#2-rdp-remoteapp-web-client) |
| Página personalizada de IIS | ✅ Funcional | [Ir a sección](#3-página-personalizada-de-iis) |

---

## 1. RDP RemoteApp

### 1.1 Configuración en el servidor

**Figura 1** — *Información general de la implementación de Servicios de Escritorio remoto, con los roles instalados (Agente de conexión, Host de sesión, Puerta de enlace y Acceso web) sobre `WIN-GLS36OR7EQI.miguel.local`.*

![Implementación RDS](images/img01.png)

**Figura 2** — *Colección de sesiones `ColeccionRemoteApp`, con los programas RemoteApp publicados (Bloc de notas y Calculadora) y el servidor host asociado.*

![Colección de sesiones](images/img02.png)

### 1.2 Demostración desde el cliente (Windows10-1)

**Figura 3** — *Inicio de sesión en el portal de Acceso web de RD (`https://miguel.local/RDWeb`) desde Windows10-1.*

![Login RD Web](images/img03.png)

**Figura 4** — *Portal tras iniciar sesión, mostrando las apps publicadas: Bloc de notas y Calculadora.*

![Apps publicadas](images/img04.png)

**Figura 5** — *Inicio de la app RemoteApp "Bloc de notas", con la sesión remota configurándose.*

![Iniciando RemoteApp](images/img05.png)

**Figura 6** — *Bloc de notas corriendo como ventana individual en el cliente, confirmando la conexión activa a `WIN-GLS36OR7EQI.miguel.local`.*

![RemoteApp funcionando](images/img06.png)

---

## 2. RDP RemoteApp Web Client

### 2.1 Acceso al portal Web Client

**Figura 7** — *Inicio de sesión del Cliente web del Escritorio remoto (`https://miguel.local/RDWeb/webclient/`) desde el navegador del cliente.*

![Login Web Client](images/img07.png)

**Figura 8** — *Panel "Work Resources" del RD Web Client, mostrando las apps disponibles (Bloc de notas y Calculadora).*

![Panel Work Resources](images/img08.png)

### 2.2 Demostración de funcionamiento

**Figura 9** — *Conexión e inicio de la app "Calculadora" directamente dentro del navegador, sin descarga de archivos.*

![Iniciando Calculadora en navegador](images/img09.png)

**Figura 10** — *Bloc de notas ejecutándose embebido dentro de la pestaña del navegador vía RD Web Client (HTML5), confirmando el funcionamiento del servicio.*

![RemoteApp embebido en navegador](images/img10.png)

---

## 3. Página Personalizada de IIS

### 3.1 Resultado desde el cliente

**Figura 11** — *Página personalizada de IIS (`https://miguel.local`) mostrando la identificación del portal "ITLA / Seguridad de Redes" y los datos del usuario (Miguel Ramírez, matrícula 2025-1367).*

![Página personalizada IIS](images/img11.png)

**Figura 12** — *Sección "Accesos" de la página personalizada, con enlaces configurados como página de inicio del servidor, incluyendo el acceso directo al RD Web Client del laboratorio.*

![Sección de accesos](images/img12.png)

---

## ✅ Conclusión

Las evidencias anteriores confirman que los tres servicios —**RDP RemoteApp**, **RDP RemoteApp Web Client** y la **página personalizada de IIS**— fueron configurados correctamente sobre el dominio `miguel.local` y se encuentran plenamente funcionales: el usuario de dominio puede acceder y ejecutar las aplicaciones publicadas tanto mediante archivos `.rdp` como directamente desde el navegador, además de contar con un portal de acceso centralizado y personalizado.
