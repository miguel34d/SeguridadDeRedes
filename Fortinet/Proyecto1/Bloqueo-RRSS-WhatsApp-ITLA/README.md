# 🚫 Bloqueo de Redes Sociales, WhatsApp e ITLA — Demostración

![Status](https://img.shields.io/badge/Estado-Funcional-success?style=flat-square)
![Autor](https://img.shields.io/badge/Autor-Miguel_Ramirez_Meli-informational?style=flat-square)
![Matricula](https://img.shields.io/badge/Matr%C3%ADcula-20251367-informational?style=flat-square)
![Docente](https://img.shields.io/badge/Docente-Jonathan_Rond%C3%B3n-lightgrey?style=flat-square)
![Materia](https://img.shields.io/badge/Materia-Seguridad_de_Redes-yellow?style=flat-square)

## Application Control

Perfil `Bloqueo-RRSS-WhatsAppCalls`: Social Media en Block, y override de `WhatsApp_VoIP.Call` en Block.

![Application Control](./images/01-application-control-perfil.png)

## DNS Filter

Perfil `Bloqueo-ITLA-Dominio`.

![Nombre del perfil DNS](./images/02-dns-filter-nombre.png)

Tabla completa: `itla.edu.do`, `*.itla.edu.do`, y los proveedores de DNS-over-HTTPS bloqueados para forzar el uso del DNS del FortiGate.

![Tabla de dominios bloqueados](./images/03-dns-filter-tabla-completa.png)

## Política de salida

`LAN-Usuarios-to-WAN` con los tres perfiles aplicados: DNS filter, Application control y SSL inspection (certificate-inspection).

![Perfiles asignados a la política](./images/04-politica-perfiles-asignados.png)

## Redes sociales bloqueadas

Facebook bloqueado por Application Control (`ERR_CONNECTION_RESET`).

![Facebook bloqueado](./images/05-facebook-bloqueado.png)

Instagram bloqueado por Application Control.

![Instagram bloqueado](./images/06-instagram-bloqueado.png)

## Dominio ITLA bloqueado

`itla.edu.do` bloqueado por el DNS Filter, mostrando el portal de bloqueo.

![itla.edu.do bloqueado](./images/07-itla-bloqueado.png)

Subdominio `campusvirtual.itla.edu.do` también bloqueado (la conexión nunca llega al sitio real).

![Subdominio ITLA bloqueado](./images/08-subdominio-itla-bloqueado.png)

## Llamadas de WhatsApp bloqueadas

Intento de llamada de voz en WhatsApp.

![Llamada de WhatsApp intentando conectar](./images/09-whatsapp-llamada-sonando.png)

La llamada falla, confirmando el bloqueo de `WhatsApp_VoIP.Call`, mientras la mensajería sigue funcionando.

![Error en la llamada de WhatsApp](./images/10-whatsapp-llamada-fallida.png)
