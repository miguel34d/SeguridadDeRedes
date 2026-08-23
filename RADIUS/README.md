<p align="center">
  <img src="https://img.shields.io/badge/Autor-Miguel%20Ramirez%20Meli-blue" />
  <img src="https://img.shields.io/badge/Matr%C3%ADcula-20251367-blue" />
  <img src="https://img.shields.io/badge/Docente-Jonathan%20Rond%C3%B3n-blue" />
  <img src="https://img.shields.io/badge/Materia-Seguridad%20de%20Redes-blue" />
  <img src="https://img.shields.io/badge/Instituci%C3%B3n-ITLA-lightgrey" />
  <img src="https://img.shields.io/badge/Estado-Completado-brightgreen" />
</p>

<h1 align="center">Configuración NPS (RADIUS)</h1>
<h3 align="center">Niveles de acceso 15, 1 y 0</h3>

<p align="center"><i>Evidencia de laboratorio — Miguel Ramirez Meli (20251367)</i></p>

<br>

## Paso 1 — Topología

<img src="https://img.shields.io/badge/Paso-1-blue" />

<p align="center">
  <img src="images/01-topologia.png" width="650"><br>
  <sub><i>Router1 — Switch2 — Windows10-1 / WindowsServer2022-1, VLAN 10 (10.13.67.0/24).</i></sub>
</p>

<br>

## Paso 2 — WindowsServer2022-1: Red y Active Directory (`miguel.local`)

<img src="https://img.shields.io/badge/Paso-2-blue" />

Se asignó la IP `10.13.67.10` al servidor y se promovió a controlador de dominio, creando el bosque `miguel.local`.

<p align="center">
  <img src="images/02-ip-windowsserver2022-1.png" width="500"><br>
  <sub><i>Configuración IP del WindowsServer2022-1.</i></sub>
</p>

<p align="center">
  <img src="images/03-instalacion-rol-adds.png" width="500"><br>
  <sub><i>Instalación del rol Servicios de dominio de Active Directory.</i></sub>
</p>

<p align="center">
  <img src="images/04-asistente-adds-dominio-miguel-local.png" width="500"><br>
  <sub><i>Creación del bosque y dominio miguel.local.</i></sub>
</p>

<br>

## Paso 3 — Grupos y usuarios de dominio

<img src="https://img.shields.io/badge/Paso-3-blue" />

Se crearon los grupos `Admins-Nivel15`, `Users-Nivel1` y `Users-Nivel0`, y los usuarios `miguel`, `carlos` y `emily`, cada uno asignado a su grupo.

<p align="center">
  <img src="images/05-grupos-seguridad-creados.png" width="500"><br>
  <sub><i>Grupos de seguridad Admins-Nivel15 y Users-Nivel1 creados en AD.</i></sub>
</p>

<p align="center">
  <img src="images/06-usuario-miguel-ramirez-meli.png" width="400"><br>
  <sub><i>Creación del usuario Miguel Ramirez Meli.</i></sub>
</p>

<p align="center">
  <img src="images/07-password-usuario.png" width="400"><br>
  <sub><i>Configuración de contraseña para el usuario creado.</i></sub>
</p>

<p align="center">
  <img src="images/08-usuario-carlos-peralta.png" width="400"><br>
  <sub><i>Creación del usuario Carlos Peralta.</i></sub>
</p>

<p align="center">
  <img src="images/09-miembros-admins-nivel15.png" width="400"><br>
  <sub><i>Miguel Ramirez Meli agregado como miembro de Admins-Nivel15.</i></sub>
</p>

<p align="center">
  <img src="images/10-usuario-emily-peralta.png" width="400"><br>
  <sub><i>Creación del usuario Emily Peralta.</i></sub>
</p>

<p align="center">
  <img src="images/11-usuarios-y-grupos-dominio.png" width="600"><br>
  <sub><i>Listado final de usuarios y grupos en el dominio.</i></sub>
</p>

<p align="center">
  <img src="images/12-miembros-users-nivel0.png" width="400"><br>
  <sub><i>Emily Peralta agregada como miembro de Users-Nivel0.</i></sub>
</p>

<br>

