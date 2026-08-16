# 🚫 Bloqueo de Redes Sociales, Llamadas de WhatsApp y Dominio itla.edu.do — FortiGate

![FortiGate](https://img.shields.io/badge/FortiGate-Firewall-red?style=flat-square&logo=fortinet&logoColor=white)
![GUI](https://img.shields.io/badge/GUI-Incluido-blue?style=flat-square)
![CLI](https://img.shields.io/badge/CLI-Incluido-brightgreen?style=flat-square)
![Status](https://img.shields.io/badge/Estado-Funcional-success?style=flat-square)
![Autor](https://img.shields.io/badge/Autor-Miguel_Ramirez_Meli-informational?style=flat-square)
![Matricula](https://img.shields.io/badge/Matr%C3%ADcula-20251367-informational?style=flat-square)
![Docente](https://img.shields.io/badge/Docente-Jonathan_Rond%C3%B3n-lightgrey?style=flat-square)
![Materia](https://img.shields.io/badge/Materia-Seguridad_de_Redes-yellow?style=flat-square)

## 📑 Contenido

1. [Application Control (redes sociales + llamadas WhatsApp)](#1-application-control-redes-sociales--llamadas-whatsapp)
2. [Excepción SSL para WhatsApp](#2-excepción-ssl-para-whatsapp)
3. [DNS Filter (bloqueo de itla.edu.do y DoH)](#3-dns-filter-bloqueo-de-itlaedudo-y-doh)
4. [Aplicar perfiles a la política de salida](#4-aplicar-perfiles-a-la-política-de-salida)

---

## 1. Application Control (redes sociales + llamadas WhatsApp)

### 🖱️ GUI

1. `Security Profiles > Application Control > Create New`
2. Name: `Bloqueo-RRSS-WhatsAppCalls`
3. Categoría **Social Media** → Action: `Block`
4. En **Application and Filter Overrides** → `Create New`
5. Buscar `whatsapp`, seleccionar solo **`WhatsApp_VoIP.Call`**
6. Action: `Block`
7. Guardar el override y `OK` al perfil

### ⌨️ CLI

```bash
config application list
    edit "Bloqueo-RRSS-WhatsAppCalls"
        config entries
            edit 1
                set category 23
                set action block
            next
            edit 2
                set application 15832
                set action block
            next
        end
    next
end
```

---

## 2. Excepción SSL para WhatsApp

Sin esto, la inspección SSL rompe WhatsApp completo (no solo las llamadas), por certificate pinning.

### 🖱️ GUI

1. `Security Profiles > SSL/SSH Inspection > certificate-inspection`
2. Sección **Exempt from SSL Inspection** → `Create New`
3. Agregar: `whatsapp.com`, `*.whatsapp.com`, `web.whatsapp.com`
4. `OK`

### ⌨️ CLI

```bash
config firewall ssl-ssh-profile
    edit "certificate-inspection"
        config ssl-exempt
            edit 1
                set type fqdn
                set fqdn "whatsapp.com"
            next
            edit 2
                set type wildcard-fqdn
                set wildcard-fqdn "*.whatsapp.com"
            next
        end
    next
end
```

---

## 3. DNS Filter (bloqueo de itla.edu.do y DoH)

Bloquear también los proveedores de DNS-over-HTTPS: si no se bloquean, el navegador se salta el DNS del FortiGate y el filtro de dominio no aplica.

### 🖱️ GUI

1. `Security Profiles > DNS Filter > Create New`
2. Name: `Bloqueo-ITLA-Dominio`
3. Activar **Static Domain Filter**
4. Agregar las siguientes entradas (`Create New` para cada una), todas con Action `Redirect to Block Portal`:

| Domain | Type |
|--------|------|
| `itla.edu.do` | Simple |
| `*.itla.edu.do` | Wildcard |
| `cloudflare-dns.com` | Simple |
| `*.cloudflare-dns.com` | Wildcard |
| `dns.google` | Simple |
| `mozilla.cloudflare-dns.com` | Simple |

5. Activar **"Allow DNS requests when a rating error occurs"**
6. `OK`

### ⌨️ CLI

```bash
config dnsfilter domain-filter
    edit 1
        set name "itla.edu.do"
        set domain "itla.edu.do"
        set type simple
        set action block
    next
    edit 2
        set name "*.itla.edu.do"
        set domain "*.itla.edu.do"
        set type wildcard
        set action block
    next
    edit 3
        set name "cloudflare-dns.com"
        set domain "cloudflare-dns.com"
        set type simple
        set action block
    next
    edit 4
        set name "*.cloudflare-dns.com"
        set domain "*.cloudflare-dns.com"
        set type wildcard
        set action block
    next
    edit 5
        set name "dns.google"
        set domain "dns.google"
        set type simple
        set action block
    next
    edit 6
        set name "mozilla.cloudflare-dns.com"
        set domain "mozilla.cloudflare-dns.com"
        set type simple
        set action block
    next
end

config dnsfilter profile
    edit "Bloqueo-ITLA-Dominio"
        set allow-dns-when-rating-error enable
        set domain-filter-table 1 2 3 4 5 6
    next
end
```

Aplicar el filtro directamente al DNS Service on Interface (no solo a la política de tráfico):

```bash
config system dns-server
    edit "port2"
        set mode recursive
        set dns-filter-profile "Bloqueo-ITLA-Dominio"
    next
end
```

---

## 4. Aplicar perfiles a la política de salida

### 🖱️ GUI

1. `Policy & Objects > Firewall Policy > LAN-Usuarios-to-WAN`
2. Application control: `Bloqueo-RRSS-WhatsAppCalls`
3. DNS filter: `Bloqueo-ITLA-Dominio`
4. SSL inspection: `certificate-inspection`
5. `OK`

### ⌨️ CLI

```bash
config firewall policy
    edit <ID-LAN-Usuarios-to-WAN>
        set application-list "Bloqueo-RRSS-WhatsAppCalls"
        set dnsfilter-profile "Bloqueo-ITLA-Dominio"
        set ssl-ssh-profile "certificate-inspection"
    next
end
```

---

## ✅ Verificación

| Prueba | Acción | Resultado esperado |
|--------|--------|---------------------|
| Redes sociales | Navegar a `facebook.com`, `instagram.com` | `ERR_CONNECTION_RESET` |
| WhatsApp mensajería | Enviar mensaje en `web.whatsapp.com` | Funciona |
| Llamadas de WhatsApp | Llamada de voz/video | Falla |
| Dominio ITLA | `nslookup itla.edu.do` | Block Portal |
| Subdominio ITLA | `nslookup campusvirtual.itla.edu.do` | Block Portal |
| Caché DNS del FortiGate | Si un dominio ya bloqueado sigue resolviendo | `diagnose test application dnsproxy 6` |
