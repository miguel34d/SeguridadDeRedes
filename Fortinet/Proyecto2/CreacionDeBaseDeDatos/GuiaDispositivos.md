# 🖥️ Configuración Completa de Servidores: Apache HTTPS y MySQL

![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04-orange?logo=ubuntu&style=for-the-badge)
![Apache](https://img.shields.io/badge/Apache-2.4_HTTPS-red?style=for-the-badge)
![MySQL](https://img.shields.io/badge/MySQL-8.0-blue?logo=mysql&style=for-the-badge)
![GitHub](https://img.shields.io/badge/GitHub-Repo-black?style=for-the-badge)

## 📋 Descripción
Este documento detalla la configuración completa desde cero de dos servidores Ubuntu en la **VLAN 20 (DMZ)**:
- **ServidorWeb** (`20.13.67.2`) - Apache con HTTPS y certificado SSL autofirmado con SAN
- **BaseDeDatos** (`20.13.67.3`) - MySQL Server con acceso remoto habilitado

---

## 🌐 PARTE 1: ServidorWeb (20.13.67.2)

### 1.1 Actualización del Sistema

```bash
sudo apt update
sudo apt upgrade -y
```

### 1.2 Instalación de Apache

```bash
sudo apt install apache2 -y
```

### 1.3 Inicio y Habilitación de Apache

```bash
sudo systemctl start apache2
sudo systemctl enable apache2
sudo systemctl status apache2
```

**Salida esperada:**
```text
● apache2.service - The Apache HTTP Server
   Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
   Active: active (running)
```

### 1.4 Creación de Página Web Personalizada

```bash
sudo nano /var/www/html/index.html
```

Contenido del archivo:
```html
<!DOCTYPE html>
<html>
<head>
    <title>ServidorWeb - VLAN 20 DMZ</title>
</head>
<body>
    <h1>¡ServidorWeb Funcionando!</h1>
    <p>Este es el servidor web en la VLAN 20 (DMZ)</p>
    <p>IP: 20.13.67.2</p>
</body>
</html>
```

Guardar con `Ctrl+O`, `Enter`, `Ctrl+X`.

### 1.5 Instalación de OpenSSL

```bash
sudo apt install openssl -y
```

### 1.6 Generación del Certificado SSL con SAN

**⚠️ IMPORTANTE:** Los navegadores modernos exigen la extensión **Subject Alternative Name (SAN)** para reconocer el certificado como válido. Sin esta extensión, el navegador mostrará "No seguro" aunque el certificado esté instalado como confiable.

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/apache-selfsigned.key \
  -out /etc/ssl/certs/apache-selfsigned.crt \
  -subj "/C=CO/ST=Bogota/L=Bogota/O=Lab/OU=IT/CN=20.13.67.2" \
  -addext "subjectAltName = IP:20.13.67.2"
```

**Parámetros explicados:**
- `-x509`: Genera un certificado autofirmado
- `-nodes`: Sin contraseña para la clave privada
- `-days 365`: Validez de 1 año
- `-newkey rsa:2048`: Clave RSA de 2048 bits
- `-subj`: Información del certificado (País, Estado, Ciudad, Organización, Unidad, Nombre Común)
- `-addext`: Extensión SAN con la IP del servidor

### 1.7 Habilitación del Módulo SSL en Apache

```bash
sudo a2enmod ssl
sudo systemctl restart apache2
```

### 1.8 Configuración del Sitio SSL

```bash
sudo nano /etc/apache2/sites-available/default-ssl.conf
```

Buscar y modificar estas líneas (alrededor de la línea 32-33):

```apache
SSLCertificateFile      /etc/ssl/certs/apache-selfsigned.crt
SSLCertificateKeyFile   /etc/ssl/private/apache-selfsigned.key
```

Guardar con `Ctrl+O`, `Enter`, `Ctrl+X`.

### 1.9 Activación del Sitio SSL

```bash
sudo a2ensite default-ssl.conf
sudo systemctl reload apache2
```

### 1.10 Verificación de Puertos

```bash
sudo ss -tlnp | grep apache
```

**Salida esperada:**
```text
LISTEN 0  511  0.0.0.0:80   0.0.0.0:*  users:(("apache2",...))
LISTEN 0  511  0.0.0.0:443  0.0.0.0:*  users:(("apache2",...))
```

✅ Apache está escuchando en los puertos 80 (HTTP) y 443 (HTTPS).

### 1.11 Prueba Local

```bash
curl -k https://localhost
```

Debería mostrar el HTML de la página web creada.

---

## 🗄️ PARTE 2: BaseDeDatos (20.13.67.3)

### 2.1 Actualización del Sistema

```bash
sudo apt update
sudo apt upgrade -y
```

### 2.2 Instalación de MySQL Server

```bash
sudo apt install mysql-server -y
```

### 2.3 Inicio y Habilitación de MySQL

```bash
sudo systemctl start mysql
sudo systemctl enable mysql
sudo systemctl status mysql
```

**Salida esperada:**
```text
● mysql.service - MySQL Community Server
   Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
   Active: active (running)
```

### 2.4 Configuración de Seguridad de MySQL

```bash
sudo mysql_secure_installation
```

**Respuestas recomendadas:**
- **Validate password component:** `N` (No)
- **Remove anonymous users:** `Y` (Sí)
- **Disallow root login remotely:** `N` (No - necesario para pruebas)
- **Remove test database:** `Y` (Sí)
- **Reload privilege tables:** `Y` (Sí)

### 2.5 Ingreso a MySQL como Root

```bash
sudo mysql -u root
```

### 2.6 Solución al Error de Política de Contraseñas (ERROR 1819)

MySQL 8.0 incluye el componente `validate_password` que exige contraseñas complejas. Para el laboratorio, relajamos la política:

```sql
-- Bajar la exigencia a LOW (solo verifica longitud)
SET GLOBAL validate_password.policy = LOW;

-- Reducir la longitud mínima a 4 caracteres
SET GLOBAL validate_password.length = 4;
```

### 2.7 Creación de Base de Datos y Usuario Remoto

```sql
-- Crear la base de datos
CREATE DATABASE lab_db;

-- Crear el usuario 'lab_user' que puede conectarse desde CUALQUIER IP ('%')
CREATE USER 'lab_user'@'%' IDENTIFIED BY 'lab123';

-- Otorgar todos los privilegios sobre 'lab_db' al nuevo usuario
GRANT ALL PRIVILEGES ON lab_db.* TO 'lab_user'@'%';

-- Aplicar los cambios y salir
FLUSH PRIVILEGES;
EXIT;
```

### 2.8 Configuración de bind-address para Acceso Remoto

Por defecto, MySQL solo escucha en `localhost` (127.0.0.1). Para que el FortiGate y la VLAN de Usuarios puedan llegar a él, debemos cambiarlo a `0.0.0.0`.

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

Buscar la línea (alrededor de la línea 31):
```ini
bind-address = 127.0.0.1
```

Cambiarla a:
```ini
bind-address = 0.0.0.0
```

**Opcional:** Comentar la línea `mysqlx-bind-address` si aparece:
```ini
#mysqlx-bind-address = 127.0.0.1
```

Guardar con `Ctrl+O`, `Enter`, `Ctrl+X`.

### 2.9 Reinicio de MySQL

```bash
sudo systemctl restart mysql
```

### 2.10 Verificación del Puerto 3306

```bash
sudo ss -tlnp | grep 3306
```

**Salida esperada:**
```text
LISTEN 0  70  127.0.0.1:33060  0.0.0.0:*  users:(("mysqld",pid=51325,fd=21))
LISTEN 0  151  0.0.0.0:3306   0.0.0.0:*  users:(("mysqld",pid=51325,fd=23))
```

✅ La línea `0.0.0.0:3306` confirma que MySQL acepta conexiones remotas desde cualquier interfaz de red.

---

## 🔄 PARTE 3: Prueba de Conectividad entre Servidores

### 3.1 Desde el ServidorWeb (20.13.67.2)

**Instalar cliente MySQL (si no está instalado):**
```bash
sudo apt install mysql-client -y
```

**Conectarse al servidor de BaseDeDatos:**
```bash
mysql -h 20.13.67.3 -u lab_user -p
```

**Contraseña:** `lab123`

**Si la conexión es exitosa, verás:**
```text
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 20
Server version: 8.0.46-0ubuntu0.22.04.4 (Ubuntu)

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| lab_db             |
+--------------------+

mysql> EXIT;
Bye
```

✅ **Conectividad entre servidores validada.**

---

##  PARTE 4: Instalación del Certificado SSL en Windows

Para que el navegador muestre el **candado de "Conexión segura"** en lugar de "No seguro", debemos instalar el certificado autofirmado en el almacén de certificados de confianza de Windows.

### 4.1 Exportación del Certificado desde el ServidorWeb

El certificado ya está generado en `/etc/ssl/certs/apache-selfsigned.crt`.

### 4.2 Descarga del Certificado a Windows

Desde **PowerShell** en tu PC Windows:

```powershell
scp miguel@20.13.67.2:/etc/ssl/certs/apache-selfsigned.crt .\servidorweb.crt
```

**Salida esperada:**
```text
apache-selfsigned.crt          100% 1318   160.9KB/s   00:00
```

### 4.3 Instalación del Certificado en Windows

1. **Abrir el archivo:** Doble clic en `servidorweb.crt`
2. **Instalar certificado:** Clic en el botón "Instalar certificado..."
3. **Almacén:** Seleccionar **"Máquina local"** (requiere permisos de administrador)
4. **Ubicación:** Elegir **"Colocar todos los certificados en el siguiente almacén"**
5. **Examinar:** Clic en "Examinar..." y seleccionar **"Entidades de certificación raíz de confianza"**
6. **Finalizar:** Aceptar → Siguiente → Finalizar → Sí a la advertencia de seguridad

### 4.4 Verificación de la Instalación

Presionar `Win + R` y escribir:
```
certlm.msc
```

Navegar a: **Entidades de certificación raíz de confianza** → **Certificados**

Deberías ver el certificado con:
- **Emitido para:** `20.13.67.2`
- **Organización:** `Lab`
- **Válido hasta:** Fecha de expiración (1 año desde la generación)

### 4.5 Prueba Final en el Navegador

1. **Cerrar completamente** el navegador (todas las ventanas)
2. **Abrir** el navegador nuevamente
3. **Navegar a:** `https://20.13.67.2`

**Resultado esperado:** 🎉 **Candado cerrado** y mensaje **"Conexión segura"**

---

## ✅ Resumen de Configuración

| Servicio | Servidor | IP | Puerto | Estado |
| :--- | :--- | :--- | :---: | :---: |
| **Apache HTTP** | ServidorWeb | `20.13.67.2` | 80 | ✅ Activo |
| **Apache HTTPS** | ServidorWeb | `20.13.67.2` | 443 | ✅ Activo |
| **MySQL Server** | BaseDeDatos | `20.13.67.3` | 3306 | ✅ Activo |

---

## 🔍 Comandos de Verificación Rápida

### ServidorWeb (20.13.67.2):
```bash
# Verificar Apache
sudo systemctl status apache2

# Verificar puertos
sudo ss -tlnp | grep -E '80|443'

# Probar localmente
curl -k https://localhost
```

### BaseDeDatos (20.13.67.3):
```bash
# Verificar MySQL
sudo systemctl status mysql

# Verificar puerto
sudo ss -tlnp | grep 3306

# Verificar usuarios y bases de datos
sudo mysql -u root -e "SHOW DATABASES; SELECT user, host FROM mysql.user;"
```

### Conectividad entre Servidores:
```bash
# Desde ServidorWeb hacia BaseDeDatos
mysql -h 20.13.67.3 -u lab_user -p
```

---

## 📊 Notas Técnicas Importantes

### Certificado SSL con SAN
Los navegadores modernos (Chrome, Edge, Firefox) **exigen** la extensión **Subject Alternative Name (SAN)** en los certificados SSL. Sin esta extensión, el navegador mostrará "No seguro" aunque el certificado esté instalado como confiable en el sistema.

**Comando correcto para generar el certificado:**
```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/apache-selfsigned.key \
  -out /etc/ssl/certs/apache-selfsigned.crt \
  -subj "/C=CO/ST=Bogota/L=Bogota/O=Lab/OU=IT/CN=20.13.67.2" \
  -addext "subjectAltName = IP:20.13.67.2"
```

### bind-address en MySQL
Por defecto, MySQL solo acepta conexiones locales (`127.0.0.1`). Para permitir conexiones remotas desde otras VLANs a través del FortiGate, es necesario cambiar `bind-address` a `0.0.0.0`.

### Política de Contraseñas en MySQL 8.0
El componente `validate_password` de MySQL 8.0 exige contraseñas complejas por defecto. Para laboratorios, se puede relajar temporalmente con:
```sql
SET GLOBAL validate_password.policy = LOW;
SET GLOBAL validate_password.length = 4;
```

---

<br>

### 👨‍💻 Realizado por: **Miguel Ramirez Meli**

![Author](https://img.shields.io/badge/Author-Miguel_Ramirez_Meli-orange?style=for-the-badge)
![Lab](https://img.shields.io/badge/Lab-FortiGate_VM64-purple?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Full_Configuration_Completed-brightgreen?style=for-the-badge)
