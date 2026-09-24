# 🗄️ Configuración del Servidor de Base de Datos (MySQL)

![MySQL](https://img.shields.io/badge/MySQL-8.0-blue?logo=mysql&style=for-the-badge)
![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04-orange?logo=ubuntu&style=for-the-badge)
![Database](https://img.shields.io/badge/Database-Remote_Access-green?style=for-the-badge)
![GitHub](https://img.shields.io/badge/GitHub-Repo-black?style=for-the-badge)

## 📋 Descripción
Este documento detalla los pasos para instalar, configurar y habilitar el acceso remoto a un servidor de base de datos **MySQL 8.0** en un entorno Ubuntu. Se incluye la creación de la base de datos, usuarios, y la configuración de red para permitir conexiones desde otras VLANs a través del FortiGate.

---

## 1️⃣ Instalación y Configuración Inicial

### Instalar MySQL Server
```bash
sudo apt update
sudo apt install mysql-server -y
```

### Iniciar y habilitar el servicio
```bash
sudo systemctl start mysql
sudo systemctl enable mysql
```

---

## 2️⃣ Creación de Base de Datos y Usuario

### Ingresar a la consola de MySQL
```bash
sudo mysql -u root
```

### ⚠️ Solución al Error de Política de Contraseñas (ERROR 1819)
MySQL 8.0 incluye el componente `validate_password` que exige contraseñas complejas. Para un entorno de laboratorio, podemos relajar esta política temporalmente ejecutando:

```sql
-- Bajar la exigencia a LOW (solo verifica longitud)
SET GLOBAL validate_password.policy = LOW;

-- Reducir la longitud mínima a 4 caracteres
SET GLOBAL validate_password.length = 4;
```

### Crear la Base de Datos y el Usuario Remoto
Una vez relajada la política, ejecutamos los comandos de creación:

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

---

## 3️⃣ Habilitar Acceso Remoto (Bind Address)

Por defecto, MySQL solo escucha en `localhost` (127.0.0.1). Para que el FortiGate y la VLAN de Usuarios puedan llegar a él, debemos cambiarlo a `0.0.0.0`.

### Editar el archivo de configuración
```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

### Modificar la directiva `bind-address`
Busca la línea (alrededor de la línea 31) y cámbiala:

```ini
# Antes:
# bind-address = 127.0.0.1

# Después:
bind-address = 0.0.0.0
```
*(Opcional: Comenta la línea `mysqlx-bind-address` poniendo un `#` al inicio si aparece).*

Guarda con `Ctrl + O`, presiona `Enter`, y sal con `Ctrl + X`.

---

## 4️⃣ Reinicio y Verificación

### Reiniciar el servicio de MySQL
```bash
sudo systemctl restart mysql
```

### Verificar que esté escuchando en todas las interfaces
```bash
sudo ss -tlnp | grep 3306
```

**Salida esperada:**
```text
LISTEN 0      70         127.0.0.1:33060      0.0.0.0:*    users:(("mysqld",...))         
LISTEN 0      151          0.0.0.0:3306       0.0.0.0:*    users:(("mysqld",...))         
```
> ✅ **Nota:** La línea `0.0.0.0:3306` confirma que el servidor acepta conexiones remotas.

---

## 5️⃣ Prueba de Conexión Remota

Para validar que la configuración es correcta, nos conectamos desde el **ServidorWeb (20.13.67.2)** hacia la **BaseDeDatos (20.13.67.3)**.

### Desde el ServidorWeb:
```bash
# Instalar el cliente de MySQL (si no está instalado)
sudo apt install mysql-client -y

# Conectarse al servidor remoto
mysql -h 20.13.67.3 -u lab_user -p
```
*(Contraseña: `lab123`)*

Si el prompt cambia a `mysql>`, la conexión remota es exitosa. Escribe `EXIT;` para salir.

---

<br>

### 👨‍💻 Realizado por: **Miguel Ramirez Meli**

![Author](https://img.shields.io/badge/Author-Miguel_Ramirez_Meli-orange?style=for-the-badge)
![Lab](https://img.shields.io/badge/Lab-FortiGate_VM64-purple?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Database_Configured-brightgreen?style=for-the-badge)
