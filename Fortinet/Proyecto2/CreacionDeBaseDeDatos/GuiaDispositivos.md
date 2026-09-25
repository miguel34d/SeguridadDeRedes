# 🖥️ Configuración de ServidorWeb y BaseDeDatos

![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04-orange?logo=ubuntu&style=for-the-badge)
![Apache](https://img.shields.io/badge/Apache-2.4_HTTPS-red?style=for-the-badge)
![MySQL](https://img.shields.io/badge/MySQL-8.0-blue?logo=mysql&style=for-the-badge)

## 📋 Descripción

Configuración de los dos servidores de la VLAN 20 (DMZ):

- **ServidorWeb** (`20.13.67.2`) — Apache con HTTPS, certificado SSL con SAN, y una página en PHP conectada a la base de datos.
- **BaseDeDatos** (`20.13.67.3`) — MySQL Server, con acceso remoto habilitado solo para el ServidorWeb.

---

## 🌐 PARTE 1: ServidorWeb (20.13.67.2)

### 1.1 Actualización del sistema

```bash
sudo apt update
sudo apt upgrade -y
```

### 1.2 Instalación de Apache y PHP

```bash
sudo apt install apache2 php libapache2-mod-php php-mysql openssl -y
```

### 1.3 Inicio y habilitación de Apache

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

### 1.4 Página web de bienvenida

```bash
sudo nano /var/www/html/index.html
```

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

### 1.5 Generación del certificado SSL con SAN

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

**Parámetros explicados:**
- `-x509`: genera un certificado autofirmado.
- `-nodes`: sin contraseña para la clave privada.
- `-days 365`: validez de 1 año.
- `-newkey rsa:2048`: clave RSA de 2048 bits.
- `-subj`: información del certificado.
- `subjectAltName`: la IP real del servidor, exigida por los navegadores modernos.
- `basicConstraints=CA:FALSE`, `keyUsage`, `extendedKeyUsage`: marcan el certificado como certificado de servidor final, lo que permite importarlo sin problemas en cualquier sistema que lo consuma después (incluyendo el FortiGate).

### 1.6 Habilitación del módulo SSL en Apache

```bash
sudo a2enmod ssl
sudo systemctl restart apache2
```

### 1.7 Configuración del sitio SSL

```bash
sudo nano /etc/apache2/sites-available/default-ssl.conf
```

Buscar y modificar estas líneas (alrededor de la 32-33):

```apache
SSLCertificateFile      /etc/ssl/certs/apache-selfsigned.crt
SSLCertificateKeyFile   /etc/ssl/private/apache-selfsigned.key
```

Guardar con `Ctrl+O`, `Enter`, `Ctrl+X`.

### 1.8 Activación del sitio SSL

```bash
sudo a2ensite default-ssl.conf
sudo systemctl reload apache2
```

### 1.9 Página dinámica conectada a la base de datos

```bash
sudo nano /var/www/html/buscar.php
```

```php
<?php
mysqli_report(MYSQLI_REPORT_OFF);
$conn = new mysqli('20.13.67.3', 'lab_user', 'lab123', 'lab_db');
if ($conn->connect_error) { die('Sin conexión a la DB: ' . $conn->connect_error); }

$q = $_GET['q'] ?? '';
$sql = "SELECT * FROM productos WHERE nombre LIKE '%$q%'";
$result = $conn->query($sql);

if (!$result) { die('Error SQL: ' . $conn->error); }
while ($row = $result->fetch_assoc()) {
    echo htmlspecialchars(implode(' - ', $row)) . "<br>";
}
```

### 1.10 Verificación de puertos

```bash
sudo ss -tlnp | grep apache
```

**Salida esperada:**
```text
LISTEN 0  511  0.0.0.0:80   0.0.0.0:*  users:(("apache2",...))
LISTEN 0  511  0.0.0.0:443  0.0.0.0:*  users:(("apache2",...))
```

### 1.11 Pruebas locales

```bash
curl -k https://localhost
curl -k "https://20.13.67.2/buscar.php?q=teclado"
```

La segunda solo responde con datos una vez creada la tabla `productos` (Parte 2.5).

---

## 🗄️ PARTE 2: BaseDeDatos (20.13.67.3)

### 2.1 Actualización del sistema

```bash
sudo apt update
sudo apt upgrade -y
```

### 2.2 Instalación de MySQL Server

```bash
sudo apt install mysql-server -y
```

### 2.3 Inicio y habilitación de MySQL

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

### 2.4 Configuración de seguridad de MySQL

```bash
sudo mysql_secure_installation
```

**Respuestas recomendadas:**
- Validate password component: `N`
- Remove anonymous users: `Y`
- Disallow root login remotely: `N` (solo para pruebas)
- Remove test database: `Y`
- Reload privilege tables: `Y`

### 2.5 Base de datos, tabla y usuario restringido al ServidorWeb

```bash
sudo mysql -u root
```

```sql
SET GLOBAL validate_password.policy = LOW;
SET GLOBAL validate_password.length = 4;

CREATE DATABASE lab_db;

CREATE TABLE lab_db.productos (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nombre VARCHAR(100),
  precio DECIMAL(10,2)
);
INSERT INTO lab_db.productos (nombre, precio) VALUES
  ('Teclado', 25.50), ('Mouse', 12.00), ('Monitor', 180.00);

-- Usuario que solo puede conectarse desde la IP del ServidorWeb
CREATE USER 'lab_user'@'20.13.67.2' IDENTIFIED BY 'lab123';
GRANT ALL PRIVILEGES ON lab_db.* TO 'lab_user'@'20.13.67.2';

FLUSH PRIVILEGES;
EXIT;
```

Limitar el usuario a `20.13.67.2` (en vez de `%`) hace que, aunque alguien de Usuarios llegara por red al puerto 3306, MySQL rechace el login porque no viene de esa IP exacta. Es una segunda capa de seguridad, además de las políticas del FortiGate.

### 2.6 bind-address para acceso remoto

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

Buscar (línea ~31):
```ini
bind-address = 127.0.0.1
```

Cambiar a:
```ini
bind-address = 0.0.0.0
```

Guardar con `Ctrl+O`, `Enter`, `Ctrl+X`.

### 2.7 Reinicio de MySQL

```bash
sudo systemctl restart mysql
```

### 2.8 Verificación del puerto 3306

```bash
sudo ss -tlnp | grep 3306
```

**Salida esperada:**
```text
LISTEN 0  70  127.0.0.1:33060  0.0.0.0:*  users:(("mysqld",...))
LISTEN 0  151  0.0.0.0:3306   0.0.0.0:*  users:(("mysqld",...))
```

---

## 🔄 PARTE 3: prueba de conectividad entre servidores

Desde el ServidorWeb:

```bash
sudo apt install mysql-client -y
mysql -h 20.13.67.3 -u lab_user -p lab_db
```

Contraseña: `lab123`

```sql
SHOW TABLES;
SELECT * FROM productos;
EXIT;
```

---

## ✅ Resumen de configuración

| Servicio | Servidor | IP | Puerto | Acceso permitido |
| :--- | :--- | :--- | :---: | :--- |
| Apache HTTP/HTTPS | ServidorWeb | `20.13.67.2` | 80 / 443 | Usuarios (VLAN 10) vía FortiGate |
| `buscar.php` | ServidorWeb | `20.13.67.2` | 443 | Consulta a la base de datos |
| MySQL Server | BaseDeDatos | `20.13.67.3` | 3306 | Solo `lab_user`@`20.13.67.2` |

---

## 🔍 Comandos de verificación rápida

**ServidorWeb:**
```bash
sudo systemctl status apache2
sudo ss -tlnp | grep -E '80|443'
curl -k "https://20.13.67.2/buscar.php?q=teclado"
```

**BaseDeDatos:**
```bash
sudo systemctl status mysql
sudo ss -tlnp | grep 3306
sudo mysql -u root -e "SELECT user, host FROM mysql.user WHERE user='lab_user';"
```

---

### 👨‍💻 Realizado por: **Miguel Ramirez Meli**

![Lab](https://img.shields.io/badge/Lab-FortiGate_VM64-purple?style=for-the-badge)
