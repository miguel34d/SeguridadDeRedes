## 🔍 Configuración: Activación de Deep Packet Inspection (DPI)

### 4.1 Vía GUI (Interfaz Gráfica)
1. Ve a **Policy & Objects** > **Firewall Policy**.
2. Haz doble clic en la política `Usuarios_to_WebServer_HTTPS`.
3. Desplázate a la sección **Security Profiles** y activa los siguientes perfiles:
   - **SSL Inspection:** Selecciona `deep-inspection`.
   - **Antivirus:** Selecciona `default`.
   - **IPS:** Selecciona `default`.
   - **Application Control:** Selecciona `default`.
   - *(Opcional)* **Web Filter:** Selecciona `default` (Nota: en VM sin licencia mostrará "invalid license").
4. Haz clic en **OK** para guardar.

### 4.2 Vía CLI (Línea de Comandos)
Si prefieres configurar los perfiles de seguridad directamente desde la consola del FortiGate, accede por SSH o CLI y ejecuta los siguientes comandos (asumiendo que la política HTTPS tiene el ID `3`):

```bash
config firewall policy
    edit 3
        set ssl-ssh-profile "deep-inspection"
        set av-profile "default"
        set ips-sensor "default"
        set application-list "default"
    next
end
```

### 4.3 Descarga e Instalación del Certificado CA en Windows
Para que el navegador confíe en el tráfico desencriptado por el FortiGate, es obligatorio instalar su certificado raíz:
1. Ve a **System** > **Certificates**, busca el certificado `FORTINET_CA_SSL` (o `FGVM...`) y haz clic en **Download**.
2. En el equipo Windows, haz **doble clic** en el archivo `.crt` descargado.
3. Clic en **Instalar certificado...** > Selecciona **Equipo local** > Siguiente.
4. Marca: **Colocar todos los certificados en el siguiente almacén**.
5. Clic en **Examinar...** > Selecciona **Entidades de certificación raíz de confianza** > Aceptar > Finalizar > **Sí**.

### 4.4 Verificación
1. Cierra **completamente** el navegador (Administrador de tareas > Finalizar tarea).
2. Navega a `https://20.13.67.2`.
3. Haz clic en el candado > **El certificado es válido**.
4. En la pestaña "Detalles", verifica que el campo **Emitido por** muestre el nombre del certificado del FortiGate (`FGVM...`). Esto confirma que el DPI está desencriptando e inspeccionando el tráfico.
