![Autor](https://img.shields.io/badge/Autor-Miguel%20Ramirez%20Meli-blue)
![Matricula](https://img.shields.io/badge/Matr%C3%ADcula-20251367-blue)
![Docente](https://img.shields.io/badge/Docente-Jonathan%20Rond%C3%B3n-blue)
![Materia](https://img.shields.io/badge/Materia-Seguridad%20de%20Redes-blue)
![Institucion](https://img.shields.io/badge/Instituci%C3%B3n-ITLA-lightgrey)
![Estado](https://img.shields.io/badge/Estado-Completado-brightgreen)

# Configuración NPS (RADIUS) — Niveles de acceso 15, 1 y 0

Evidencia de laboratorio — Miguel Ramirez Meli (20251367)

## Paso 1: Topología

Router1 — Switch2 — Windows10-1 / WindowsServer2022-1, todo en VLAN 10 (`10.13.67.0/24`).

![Topología](images/01-topologia.png)

## Paso 2: WindowsServer2022-1 — Red y Active Directory (`miguel.local`)

Se asignó la IP `10.13.67.10` al servidor y se promovió a controlador de dominio, creando el bosque `miguel.local`.

![Configuración IP del WindowsServer2022-1](images/02-ip-windowsserver2022-1.png)

![Instalación del rol Servicios de dominio de Active Directory](images/03-instalacion-rol-adds.png)

![Creación del bosque y dominio miguel.local](images/04-asistente-adds-dominio-miguel-local.png)

## Paso 3: Grupos y usuarios de dominio

Se crearon los grupos `Admins-Nivel15`, `Users-Nivel1` y `Users-Nivel0`, y los usuarios `miguel`, `carlos` y `emily`, cada uno asignado a su grupo.

![Grupos de seguridad Admins-Nivel15 y Users-Nivel1 creados en AD](images/05-grupos-seguridad-creados.png)

![Creación del usuario Miguel Ramirez Meli](images/06-usuario-miguel-ramirez-meli.png)

![Configuración de contraseña para el usuario creado](images/07-password-usuario.png)

![Creación del usuario Carlos Peralta](images/08-usuario-carlos-peralta.png)

![Miguel Ramirez Meli agregado como miembro de Admins-Nivel15](images/09-miembros-admins-nivel15.png)

![Creación del usuario Emily Peralta](images/10-usuario-emily-peralta.png)

![Listado final de usuarios y grupos en el dominio](images/11-usuarios-y-grupos-dominio.png)

![Emily Peralta agregada como miembro de Users-Nivel0](images/12-miembros-users-nivel0.png)

## Paso 4: Instalación del rol NPS

Se instaló el rol Servicios de acceso y directivas de redes (NPS) en el controlador de dominio.

![Instalación del rol Servicios de acceso y directivas de redes (NPS)](images/13-instalacion-rol-nps.png)

## Paso 5: Router1 como cliente RADIUS

Se registró Router1 en NPS como cliente RADIUS con secreto compartido.

![Registro de Router1 como cliente RADIUS en NPS](images/14-cliente-radius-router1.png)

## Paso 6: Directivas de red — Nivel 15, Nivel 1 y Nivel 0

Se crearon las tres directivas de red, cada una condicionada a su grupo de dominio, con `Service-Type = NAS Prompt` y el `Cisco-AV-Pair` correspondiente (`shell:priv-lvl=15/1/0`).

![Condición de directiva: grupo MIGUEL\Admins-Nivel15](images/15-condicion-grupo-admins-nivel15.png)

![Service-Type = NAS Prompt configurado (Framed-Protocol removido)](images/16-atributos-radius-service-type-nas-prompt.png)

![Cisco-AV-Pair = shell:priv-lvl=15 agregado a Acceso-Nivel15](images/17-cisco-av-pair-nivel15.png)

![Directiva Acceso-Nivel1 finalizada (Users-Nivel1, shell:priv-lvl=1)](images/18-finalizacion-acceso-nivel1.png)

![Directiva Acceso-Nivel0 finalizada (Users-Nivel0, shell:priv-lvl=0)](images/19-finalizacion-acceso-nivel0.png)

![Las tres directivas habilitadas y en orden de procesamiento](images/20-listado-directivas-red.png)

## Paso 7: Pruebas de conectividad y verificación

Se validó el acceso SSH de los tres usuarios y su nivel de privilegio exacto, tanto por sesión SSH como por prueba directa de autenticación (`test aaa`) desde Router1.

![miguel inicia sesión en Router1 con privilegio 15](images/21-prueba-ssh-miguel-nivel15.png)

![carlos inicia sesión con privilegio restringido (no puede usar enable)](images/22-prueba-ssh-carlos-restringido.png)

![emily inicia sesión con privilegio restringido (no puede usar enable)](images/23-prueba-ssh-emily-restringido.png)

![show privilege confirma nivel 15 para miguel](images/24-show-privilege-miguel-nivel15.png)

![show privilege confirma nivel 1 para carlos](images/25-show-privilege-carlos-nivel1.png)

![En nivel 0, ni show está permitido (confirma la restricción máxima)](images/26-show-privilege-emily-nivel0-restringido.png)

![test aaa group radius confirma autenticación exitosa contra el NPS para los tres usuarios](images/27-test-aaa-radius-autenticacion.png)
