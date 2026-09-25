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

> ⚠️ **Si ya aplicaste la Parte 3 (restricción de tráfico con `ufw`) antes de este paso**, el ServidorWeb no podrá salir a Internet para descargar paquetes. Verifica con `sudo ufw status`; si ya está activo con `deny (outgoing)` por defecto, abre temporalmente la salida:
> ```bash
> sudo ufw allow out 80/tcp
> sudo ufw allow out 443/tcp
> sudo ufw allow out 53
> ```
> Instala los paquetes, y al terminar esta parte **ciérralas de nuevo** (ver nota al final de la Parte 1).

```bash
sudo apt install apache2 php libapache2-mod-php php-mysql openssl -y
```

### 1.2.1 Habilitar el módulo PHP en Apache

La instalación no siempre habilita el módulo automáticamente. Verifica y habilítalo:

```bash
ls /etc/apache2/mods-available/ | grep php
```

Con el número de versión que aparezca (ej. `php8.1`):

```bash
sudo a2enmod php8.1
sudo systemctl restart apache2
```

**Cómo confirmar que PHP se está ejecutando (y no solo mostrando el código fuente):**

```bash
curl -k "https://localhost/buscar.php?q=teclado"
```

- Si la respuesta es el **código fuente del PHP tal cual** (`<?php mysqli_report(...`), el módulo no está habilitado: repite el `a2enmod` de arriba.
- Si la respuesta es un error de SQL (`Table 'lab_db.productos' doesn't exist`), PHP sí está funcionando: falta crear la tabla (Parte 2.5).
- Si la respuesta es `1 - Teclado - 25.50`, todo quedó correcto.

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

### 1.12 Cerrar el firewall si lo abriste para instalar paquetes

Si abriste la salida temporalmente en el paso 1.2, ciérrala ahora que ya terminaste de instalar todo (esto se hace **después** de completar también la Parte 2 y crear la tabla, para no tener que reabrirla de nuevo):

```bash
sudo ufw delete allow out 80/tcp
sudo ufw delete allow out 443/tcp
sudo ufw delete allow out 53
sudo ufw status verbose
```

En la sección de salida (`OUT`) solo debe quedar:
```text
20.13.67.3 3306/tcp        ALLOW OUT   Anywhere
```

Verifica que el aislamiento quedó correcto:
```bash
ping 8.8.8.8              # debe fallar
nc -zv 20.13.67.3 3306    # debe conectar
```

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

> ⚠️ **Ejecutar todo este bloque en el BaseDeDatos (`20.13.67.3`)**, con `sudo mysql -u root` (acceso local, sin restricción de host). Confirma en qué máquina estás con `ip a` antes de continuar: debe mostrar `20.13.67.3`, no `20.13.67.2`.

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

-- Verificar que la tabla quedó creada
SELECT * FROM lab_db.productos;

EXIT;
```

**Salida esperada del `SELECT`:**
```text
+----+---------+--------+
| id | nombre  | precio |
+----+---------+--------+
|  1 | Teclado |  25.50 |
|  2 | Mouse   |  12.00 |
|  3 | Monitor | 180.00 |
+----+---------+--------+
```

Limitar el usuario a `20.13.67.2` (en vez de `%`) hace que, aunque alguien de Usuarios llegara por red al puerto 3306, MySQL rechace el login porque no viene de esa IP exacta. Es una segunda capa de seguridad, además de las políticas del FortiGate.

> ⚠️ **No pruebes `mysql -h 20.13.67.3 -u lab_user -p` estando dentro de la propia BaseDeDatos.** MySQL vería la conexión como si viniera de `20.13.67.3`, no de `20.13.67.2`, y la rechazaría (`Access denied for user 'lab_user'@'20.13.67.3'`) aunque la contraseña sea correcta. Esa prueba de conectividad remota (Parte 3) se hace **desde el ServidorWeb**.

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
