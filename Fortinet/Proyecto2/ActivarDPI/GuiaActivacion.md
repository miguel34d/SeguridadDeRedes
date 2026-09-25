# 🔍 Activar DPI (Deep Packet Inspection) en FortiGate

![FortiGate](https://img.shields.io/badge/FortiGate-VM64_KVM-EE3124?style=for-the-badge)

## 📋 Descripción

El DPI permite que el FortiGate descifre e inspeccione el tráfico HTTPS hacia el ServidorWeb, para poder aplicar después IPS (detección de SQL Injection) y File Filter (bloqueo de `.exe`). Sin este paso, el FortiGate ve el tráfico HTTPS como una caja cerrada y no puede aplicar esas reglas dentro de la conexión cifrada.

---

## Paso 1: activar visibilidad de funciones

| Campo | Valor | Para qué sirve |
|---|---|---|
| SSL/SSH Inspection | `ON` | Habilita el menú de inspección profunda |
| Intrusion Prevention | `ON` | Habilita el motor IPS |
| Application Control | `ON` | Habilita control de aplicaciones y archivos |

### GUI
| Paso | Menú | Acción |
|---|---|---|
| 1 | `System > Feature Visibility` | Activar `SSL/SSH Inspection`, `Intrusion Prevention`, `Application Control` |
| 2 | Misma pantalla | `Apply` |

### CLI
```
config system settings
    set gui-ips-db enable
end
```

---

## Paso 2: generar el certificado del ServidorWeb (con SAN y CA:FALSE)

En el ServidorWeb (`20.13.67.2`):

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/apache-selfsigned.key \
  -out /etc/ssl/certs/apache-selfsigned.crt \
  -subj "/C=CO/ST=Bogota/L=Bogota/O=Lab/OU=IT/CN=20.13.67.2" \
  -addext "subjectAltName = IP:20.13.67.2" \
  -addext "basicConstraints=critical,CA:FALSE" \
  -addext "keyUsage=digitalSignature,keyEncipherment" \
  -addext "extendedKeyUsage=serverAuth"
```

Copiar los archivos hacia el equipo desde donde administras el FortiGate:

```bash
sudo chmod 644 /etc/ssl/private/apache-selfsigned.key
scp miguel@20.13.67.2:/etc/ssl/certs/apache-selfsigned.crt ~/web.crt
scp miguel@20.13.67.2:/etc/ssl/private/apache-selfsigned.key ~/web.key
sudo chmod 600 /etc/ssl/private/apache-selfsigned.key   # de vuelta en el servidor
```

---

## Paso 3: importar el certificado en el FortiGate

| Campo | Valor |
|---|---|
| Type | `Certificate` |
| Certificate file | `web.crt` |
| Key file | `web.key` |
| Password | vacío |
| Certificate name | `web-server-cert` |

### GUI
| Paso | Menú | Acción |
|---|---|---|
| 1 | `System > Certificates > Create/Import` | Elegir `Import Certificate` |
| 2 | Certificate Details | Type `Certificate`, subir `web.crt` y `web.key`, nombre `web-server-cert` |
| 3 | Create Certificate | `Create` |
| 4 | `System > Certificates` | Confirmar que aparece en la sección `Local Certificate` con `Status: Valid` |

---

## Paso 4: configurar el perfil SSL/SSH Inspection

| Campo | Valor | Para qué sirve |
|---|---|---|
| Enable SSL inspection of | `Protecting SSL Server` | El FortiGate se hace pasar por el ServidorWeb usando su certificado real |
| Inspection method | `Full SSL Inspection` | Descifra el tráfico para poder inspeccionarlo |
| Server certificate | `web-server-cert` | El importado en el paso 3 |
| Malicious certificates | `Block` | Bloquea certificados marcados como maliciosos |
| Untrusted SSL certificates | `Block` | Bloquea certificados no confiables |
| HTTPS (Protocol Port Mapping) | `443`, activado | Puerto que se inspecciona |

### GUI
| Paso | Menú | Acción |
|---|---|---|
| 1 | `Security Profiles > SSL/SSH Inspection` | Seleccionar un perfil existente (ej. `custom-deep-inspection`) |
| 2 | Misma pantalla | `Enable SSL inspection of`: `Protecting SSL Server` |
| 3 | Misma pantalla | `Server certificate`: `web-server-cert` |
| 4 | Misma pantalla | `HTTPS`: puerto `443`, activado |
| 5 | Misma pantalla | `OK` |

### CLI
```
config firewall ssl-ssh-profile
    edit "custom-deep-inspection"
        set server-cert-mode replace
        set server-cert "web-server-cert"
        config https
            set ports 443
            set status deep-inspection
        end
    next
end
```

---

## Paso 5: aplicar el perfil a la política de firewall

| Campo | Valor |
|---|---|
| Política | `Usuarios_to_WebServer_HTTPS` (vlan10 → vlan20, servicio HTTPS) |
| SSL Inspection | `custom-deep-inspection` |
| Log Allowed Traffic | `All Sessions` |

### GUI
| Paso | Menú | Acción |
|---|---|---|
| 1 | `Policy & Objects > Firewall Policy` | Abrir la política de Usuarios hacia WEB-Server |
| 2 | Security Profiles | `SSL Inspection`: `custom-deep-inspection` |
| 3 | Logging Options | `Log Allowed Traffic`: `All Sessions` |
| 4 | Misma pantalla | `OK` |

### CLI
```
config firewall policy
    edit <ID_DE_LA_POLITICA>
        set ssl-ssh-profile "custom-deep-inspection"
        set logtraffic all
    next
end
```

---

## Verificación

| Qué probar | Cómo |
|---|---|
| El certificado quedó bien clasificado | `System > Certificates`, buscar `web-server-cert` bajo `Local Certificate` |
| El tráfico HTTPS pasa por DPI | Desde Usuarios: `curl -k "https://20.13.67.2/buscar.php?q=teclado"` |
| El log confirma inspección profunda | `Log & Report > Forward Traffic`, revisar el detalle de la sesión (debe indicar `deep-inspection`, no solo `certificate-inspection`) |

Con el DPI activo, el siguiente paso es crear el sensor IPS que detecte SQL Injection sobre este mismo tráfico ya descifrado.

---

### 👨‍💻 Realizado por: **Miguel Ramirez Meli**

![Lab](https://img.shields.io/badge/Lab-FortiGate_VM64-purple?style=for-the-badge)
![Status](https://img.shields.io/badge/DPI-Activado-brightgreen?style=for-the-badge)
