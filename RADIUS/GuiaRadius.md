![Autor](https://img.shields.io/badge/Autor-Miguel%20Ramirez%20Meli-blue)
![Matricula](https://img.shields.io/badge/Matr%C3%ADcula-20251367-blue)
![Docente](https://img.shields.io/badge/Docente-Jonathan%20Rond%C3%B3n-blue)
![Materia](https://img.shields.io/badge/Materia-Seguridad%20de%20Redes-blue)
![Institucion](https://img.shields.io/badge/Instituci%C3%B3n-ITLA-lightgrey)
![Estado](https://img.shields.io/badge/Estado-Completado-brightgreen)

# Configuración de servicio NPS (RADIUS Server) — Niveles de acceso 15, 1 y 0

## Objetivo

Configurar un servidor **NPS (Network Policy Server)** sobre un controlador de dominio (Active Directory) como RADIUS para autenticar el acceso administrativo a **Router1**, definiendo grupos de acceso con distintos niveles de privilegio Cisco IOS.

## Topología

- **Router1** (e0/0) — **Switch2** — **Windows10-1** (NIC1) / **WindowsServer2022-1** (NIC1)
- VLAN 10 → `10.13.67.0/24`
- Dominio Active Directory: `miguel.local`

| Dispositivo | Rol | Dirección IP |
|---|---|---|
| Router1 | Gateway / Cliente RADIUS | `10.13.67.1` |
| WindowsServer2022-1 | Controlador de dominio + NPS | `10.13.67.10` |
| Windows10-1 | Cliente de prueba | `10.13.67.11` |

### Usuarios y grupos de dominio

| Usuario | Grupo | Nivel de privilegio Cisco |
|---|---|---|
| miguel (Miguel Ramirez Meli) | `Admins-Nivel15` | 15 (acceso total) |
| carlos (Carlos Peralta) | `Users-Nivel1` | 1 (acceso básico) |
| emily (Emily Peralta) | `Users-Nivel0` | 0 (acceso mínimo) |

---

## 1. Switch2 — Configuración de VLAN

### GUI

| Paso | Acción |
|---|---|
| 1 | Abrir la consola del switch en GNS3/EVE-NG (clic derecho → Console). |
| 2 | Entrar a modo privilegiado y luego a modo de configuración global. |
| 3 | Crear la VLAN 10 y asignarle el nombre `LAN_USUARIOS`. |
| 4 | Seleccionar el rango de interfaces `e0/0` a `e0/2`. |
| 5 | Configurar cada interfaz como modo access y asignarla a la VLAN 10. |
| 6 | Guardar la configuración en memoria (`write memory`). |

### CLI

```
enable
configure terminal
hostname Switch2
vlan 10
 name LAN_USUARIOS
exit
interface range e0/0-2
 switchport mode access
 switchport access vlan 10
exit
end
write memory
```

---

## 2. WindowsServer2022-1 — Red y Active Directory Domain Services

### GUI

| Paso | Acción |
|---|---|
| 1 | Configurar la tarjeta de red: IP `10.13.67.10`, máscara `255.255.255.0`, gateway `10.13.67.1`, DNS `10.13.67.10`. |
| 2 | Abrir **Server Manager** → **Add Roles and Features**. |
| 3 | Seleccionar el rol **Servicios de dominio de Active Directory** → Instalar. |
| 4 | Al finalizar, clic en la notificación de la bandera → **Promover este servidor a controlador de dominio**. |
| 5 | Elegir **Agregar un nuevo bosque** → Nombre de dominio raíz: `miguel.local`. |
| 6 | Dejar el nivel funcional por defecto, mantener marcado el DNS Server, definir la contraseña DSRM. |
| 7 | Confirmar y ejecutar la instalación (el servidor reinicia). |
| 8 | Tras reiniciar, verificar que el DNS del servidor apunte a sí mismo (`10.13.67.10`). |

### CLI

```powershell
# Red
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.13.67.10 -PrefixLength 24 -DefaultGateway 10.13.67.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.13.67.10

# Instalar AD DS
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

# Promover a controlador de dominio
Install-ADDSForest `
  -DomainName "miguel.local" `
  -DomainNetbiosName "MIGUEL" `
  -InstallDns `
  -SafeModeAdministratorPassword (ConvertTo-SecureString "DSRMPass123!" -AsPlainText -Force) `
  -Force:$true
```

---

## 3. Grupos y usuarios de dominio

### GUI

| Paso | Acción |
|---|---|
| 1 | Abrir **Server Manager** → **Tools** → **Usuarios y equipos de Active Directory**. |
| 2 | Clic derecho sobre `miguel.local` (o la OU deseada) → **Nuevo** → **Grupo**. |
| 3 | Crear el grupo `Admins-Nivel15` (Ámbito: Global, Tipo: Seguridad). |
| 4 | Crear el grupo `Users-Nivel1` con la misma configuración. |
| 5 | Crear el grupo `Users-Nivel0` con la misma configuración. |
| 6 | Clic derecho → **Nuevo** → **Usuario** para crear `miguel`, `carlos` y `emily`, con sus contraseñas. |
| 7 | Abrir las propiedades de cada usuario → pestaña **Miembro de** → agregarlo a su grupo correspondiente (`miguel`→`Admins-Nivel15`, `carlos`→`Users-Nivel1`, `emily`→`Users-Nivel0`). |

### CLI

```powershell
New-ADGroup -Name "Admins-Nivel15" -GroupScope Global -GroupCategory Security
New-ADGroup -Name "Users-Nivel1" -GroupScope Global -GroupCategory Security
New-ADGroup -Name "Users-Nivel0" -GroupScope Global -GroupCategory Security

New-ADUser -Name "miguel" -SamAccountName "miguel" `
  -AccountPassword (ConvertTo-SecureString "Segura2121..." -AsPlainText -Force) -Enabled $true
New-ADUser -Name "carlos" -SamAccountName "carlos" `
  -AccountPassword (ConvertTo-SecureString "Segura2121..." -AsPlainText -Force) -Enabled $true
New-ADUser -Name "emily" -SamAccountName "emily" `
  -AccountPassword (ConvertTo-SecureString "Segura2121..." -AsPlainText -Force) -Enabled $true

Add-ADGroupMember -Identity "Admins-Nivel15" -Members "miguel"
Add-ADGroupMember -Identity "Users-Nivel1" -Members "carlos"
Add-ADGroupMember -Identity "Users-Nivel0" -Members "emily"
```

---

## 4. Instalación del rol NPS

### GUI

| Paso | Acción |
|---|---|
| 1 | Abrir **Server Manager** → **Add Roles and Features**. |
| 2 | Seleccionar **Servicios de acceso y directivas de redes**. |
| 3 | Confirmar e instalar el rol. |

### CLI

```powershell
Install-WindowsFeature NPAS -IncludeManagementTools
```

---

## 5. Registrar Router1 como cliente RADIUS en NPS

### GUI

| Paso | Acción |
|---|---|
| 1 | Abrir **Herramientas** → **Servidor de directivas de redes**. |
| 2 | Ir a **Clientes y servidores RADIUS** → **Clientes RADIUS**. |
| 3 | Clic derecho → **Nuevo cliente RADIUS**. |
| 4 | Nombre descriptivo: `Router1`. |
| 5 | Dirección (IP o DNS): `10.13.67.1`. |
| 6 | Secreto compartido (Manual): `RadiusKey123`. |
| 7 | Confirmar secreto compartido y **Aceptar**. |

### CLI

```powershell
Import-Module NPS
New-NpsRadiusClient -Name "Router1" -Address "10.13.67.1" -SharedSecret "RadiusKey123"
```

---

## 6. Network Policies — Nivel 15, Nivel 1 y Nivel 0

### GUI

| Paso | Acción |
|---|---|
| 1 | En NPS, ir a **Directivas** → **Directivas de red**. |
| 2 | Clic derecho → **Nueva**. |
| 3 | Nombrar la política `Acceso-Nivel15`. |
| 4 | En **Especificar condiciones**, agregar **Grupos de Windows** → `MIGUEL\Admins-Nivel15`. |
| 5 | Seleccionar **Conceder acceso**. |
| 6 | En **Configurar métodos de autenticación**, marcar únicamente **"Autenticación sin cifrado (PAP, SPAP)"**. MS-CHAP/MS-CHAP-v2 pueden quedar marcados sin afectar; CHAP y "permitir sin negociar" deben quedar **sin marcar**. |
| 7 | En **Configurar restricciones**, avanzar sin cambios hasta **Configurar opciones**. |
| 8 | En **Atributos RADIUS → Estándar**, seleccionar `Framed-Protocol` (valor `PPP`) → **Quitar** (no aplica: es para PPP/VPN). |
| 9 | Seleccionar `Service-Type` → **Editar** → marcar el radio button **"Otros"** → en su lista desplegable elegir **`NAS Prompt`** (no "Callback NAS Prompt", no la sección de 802.1x) → **Aceptar**. |
| 10 | Ir a **Atributos RADIUS → Específico del proveedor** → **Agregar** → seleccionar **Cisco** → **Agregar**. |
| 11 | En el atributo `Cisco-AV-Pair` → **Agregar** → escribir `shell:priv-lvl=15` → **Aceptar** → **Cerrar**. |
| 12 | **Siguiente** → **Finalizar**. |
| 13 | Repetir del paso 2 al 12 para crear `Acceso-Nivel1`, con el grupo `MIGUEL\Users-Nivel1` y el valor `shell:priv-lvl=1`. |
| 14 | Repetir del paso 2 al 12 para crear `Acceso-Nivel0`, con el grupo `MIGUEL\Users-Nivel0` y el valor `shell:priv-lvl=0`. |
| 15 | En el listado de **Directivas de red**, confirmar el orden de procesamiento: `Acceso-Nivel15` (1) → `Acceso-Nivel1` (2) → `Acceso-Nivel0` (3), todas **Habilitada** y con **Tipo de acceso = Conceder acceso**. |

### CLI

```powershell
Import-Module NPS

New-NpsNetworkPolicy -Name "Acceso-Nivel15" `
  -Enabled $true -PolicyState "Enabled" -AccessPermission "Grant" `
  -GroupName "MIGUEL\Admins-Nivel15"

New-NpsNetworkPolicy -Name "Acceso-Nivel1" `
  -Enabled $true -PolicyState "Enabled" -AccessPermission "Grant" `
  -GroupName "MIGUEL\Users-Nivel1"

New-NpsNetworkPolicy -Name "Acceso-Nivel0" `
  -Enabled $true -PolicyState "Enabled" -AccessPermission "Grant" `
  -GroupName "MIGUEL\Users-Nivel0"
```

> **Nota:** los atributos `Service-Type`, la eliminación de `Framed-Protocol` y el `Cisco-AV-Pair` (Vendor Specific) no se pueden configurar de forma confiable por cmdlet nativo de PowerShell — el CLI de arriba solo crea el esqueleto de la política. Los pasos 8 al 11 de la tabla GUI son obligatorios y deben hacerse manualmente en la consola de NPS para las tres políticas.

---

## 7. Router1 — AAA y RADIUS

### GUI

> No aplica GUI en este dispositivo (equipo Cisco IOS gestionado por consola/CLI).

### CLI

```
enable
configure terminal
hostname Router1
interface e0/0
 ip address 10.13.67.1 255.255.255.0
 no shutdown
exit

username admin privilege 15 secret Admin123!

aaa new-model
aaa authentication login default group radius local
aaa authorization exec default group radius local

radius server NPS1
 address ipv4 10.13.67.10 auth-port 1812 acct-port 1813
 key RadiusKey123
exit

ip domain-name miguel.local
crypto key generate rsa modulus 2048
ip ssh version 2

line vty 0 4
 login authentication default
 transport input ssh telnet
exit

end
write memory
```

---

## 8. Windows10-1 — Cliente de prueba

### GUI

| Paso | Acción |
|---|---|
| 1 | Abrir **Panel de control** → **Redes e Internet** → **Centro de redes**. |
| 2 | Clic en el adaptador Ethernet → **Propiedades** → **Protocolo de Internet versión 4 (TCP/IPv4)**. |
| 3 | Configurar IP `10.13.67.11`, máscara `255.255.255.0`, gateway `10.13.67.1`. |
| 4 | Configurar DNS preferido: `10.13.67.10`. |

### CLI

```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.13.67.11 -PrefixLength 24 -DefaultGateway 10.13.67.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.13.67.10
```

---

## 9. Verificación / Pruebas de conectividad

### GUI

| Paso | Acción |
|---|---|
| 1 | Desde WindowsServer2022-1, abrir **Visor de eventos** → **Vistas personalizadas** → **Roles de servidor** → **Network Policy and Access Services**. |
| 2 | Revisar los eventos de conexión aceptados/rechazados tras cada intento de login. |

### CLI

```
! Verificación general en Router1
Router1#show ip interface brief
Router1#show running-config | section radius
Router1#show radius statistics

! Prueba directa de autenticación contra el NPS (sin necesidad de nueva sesión SSH)
Router1#test aaa group radius miguel Segura2121... legacy
Router1#test aaa group radius carlos Segura2121... legacy
Router1#test aaa group radius emily Segura2121... legacy

! Prueba de acceso vía SSH + confirmación exacta del nivel con "show privilege"
! - miguel (Admins-Nivel15) -> entra directo en modo privilegiado (#) -> show privilege = 15
! - carlos (Users-Nivel1)   -> entra en modo usuario (>) -> show privilege = 1
! - emily (Users-Nivel0)    -> entra en modo usuario (>) -> "show privilege" da
!   "% Invalid input" porque el nivel 0 ni siquiera permite comandos "show"
!   (comportamiento esperado: confirma el nivel más restringido)
```

### Resultado esperado

- `miguel` (grupo `Admins-Nivel15`) inicia sesión con privilegio **15** — confirmado con `show privilege`.
- `carlos` (grupo `Users-Nivel1`) inicia sesión con privilegio **1** — confirmado con `show privilege`.
- `emily` (grupo `Users-Nivel0`) inicia sesión con privilegio **0** — confirmado indirectamente: ni `show privilege` está disponible en ese nivel.
- `test aaa group radius` confirma autenticación exitosa contra el NPS para los tres usuarios.
- El servidor NPS registra las tres conexiones en el Event Viewer.
