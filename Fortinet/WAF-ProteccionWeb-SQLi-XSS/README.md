# Evidencia — WAF en Servidor Web (WEB-1)

**Alumno:** Miguel Ramirez Meli — **Matrícula:** 20251367 — **Docente:** Jonathan Rondón — **Materia:** Seguridad de Redes — **Institución:** ITLA

## images/

1. `01-topologia.png` — Topología base del laboratorio en GNS3.
2. `02-interfaces-lan-wan.png` — Interfaces del FortiGate (Lan-Users, Lan-Servers, Salida-Internet).
3. `03-politicas-firewall-antes.png` — Políticas de firewall antes de aplicar WAF.
4. `04-feature-visibility-bloqueada.png` — Toggle de WAF bloqueado por falta de modo Proxy.
5. `05-cli-gui-proxy-inspection.png` — Habilitando `gui-proxy-inspection` y modo proxy en la política por CLI.
6. `06-feature-visibility-habilitada.png` — Toggle de Web Application Firewall ya habilitado.
7. `07-perfil-waf-signatures-100.png` — Perfil WAF con Signatures al 100% (SQLi, XSS, Generic Attacks, Known Exploits, etc.).
8. `08-perfil-waf-constraints-100.png` — Perfil WAF con Constraints al 100% (Header/URL/Method limits).
9. `09-politica-waf-aplicada.png` — Política `Usuarios-to-Servidores-HTTP` con el perfil WAF asignado en modo Proxy-based.
10. `10-prueba-bloqueo-navegador.png` — Página de bloqueo del WAF al acceder desde Kali.
11. `11-navegador-legitimo-carga-ok.png` — Acceso legítimo desde Windows10-1 cargando sin bloqueo.
12. `12-logs-waf-resumen.png` — Resumen de Security Events mostrando bloqueos por WAF.
13. `13-log-detalle-bad-robot.png` — Detalle de log: bloqueo por firma "Bad Robot" (herramientas automatizadas).
14. `14-curl-pruebas-iniciales.png` — Pruebas curl: tráfico limpio (200), SQLi sin encodear (000), XSS bloqueado (403).
15. `15-curl-sqli-encoded-bloqueado.png` — Prueba SQLi con URL correctamente codificada, bloqueada (403).
16. `16-log-detalle-xss.png` — Detalle de log: bloqueo por firma "Cross Site Scripting".
17. `17-log-detalle-sqli.png` — Detalle de log: bloqueo por firma "SQL Injection".
