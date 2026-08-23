![Autor](https://img.shields.io/badge/Autor-Miguel%20Ramirez%20Meli-blue)
![Matricula](https://img.shields.io/badge/Matr%C3%ADcula-20251367-blue)
![Docente](https://img.shields.io/badge/Docente-Jonathan%20Rond%C3%B3n-blue)
![Materia](https://img.shields.io/badge/Materia-Seguridad%20de%20Redes-blue)
![Institucion](https://img.shields.io/badge/Instituci%C3%B3n-ITLA-lightgrey)
![Estado](https://img.shields.io/badge/Estado-Completado-brightgreen)

# Configuración NPS (RADIUS) — Niveles de acceso 15, 1 y 0

Evidencia de laboratorio — **Miguel Ramirez Meli** (20251367)

---

## 1. Topología

Router1 — Switch2 — Windows10-1 / WindowsServer2022-1, todo en VLAN 10 (`10.13.67.0/24`).

![Topología](images/01-topologia.png)
*Topología del laboratorio.*

---

## 2. WindowsServer2022-1 — Red y Active Directory (`miguel.local`)

Se asignó la IP `10.13.67.10` al servidor y se promovió a controlador de dominio, creando el bosque `miguel.local`.

![IP Server](images/02-ip-windowsserver2022-1.png)
*Configuración IP del WindowsServer2022-1.*

![Rol AD DS](images/03-instalacion-rol-adds.png)
*Instalación del rol Servicios de dominio de Active Directory.*

![Dominio miguel.local](images/04-asistente-adds-dominio-miguel-local.png)
*Creación del bosque y dominio `miguel.local`.*

---

## 3. Grupos y usuarios de dominio

Se crearon los grupos `Admins-Nivel15`, `Users-Nivel1` y `Users-Nivel0`, y los usuarios `miguel`, `carlos` y `emily`, asignando cada uno a su grupo correspondiente.

![Grupos creados](images/05-grupos-seguridad-creados.png)
*Grupos de seguridad `Admins-Nivel15` y `Users-Nivel1` creados en AD.*

![Usuario miguel](images/06-usuario-miguel-ramirez-meli.png)
*Creación del usuario Miguel Ramirez Meli.*

![Contraseña](images/07-password-usuario.png)
*Configuración de contraseña para el usuario creado.*

![Usuario carlos](images/08-usuario-carlos-peralta.png)
*Creación del usuario Carlos Peralta.*

![Miembro Nivel15](images/09-miembros-admins-nivel15.png)
*Miguel Ramirez Meli agregado como miembro de `Admins-Nivel15`.*

![Usuario emily](images/10-usuario-emily-peralta.png)
*Creación del usuario Emily Peralta.*

![Usuarios y grupos](images/11-usuarios-y-grupos-dominio.png)
*Listado final de usuarios y grupos en el dominio.*

![Miembro Nivel0](images/12-miembros-users-nivel0.png)
*Emily Peralta agregada como miembro de `Users-Nivel0`.*

---

## 4. Instalación del rol NPS

Se instaló el rol Servicios de acceso y directivas de redes (NPS) en el controlador de dominio.

![Rol NPS](images/13-instalacion-rol-nps.png)
*Instalación del rol Servicios de acceso y directivas de redes (NPS).*

---

## 5. Router1 como cliente RADIUS

Se registró Router1 en NPS como cliente RADIUS con secreto compartido.

![Cliente RADIUS](images/14-cliente-radius-router1.png)
*Registro de Router1 como cliente RADIUS en NPS.*

---

## 6. Directivas de red — Nivel 15, Nivel 1 y Nivel 0

Se crearon las tres directivas de red, cada una condicionada a su grupo de dominio, con el atributo `Service-Type = NAS Prompt` y el `Cisco-AV-Pair` correspondiente (`shell:priv-lvl=15/1/0`).

![Condición Nivel15](images/15-condicion-grupo-admins-nivel15.png)
*Condición de directiva: grupo `MIGUEL\Admins-Nivel15`.*

![Service-Type NAS Prompt](images/16-atributos-radius-service-type-nas-prompt.png)
*Atributo `Service-Type = NAS Prompt` configurado (Framed-Protocol removido).*

![Cisco-AV-Pair Nivel15](images/17-cisco-av-pair-nivel15.png)
*Atributo `Cisco-AV-Pair = shell:priv-lvl=15` agregado a Acceso-Nivel15.*

![Finalización Nivel1](images/18-finalizacion-acceso-nivel1.png)
*Directiva `Acceso-Nivel1` finalizada (grupo `Users-Nivel1`, `shell:priv-lvl=1`).*

![Finalización Nivel0](images/19-finalizacion-acceso-nivel0.png)
*Directiva `Acceso-Nivel0` finalizada (grupo `Users-Nivel0`, `shell:priv-lvl=0`).*

![Listado directivas](images/20-listado-directivas-red.png)
*Las tres directivas habilitadas y en orden de procesamiento.*

---

## 7. Pruebas de conectividad y verificación

Se validó el acceso SSH de los tres usuarios y su nivel de privilegio exacto, tanto por sesión SSH como por prueba directa de autenticación (`test aaa`) desde el propio Router1.

![SSH miguel](images/21-prueba-ssh-miguel-nivel15.png)
*`miguel` inicia sesión en Router1 con privilegio 15.*

![SSH carlos](images/22-prueba-ssh-carlos-restringido.png)
*`carlos` inicia sesión con privilegio restringido (no puede usar `enable`).*

![SSH emily](images/23-prueba-ssh-emily-restringido.png)
*`emily` inicia sesión con privilegio restringido (no puede usar `enable`).*

![show privilege miguel](images/24-show-privilege-miguel-nivel15.png)
*`show privilege` confirma nivel 15 para `miguel`.*

![show privilege carlos](images/25-show-privilege-carlos-nivel1.png)
*`show privilege` confirma nivel 1 para `carlos`.*

![show privilege emily](images/26-show-privilege-emily-nivel0-restringido.png)
*En nivel 0, ni `show` está permitido (confirma la restricción máxima).*

![test aaa radius](images/27-test-aaa-radius-autenticacion.png)
*`test aaa group radius` confirma autenticación exitosa contra el NPS para los tres usuarios.*