## Paso 4 — Instalación del rol NPS

<img src="https://img.shields.io/badge/Paso-4-blue" />

<p align="center">
  <img src="images/13-instalacion-rol-nps.png" width="600"><br>
  <sub><i>Instalación del rol Servicios de acceso y directivas de redes (NPS).</i></sub>
</p>

<br>

## Paso 5 — Router1 como cliente RADIUS

<img src="https://img.shields.io/badge/Paso-5-blue" />

<p align="center">
  <img src="images/14-cliente-radius-router1.png" width="600"><br>
  <sub><i>Registro de Router1 como cliente RADIUS en NPS.</i></sub>
</p>

<br>

## Paso 6 — Directivas de red: Nivel 15, Nivel 1 y Nivel 0

<img src="https://img.shields.io/badge/Paso-6-blue" />

Se crearon las tres directivas de red, cada una condicionada a su grupo de dominio, con `Service-Type = NAS Prompt` y el `Cisco-AV-Pair` correspondiente.

<p align="center">
  <img src="images/15-condicion-grupo-admins-nivel15.png" width="600"><br>
  <sub><i>Condición de directiva: grupo MIGUEL\Admins-Nivel15.</i></sub>
</p>

<p align="center">
  <img src="images/16-atributos-radius-service-type-nas-prompt.png" width="500"><br>
  <sub><i>Service-Type = NAS Prompt configurado (Framed-Protocol removido).</i></sub>
</p>

<p align="center">
  <img src="images/17-cisco-av-pair-nivel15.png" width="500"><br>
  <sub><i>Cisco-AV-Pair = shell:priv-lvl=15 agregado a Acceso-Nivel15.</i></sub>
</p>

<p align="center">
  <img src="images/18-finalizacion-acceso-nivel1.png" width="500"><br>
  <sub><i>Directiva Acceso-Nivel1 finalizada (Users-Nivel1, shell:priv-lvl=1).</i></sub>
</p>

<p align="center">
  <img src="images/19-finalizacion-acceso-nivel0.png" width="500"><br>
  <sub><i>Directiva Acceso-Nivel0 finalizada (Users-Nivel0, shell:priv-lvl=0).</i></sub>
</p>

<p align="center">
  <img src="images/20-listado-directivas-red.png" width="700"><br>
  <sub><i>Las tres directivas habilitadas y en orden de procesamiento.</i></sub>
</p>

<br>

## Paso 7 — Pruebas de conectividad y verificación

<img src="https://img.shields.io/badge/Paso-7-blue" />

Se validó el acceso SSH de los tres usuarios y su nivel de privilegio exacto, tanto por sesión SSH como por prueba directa de autenticación (`test aaa`) desde Router1.

<p align="center">
  <img src="images/21-prueba-ssh-miguel-nivel15.png" width="450"><br>
  <sub><i>miguel inicia sesión en Router1 con privilegio 15.</i></sub>
</p>

<p align="center">
  <img src="images/22-prueba-ssh-carlos-restringido.png" width="450"><br>
  <sub><i>carlos inicia sesión con privilegio restringido (no puede usar enable).</i></sub>
</p>

<p align="center">
  <img src="images/23-prueba-ssh-emily-restringido.png" width="450"><br>
  <sub><i>emily inicia sesión con privilegio restringido (no puede usar enable).</i></sub>
</p>

<p align="center">
  <img src="images/24-show-privilege-miguel-nivel15.png" width="450"><br>
  <sub><i>show privilege confirma nivel 15 para miguel.</i></sub>
</p>

<p align="center">
  <img src="images/25-show-privilege-carlos-nivel1.png" width="450"><br>
  <sub><i>show privilege confirma nivel 1 para carlos.</i></sub>
</p>

<p align="center">
  <img src="images/26-show-privilege-emily-nivel0-restringido.png" width="450"><br>
  <sub><i>En nivel 0, ni show está permitido (confirma la restricción máxima).</i></sub>
</p>

<p align="center">
  <img src="images/27-test-aaa-radius-autenticacion.png" width="600"><br>
  <sub><i>test aaa group radius confirma autenticación exitosa contra el NPS para los tres usuarios.</i></sub>
</p>
