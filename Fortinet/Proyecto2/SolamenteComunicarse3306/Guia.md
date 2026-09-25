# 🔒 Restricción: WEB-Server solo puede hablar con DB-Server (puerto 3306)

![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04-orange?logo=ubuntu&style=for-the-badge)
![UFW](https://img.shields.io/badge/UFW-Firewall_Local-blue?style=for-the-badge)

## 📋 Descripción

Requisito: el ServidorWeb (`20.13.67.2`) solo puede comunicarse con el BaseDeDatos (`20.13.67.3`), y únicamente por el puerto `3306`. Ninguna otra conexión, entrante o saliente, debe estar permitida desde el ServidorWeb.

> ⚠️ **Por qué esto se hace con `ufw` y no con una política del FortiGate:** ServidorWeb y BaseDeDatos están en la **misma VLAN** (`20.13.67.0/29`), conectados al mismo switch. Su tráfico se resuelve en capa 2 y **nunca cruza el FortiGate**, así que una política de firewall en el FortiGate no puede filtrarlo. Por eso la restricción se aplica con el firewall local (`ufw`) en cada servidor, que sí ve todo el tráfico que entra y sale de la máquina.

---

## Paso 1: instalar UFW en ambos servidores

```bash
sudo apt install ufw -y
```

Ejecutar en **ServidorWeb** y en **BaseDeDatos**.

---

## Paso 2: configurar BaseDeDatos (20.13.67.3)

Solo debe aceptar conexiones desde el ServidorWeb, y únicamente al puerto 3306.

```bash
# Permitir solo al ServidorWeb, solo al puerto 3306
sudo ufw allow from 20.13.67.2 to any port 3306 proto tcp

# Permitir SSH para no perder el acceso de administración
sudo ufw allow ssh

# Bloquear todo lo demás entrante por defecto
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Activar
sudo ufw enable
sudo ufw status verbose
```

**Salida esperada:**
```text
Status: active
Default: deny (incoming), allow (outgoing), disabled (routed)

To                         Action      From
--                         ------      ----
3306/tcp                   ALLOW IN    20.13.67.2
22/tcp                     ALLOW IN    Anywhere
```

---

## Paso 3: configurar ServidorWeb (20.13.67.2)

Debe seguir recibiendo tráfico web normal, pero solo puede **salir** hacia la base de datos en el puerto 3306.

```bash
# Tráfico entrante permitido: web y SSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow ssh

# Bloquear entrante por defecto (solo lo de arriba queda permitido)
sudo ufw default deny incoming

# Bloquear TODA salida por defecto
sudo ufw default deny outgoing

# Única excepción de salida: la base de datos, puerto 3306
sudo ufw allow out to 20.13.67.3 port 3306 proto tcp

# Activar
sudo ufw enable
sudo ufw status verbose
```

**Salida esperada:**
```text
Status: active
Default: deny (incoming), deny (outgoing), disabled (routed)

To                         Action      From
--                         ------      ----
80/tcp                     ALLOW IN    Anywhere
443/tcp                    ALLOW IN    Anywhere
22/tcp                     ALLOW IN    Anywhere
20.13.67.3 3306/tcp        ALLOW OUT   Anywhere
```

> ⚠️ **Orden importante:** la regla `allow out to ... port 3306` debe agregarse **después** de `ufw default deny outgoing`, o de lo contrario hay que verificar con `ufw status verbose` que ambas quedaron activas. Si el `nc` de prueba se queda colgado sin conectar ni fallar con mensaje claro, casi siempre significa que la regla de salida al 3306 no se aplicó.

---

## Paso 4: verificación completa

### Debe conectar (permitido)

```bash
# Desde ServidorWeb hacia BaseDeDatos, puerto 3306
nc -zv 20.13.67.3 3306
```
```text
Connection to 20.13.67.3 3306 port [tcp/mysql] succeeded!
```

### Debe fallar (bloqueado)

```bash
# Desde ServidorWeb hacia BaseDeDatos, cualquier otro puerto
nc -zv 20.13.67.3 22

# Desde ServidorWeb hacia Internet, cualquier destino
nc -zv 8.8.8.8 443

# Desde Usuarios (Kali) hacia BaseDeDatos, puerto 3306 (bloqueado por el FortiGate)
nc -zv 20.13.67.3 3306
```

Los tres deben quedarse colgados o devolver `Connection refused` / timeout.

---

## ✅ Resumen de verificación

| Origen | Destino | Puerto | Resultado esperado |
|---|---|---|---|
| ServidorWeb (`20.13.67.2`) | BaseDeDatos (`20.13.67.3`) | 3306 | ✅ Permitido |
| ServidorWeb (`20.13.67.2`) | BaseDeDatos (`20.13.67.3`) | 22 | ❌ Bloqueado |
| ServidorWeb (`20.13.67.2`) | Internet (`8.8.8.8`) | 443 | ❌ Bloqueado |
| Usuarios (VLAN 10) | BaseDeDatos (`20.13.67.3`) | 3306 | ❌ Bloqueado (política FortiGate) |

Con esto, el ServidorWeb queda completamente aislado: solo puede hablar con la base de datos, y solo en el puerto que necesita para funcionar.

---

### 👨‍💻 Realizado por: **Miguel Ramirez Meli**

![Lab](https://img.shields.io/badge/Lab-FortiGate_VM64-purple?style=for-the-badge)
![Status](https://img.shields.io/badge/Restriccion-Verificada-brightgreen?style=for-the-badge)
