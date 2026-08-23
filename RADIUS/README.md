# Configuración NPS (RADIUS) — Niveles de acceso 15 y 1

Evidencia de laboratorio — Miguel Ramirez Meli (20251367)

## Imágenes

1. `01-topologia.png` — Topología del laboratorio (Router1 – Switch2 – Windows10-1 / WindowsServer2022-1).
2. `02-ip-windowsserver2022-1.png` — Configuración IP del WindowsServer2022-1 (10.13.67.10).
3. `03-instalacion-rol-adds.png` — Instalación del rol Servicios de dominio de Active Directory.
4. `04-asistente-adds-dominio-miguel-local.png` — Creación del bosque y dominio `miguel.local`.
5. `05-grupos-seguridad-creados.png` — Grupos de seguridad `Admins-Nivel15` y `Users-Nivel1` creados en AD.
6. `06-usuario-miguel-ramirez-meli.png` — Creación del usuario Miguel Ramirez Meli.
7. `07-password-usuario.png` — Configuración de contraseña para el usuario creado.
8. `08-usuario-carlos-peralta.png` — Creación del usuario Carlos Peralta.
9. `09-miembros-admins-nivel15.png` — Miguel Ramirez Meli agregado como miembro de `Admins-Nivel15`.
10. `10-usuario-emily-peralta.png` — Creación del usuario Emily Peralta.
11. `11-usuarios-y-grupos-dominio.png` — Listado final de usuarios y grupos en el dominio.
12. `12-miembros-users-nivel0.png` — Emily Peralta agregada como miembro de `Users-Nivel0`.
13. `13-instalacion-rol-nps.png` — Instalación del rol Servicios de acceso y directivas de redes (NPS).
14. `14-cliente-radius-router1.png` — Registro de Router1 como cliente RADIUS en NPS (secreto compartido configurado).
15. `15-condicion-grupo-admins-nivel15.png` — Condición de directiva: grupo `MIGUEL\Admins-Nivel15`.
16. `16-atributos-radius-service-type-nas-prompt.png` — Atributo `Service-Type = NAS Prompt` configurado (Framed-Protocol removido).
17. `17-cisco-av-pair-nivel15.png` — Atributo `Cisco-AV-Pair = shell:priv-lvl=15` agregado a Acceso-Nivel15.
18. `18-finalizacion-acceso-nivel1.png` — Directiva `Acceso-Nivel1` finalizada (grupo `Users-Nivel1`, `shell:priv-lvl=1`).
19. `19-finalizacion-acceso-nivel0.png` — Directiva `Acceso-Nivel0` finalizada (grupo `Users-Nivel0`, `shell:priv-lvl=0`).
20. `20-listado-directivas-red.png` — Las tres directivas (`Acceso-Nivel15`, `Acceso-Nivel1`, `Acceso-Nivel0`) habilitadas y en orden.
21. `21-prueba-ssh-miguel-nivel15.png` — Prueba de conectividad: `miguel` inicia sesión en Router1 con privilegio 15.
22. `22-prueba-ssh-carlos-restringido.png` — Prueba de conectividad: `carlos` inicia sesión con privilegio restringido (no puede usar `enable`).
23. `23-prueba-ssh-emily-restringido.png` — Prueba de conectividad: `emily` inicia sesión con privilegio restringido (no puede usar `enable`).
24. `24-show-privilege-miguel-nivel15.png` — `show privilege` confirma nivel 15 para `miguel`.
25. `25-show-privilege-carlos-nivel1.png` — `show privilege` confirma nivel 1 para `carlos`.
26. `26-show-privilege-emily-nivel0-restringido.png` — En nivel 0, ni `show` está permitido (confirma la restricción máxima).
27. `27-test-aaa-radius-autenticacion.png` — `test aaa group radius` confirma autenticación exitosa contra el NPS para los tres usuarios.
