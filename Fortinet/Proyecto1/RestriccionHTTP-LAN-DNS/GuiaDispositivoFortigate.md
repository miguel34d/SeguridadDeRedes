# 🔒 Solo HTTP (Usuarios → Servidores) + DNS Local — FortiGate

![FortiGate](https://img.shields.io/badge/FortiGate-Firewall-red?style=flat-square&logo=fortinet&logoColor=white)
![GUI](https://img.shields.io/badge/GUI-Incluido-blue?style=flat-square)
![CLI](https://img.shields.io/badge/CLI-Incluido-brightgreen?style=flat-square)
![Status](https://img.shields.io/badge/Estado-Funcional-success?style=flat-square)
![Autor](https://img.shields.io/badge/Autor-Miguel_Ramirez_Meli-informational?style=flat-square)
![Matricula](https://img.shields.io/badge/Matr%C3%ADcula-20251367-informational?style=flat-square)
![Docente](https://img.shields.io/badge/Docente-Jonathan_Rond%C3%B3n-lightgrey?style=flat-square)
![Materia](https://img.shields.io/badge/Materia-Seguridad_de_Redes-yellow?style=flat-square)

Objetivo: permitir **solo tráfico HTTP** desde la LAN de Usuarios (port2) hacia la LAN de Servidores (port3), bloquear todo lo demás, y publicar una **resolución DNS local** (`miguel.edu.do` → `20.13.67.10`) directamente en el FortiGate — equivalente a los comandos Cisco `ip host`, `ip dns server` e `ip domain lookup`.

## 📑 Contenido

1. [Configuración de WEB-1 (servidor HTTP legítimo)](#1-configuración-de-web-1-servidor-http-legítimo)
2. [Política: permitir HTTP](#2-política-permitir-http)
3. [Política: bloquear todo lo demás](#3-política-bloquear-todo-lo-demás)
4. [DNS local (equivalente a `ip host`)](#4-dns-local-equivalente-a-ip-host)
5. [Habilitar el FortiGate como DNS server para la LAN](#5-habilitar-el-fortigate-como-dns-server-para-la-lan)

---

## 1. Configuración de WEB-1 (servidor HTTP legítimo)

IP estática en la LAN de Servidores y servidor HTTP básico sirviendo un `index.html`.

### ⌨️ CLI (WEB-1, Linux)

```bash
sudo ip addr add 20.13.67.10/24 dev eth0
sudo ip route add default via 20.13.67.1

mkdir ~/web_legitimo
cd ~/web_legitimo
nano index.html
```

Contenido de `index.html` (ejemplo mínimo):

```html
<html>
  <body>
    <h1>WEB-1 - Servidor legitimo</h1>
  </body>
</html>
```

Levantar el servidor en el puerto 80:

```bash
cd ~/web_legitimo
sudo python3 -m http.server 80
```

---

## 2. Política: permitir HTTP

### 🖱️ GUI

1. `Policy & Objects > Firewall Policy > Create New`
2. Name: `Usuarios-to-Servidores-HTTP`
3. Incoming Interface: `port2` (Lan-Users)
4. Outgoing Interface: `port3` (Lan-Servers)
5. Source: `all`
6. Destination: `all`
7. Schedule: `always`
8. Service: quitar `ALL`, agregar solo `HTTP`
9. Action: `ACCEPT`
10. NAT: **OFF**
11. `OK`

### ⌨️ CLI

```bash
config firewall policy
    edit 0
        set name "Usuarios-to-Servidores-HTTP"
        set srcintf "port2"
        set dstintf "port3"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "HTTP"
        set nat disable
        set logtraffic all
    next
end
```

---

## 3. Política: bloquear todo lo demás

### 🖱️ GUI

1. `Create New`
2. Name: `Usuarios-to-Servidores-DENY`
3. Incoming Interface: `port2`
4. Outgoing Interface: `port3`
5. Source: `all` — Destination: `all`
6. Schedule: `always` — Service: `ALL`
7. Action: `DENY`
8. Log Violation Traffic: **ON**
9. `OK`
10. Verifica el orden: `Usuarios-to-Servidores-HTTP` debe ir **arriba** de `Usuarios-to-Servidores-DENY`

### ⌨️ CLI

```bash
config firewall policy
    edit 0
        set name "Usuarios-to-Servidores-DENY"
        set srcintf "port2"
        set dstintf "port3"
        set srcaddr "all"
        set dstaddr "all"
        set action deny
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
end

! Asegurar el orden correcto (HTTP antes que DENY)
config firewall policy
    move <ID-politica-HTTP> before <ID-politica-DENY>
end
```

---

## 4. DNS local (equivalente a `ip host`)

Equivalente FortiGate de:
```
ip host miguel.edu.do 20.13.67.10
```

### 🖱️ GUI

1. `Network > DNS Servers > DNS Database`
2. `Create New`
3. Type: `Primary DNS Zone`
4. DNS Zone: `miguel.edu.do`
5. Domain name: `miguel.edu.do`
6. View: `Shadow` (o `Public` si lo necesitas fuera de la LAN)
7. En **DNS Entries**, `Create New`:
   - Type: `Address (A)`
   - Hostname: `miguel.edu.do` (o `www` si prefieres subdominio)
   - IP Address: `20.13.67.10`
8. `OK`

### ⌨️ CLI

```bash
config system dns-database
    edit "miguel.edu.do"
        set domain "miguel.edu.do"
        set type primary
        set view shadow
        config dns-entry
            edit 1
                set hostname "miguel.edu.do"
                set type A
                set ip 20.13.67.10
            next
        end
    next
end
```

---

## 5. Habilitar el FortiGate como DNS server para la LAN

Equivalente a `ip dns server` + `ip domain lookup` — permite que los clientes de la LAN usen el FortiGate para resolver nombres.

### 🖱️ GUI

**DNS Service on Interface**
1. `Network > DNS Servers > DNS Service on Interface > Create New`
2. Interface: `Lan-Users (port2)`
3. Mode: `Recursive` (resuelve la zona local y reenvía todo lo demás a los DNS externos)
4. `OK`

**DNS del DHCP en Port2 (el orden importa)**
1. `Network > Interfaces > Edit port2`
2. En **DHCP Server > DNS server**, cambia a `Specify`
3. DNS server 1: `10.13.67.1` (el FortiGate — debe ir **primero**)
4. DNS server 2: `8.8.8.8` (respaldo)
5. `OK`

> ⚠️ Si el FortiGate queda como servidor secundario, el cliente consulta primero a `8.8.8.8`, que no conoce `miguel.edu.do` y la resolución local no funciona. El FortiGate debe ser el DNS server 1.

### ⌨️ CLI

```bash
config system dns-server
    edit "port2"
        set mode recursive
    next
end

config system dhcp server
    edit <id-dhcp-port2>
        set dns-service specify
        set dns-server1 10.13.67.1
        set dns-server2 8.8.8.8
    next
end
```

---

## ✅ Verificación

Desde Windows10-1, renovar IP para tomar el nuevo DNS del DHCP:

```
ipconfig /release
ipconfig /renew
```

| Prueba | Comando | Resultado esperado |
|--------|---------|---------------------|
| Resolución DNS local | `nslookup miguel.edu.do` | Responde `20.13.67.10` |
| Internet sigue funcionando | `ping google.com` | Respuesta exitosa (forward recursivo a DNS externos) |
| HTTP permitido | `curl http://miguel.edu.do` | Respuesta exitosa |
| HTTPS bloqueado | `curl https://miguel.edu.do` | Falla (timeout / rechazado) — solo HTTP está permitido |
| Ping bloqueado | `ping 20.13.67.10` | Timeout / sin respuesta |
| Log de bloqueo | `Log & Report > Forward Traffic` | Sesiones al puerto 443 con acción `Deny` desde port2 a port3 |
