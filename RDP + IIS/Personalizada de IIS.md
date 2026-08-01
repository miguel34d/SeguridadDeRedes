# 🌐 Lab: Configuración de una Página Personalizada en IIS (HTTPS/443)

![Estado](https://img.shields.io/badge/Estado-Completado-brightgreen)
![Resultado](https://img.shields.io/badge/Resultado-Sitio%20Web%20Funcional-brightgreen)
![Dominio](https://img.shields.io/badge/Dominio-miguel.local-orange)
![HTTPS](https://img.shields.io/badge/Acceso-HTTPS%20Seguro-success)
![Certificado](https://img.shields.io/badge/Certificado-Miguel.localssl-blueviolet)

---

## ✅ Checklist rápido

- [ ] Rol **Servidor Web (IIS)** instalado
- [ ] Carpeta del sitio creada con la página personalizada (portal de accesos)
- [ ] Documento predeterminado configurado
- [ ] Certificado `Miguel.localssl` ya creado (documento anterior) — reutilizado aquí
- [ ] Sitio creado en IIS Manager con enlace HTTPS en el puerto 443 usando `Miguel.localssl`
- [ ] Certificado confiado en Windows10-1 (ya lo estaba)
- [ ] Regla de Firewall habilitada para HTTPS (puerto 443)
- [ ] Acceso probado desde `https://miguel.local/` sin errores de certificado

---

## 🔧 1. Instalar el rol Servidor Web (IIS)

**En WindowsServer2022-1 → Administrador del servidor → Administrar → Agregar roles y características**

| Paso | Click |
|---|---|
| 1 | ⭕ **Instalación basada en características o en roles** → Siguiente |
| 2 | Seleccionar servidor → Siguiente |
| 3 | Roles de servidor → ✅ **Servidor Web (IIS)** |
| 4 | **Agregar características** (para agregar las herramientas de administración) → Siguiente |
| 5 | Características → Siguiente |
| 6 | Servidor Web (IIS) → Siguiente |
| 7 | Servicios de rol → asegúrate de incluir ✅ **Seguridad → Seguridad de contenido estático** y ✅ **Rendimiento en el desarrollo de aplicaciones → según lo necesites** (deja marcados los predeterminados) → Siguiente |
| 8 | Confirmación → **Instalar** → esperar → **Cerrar** |

---

## 📁 2. Crear la carpeta del sitio y la página personalizada

**Explorador de archivos en WindowsServer2022-1:**

| Paso | Click |
|---|---|
| 1 | Crea la carpeta `C:\inetpub\miguel-site` |
| 2 | Dentro de esa carpeta, crea un archivo de texto y renómbralo a `index.html` |
| 3 | Ábrelo con el Bloc de notas y pega el contenido HTML del portal (ver abajo) |
| 4 | Guarda el archivo con codificación **UTF-8** |

La página es un **portal de accesos rápidos** con la identidad del estudiante y enlaces a recursos externos y al cliente web de escritorio remoto (RDWeb):

- 📰 Diario Libre
- ▶ YouTube
- 🎓 Página oficial del ITLA
- 🖥 Escritorio remoto (`https://miguel.local/RDWeb/webclient/`)

**Contenido de `index.html`:**

```html
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Miguel Ramírez — Portal | ITLA</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@400;500;600;700&family=IBM+Plex+Sans:wght@300;400;500;600&display=swap" rel="stylesheet">
<style>
  :root{
    --bg-0:#0d1520;
    --bg-1:#111c2b;
    --bg-2:#162335;
    --line:#233248;
    --cyan:#5fd8c4;
    --amber:#e8a13a;
    --paper:#eef2f6;
    --muted:#7f92a8;
    --mono:'IBM Plex Mono', monospace;
    --sans:'IBM Plex Sans', sans-serif;
  }
  *{box-sizing:border-box; margin:0; padding:0;}
  body{
    background:
      radial-gradient(ellipse at 15% -10%, rgba(95,216,196,0.08), transparent 45%),
      radial-gradient(ellipse at 90% 10%, rgba(232,161,58,0.06), transparent 40%),
      var(--bg-0);
    color:var(--paper);
    font-family:var(--sans);
    line-height:1.6;
    min-height:100vh;
  }
  body::before{
    content:"";
    position:fixed; inset:0;
    background-image:
      linear-gradient(var(--line) 1px, transparent 1px),
      linear-gradient(90deg, var(--line) 1px, transparent 1px);
    background-size:64px 64px;
    opacity:0.18;
    pointer-events:none;
    z-index:0;
  }
  .wrap{position:relative; z-index:1; max-width:960px; margin:0 auto; padding:0 28px;}
  .topbar{
    display:flex; align-items:center; justify-content:space-between;
    padding:22px 28px;
    max-width:960px; margin:0 auto;
    border-bottom:1px solid var(--line);
  }
  .brand{font-family:var(--mono); font-size:13px; letter-spacing:.12em; color:var(--muted); text-transform:uppercase;}
  .brand strong{color:var(--cyan); font-weight:600;}
  .status{display:flex; align-items:center; gap:8px; font-family:var(--mono); font-size:12px; color:var(--muted);}
  .dot{width:7px; height:7px; border-radius:50%; background:var(--cyan); box-shadow:0 0 8px var(--cyan); animation:pulse 2.4s ease-in-out infinite;}
  @keyframes pulse{0%,100%{opacity:1;} 50%{opacity:.35;}}
  .hero{padding:56px 0 20px;}
  .term{
    background:var(--bg-1); border:1px solid var(--line); border-radius:10px;
    overflow:hidden; max-width:520px; box-shadow:0 30px 60px -25px rgba(0,0,0,0.6);
  }
  .term-head{display:flex; align-items:center; gap:8px; padding:12px 16px; background:var(--bg-2); border-bottom:1px solid var(--line);}
  .term-head span{width:10px; height:10px; border-radius:50%; background:#2a3a50;}
  .term-head .title{margin-left:8px; font-family:var(--mono); font-size:12px; color:var(--muted);}
  .term-body{padding:20px 24px; font-family:var(--mono); font-size:13.5px;}
  .term-body .line{margin-bottom:8px; color:var(--muted);}
  .term-body .key{color:var(--amber);}
  .term-body .val{color:var(--paper); font-weight:500;}
  .cursor{display:inline-block; width:8px; height:15px; background:var(--cyan); vertical-align:-3px; animation:blink 1s step-end infinite;}
  @keyframes blink{50%{opacity:0;}}
  h1{font-family:var(--mono); font-size:clamp(26px, 4vw, 38px); font-weight:700; margin:32px 0 8px; color:var(--paper);}
  h1 .accent{color:var(--cyan);}
  .subtitle{font-family:var(--sans); font-size:15.5px; color:var(--muted); max-width:56ch;}
  section{padding:48px 0 64px;}
  .eyebrow{font-family:var(--mono); font-size:12px; letter-spacing:.16em; color:var(--amber); text-transform:uppercase; margin-bottom:10px; display:flex; align-items:center; gap:10px;}
  .eyebrow::after{content:""; flex:1; height:1px; background:var(--line);}
  h2{font-family:var(--mono); font-size:22px; font-weight:600; margin-bottom:8px; color:var(--paper);}
  .lead{color:var(--muted); max-width:62ch; margin-bottom:32px; font-size:14.5px;}
  .grid{display:grid; grid-template-columns:repeat(auto-fit, minmax(230px,1fr)); gap:18px;}
  .card{
    position:relative; display:flex; flex-direction:column; gap:14px;
    background:var(--bg-1); border:1px solid var(--line); border-radius:12px;
    padding:24px 22px; text-decoration:none; color:inherit;
    transition:border-color .2s ease, transform .2s ease, background .2s ease;
  }
  .card:hover{border-color:var(--cyan); transform:translateY(-3px); background:var(--bg-2);}
  .card .icon{
    width:38px; height:38px; display:flex; align-items:center; justify-content:center;
    border-radius:9px; background:rgba(95,216,196,0.08); border:1px solid var(--line); font-size:18px;
  }
  .card h3{font-family:var(--sans); font-size:16px; font-weight:600; color:var(--paper);}
  .card p{font-size:13px; color:var(--muted); line-height:1.5;}
  .card .go{margin-top:auto; font-family:var(--mono); font-size:11.5px; color:var(--cyan); display:flex; align-items:center; gap:6px;}
  .card .go::after{content:"→"; transition:transform .2s ease;}
  .card:hover .go::after{transform:translateX(4px);}
  .card.remote{border-color:rgba(232,161,58,0.35);}
  .card.remote .icon{background:rgba(232,161,58,0.1); border-color:rgba(232,161,58,0.35);}
  .card.remote .go{color:var(--amber);}
  footer{
    border-top:1px solid var(--line); padding:28px 0 40px;
    display:flex; justify-content:space-between; align-items:center; flex-wrap:wrap; gap:12px;
    font-family:var(--mono); font-size:12px; color:var(--muted);
  }
  footer span{color:var(--cyan);}
  @media (max-width:640px){.topbar{flex-direction:column; align-items:flex-start; gap:10px;}}
  @media (prefers-reduced-motion: reduce){.dot, .cursor{animation:none;}}
</style>
</head>
<body>

  <div class="topbar">
    <div class="brand">ITLA <strong>/</strong> Seguridad de Redes</div>
    <div class="status"><span class="dot"></span> portal activo</div>
  </div>

  <div class="wrap">
    <section class="hero">
      <div class="term">
        <div class="term-head">
          <span></span><span></span><span></span>
          <span class="title">meliolin@itla — sesión</span>
        </div>
        <div class="term-body">
          <div class="line"><span class="key">usuario</span>&nbsp;&nbsp;<span class="val">Miguel Ramírez</span></div>
          <div class="line"><span class="key">matrícula</span>&nbsp;<span class="val">2025-1367</span></div>
          <div class="line"><span class="key">materia</span>&nbsp;&nbsp;&nbsp;<span class="val">Seguridad de Redes — ITLA</span></div>
          <div class="line">accesos disponibles <span class="cursor"></span></div>
        </div>
      </div>

      <h1>Portal de <span class="accent">accesos rápidos</span></h1>
      <p class="subtitle">
        Página de inicio personalizada del laboratorio. Selecciona un destino para abrirlo directamente.
      </p>
    </section>

    <section>
      <div class="eyebrow">Navegación</div>
      <h2>Accesos</h2>
      <p class="lead">Enlaces configurados como página de inicio del servidor.</p>

      <div class="grid">
        <a class="card" href="https://www.diariolibre.com" target="_blank" rel="noopener">
          <div class="icon">📰</div>
          <h3>Diario Libre</h3>
          <p>Noticias y actualidad de República Dominicana.</p>
          <div class="go">abrir</div>
        </a>

        <a class="card" href="https://www.youtube.com" target="_blank" rel="noopener">
          <div class="icon">▶</div>
          <h3>YouTube</h3>
          <p>Plataforma de video para consulta y referencia.</p>
          <div class="go">abrir</div>
        </a>

        <a class="card" href="https://www.itla.edu.do" target="_blank" rel="noopener">
          <div class="icon">🎓</div>
          <h3>Página del ITLA</h3>
          <p>Sitio oficial del Instituto Tecnológico de Las Américas.</p>
          <div class="go">abrir</div>
        </a>

        <a class="card remote" href="https://miguel.local/RDWeb/webclient/" target="_blank" rel="noopener">
          <div class="icon">🖥</div>
          <h3>Escritorio remoto</h3>
          <p>Cliente web RDWeb del laboratorio (miguel.local).</p>
          <div class="go">conectar</div>
        </a>
      </div>
    </section>
  </div>

  <footer>
    <div class="wrap" style="display:flex; justify-content:space-between; align-items:center; flex-wrap:wrap; gap:12px; width:100%;">
      <div>© 2026 · Miguel Ramírez · ITLA</div>
      <div>status: <span>online</span></div>
    </div>
  </footer>

</body>
</html>
```

---

## 🔒 3. Reutilizar el certificado `Miguel.localssl`

No hace falta crear un certificado nuevo para IIS. El mismo `Miguel.localssl` creado por PowerShell en el documento **RDP-RemoteAPP.md** ya está en el almacén `Cert:\LocalMachine\My` de `WindowsServer2022-1` con el Key Usage correcto (`DigitalSignature,KeyEncipherment`), así que aparecerá directamente en el desplegable de certificados al crear el sitio en IIS (paso 4.6 más abajo).

---

## 🔧 4. Crear el sitio web en IIS Manager con enlace HTTPS

**Herramientas → Administrador de Internet Information Services (IIS)**

| Paso | Click |
|---|---|
| 1 | En el panel izquierdo, expande el servidor → click derecho en **Sitios** → **Agregar sitio web...** |
| 2 | Nombre del sitio: `miguel-site` |
| 3 | Ruta de acceso física: `C:\inetpub\miguel-site` |
| 4 | Tipo: **https** — Dirección IP: **Todas sin asignar** — Puerto: **443** |
| 5 | Nombre de host: `miguel.local` |
| 6 | Certificado SSL: selecciona **Miguel.localssl** (el mismo certificado ya creado y usado en RDS/Gateway) del menú desplegable |
| 7 | Aceptar |

---

## 📄 5. Configurar el documento predeterminado

**En IIS Manager, con el sitio `miguel-site` seleccionado:**

| Paso | Click |
|---|---|
| 1 | Doble click en **Documento predeterminado** (panel central) |
| 2 | Verifica que `index.html` esté en la lista |
| 3 | Si no está, click en **Agregar...** (panel derecho) → escribe `index.html` → Aceptar |
| 4 | Selecciónalo y usa **Subir** (panel derecho) para que quede de primero en la lista |

---

## 🔥 6. Habilitar el Firewall para HTTPS

**En WindowsServer2022-1 → PowerShell como administrador:**

```powershell
New-NetFirewallRule -DisplayName "Permitir HTTPS (Puerto 443)" -Direction Inbound -Protocol TCP -LocalPort 443 -Action Allow
```

---

## 🔒 7. Confirmar que Windows10-1 ya confía en el certificado

Como el certificado es el mismo `Miguel.localssl` confiado en los documentos anteriores, **normalmente no hace falta volver a importarlo**. Solo verifica:

| Paso | Click |
|---|---|
| 1 | En Windows10-1: `Win + R` → `certmgr.msc` |
| 2 | Expande **Entidades de certificación raíz de confianza → Certificados** |
| 3 | Confirma que aparece `miguel.local` en la lista |

Si no aparece (por ejemplo, porque este es el primer documento que haces del lab), expórtalo sin clave privada y trúalo en Windows10-1:

**En WindowsServer2022-1:**

| Paso | Click |
|---|---|
| 1 | `Win + R` → `mmc` → Archivo → **Agregar o quitar complementos...** → **Certificados** → **Agregar** |
| 2 | ⭕ **Cuenta de equipo** → Siguiente → ⭕ **Equipo local** → Finalizar → Aceptar |
| 3 | Expande **Certificados (Equipo local) → Personal → Certificados** |
| 4 | Click derecho en `CN=miguel.local` (nombre descriptivo **Miguel.localssl**) → **Todas las tareas → Exportar...** |
| 5 | Siguiente → ⭕ **No, no exportar la clave privada** → Siguiente |
| 6 | ⭕ **X.509 codificado en Base64 (.CER)** → Siguiente |
| 7 | Nombre de archivo: `C:\Certificados\Miguel.localssl.cer` → Siguiente → **Finalizar** |
| 8 | Copia `Miguel.localssl.cer` a Windows10-1 (por la carpeta compartida `\\WIN-3RVTQIDV70S\Compartido`) |

**En Windows10-1:**

| Paso | Click |
|---|---|
| 9 | Doble click sobre el archivo → **Instalar certificado...** → ⭕ **Equipo local** → Siguiente |
| 10 | ⭕ **Colocar todos los certificados en el siguiente almacén** → **Examinar...** → **Entidades de certificación raíz de confianza** → Aceptar |
| 11 | Siguiente → **Finalizar** → debe salir **"La importación se realizó correctamente"** |

---

## 🧪 8. Probar el acceso desde Windows10-1

| Paso | Click |
|---|---|
| 1 | Cierra el navegador por completo y ábrelo de nuevo en Windows10-1 |
| 2 | Escribe `https://miguel.local/` |
| 3 | Verifica el candado 🔒 sin advertencia de certificado |
| 4 | Confirma que cargue el **portal de accesos** con la información del estudiante y las tarjetas de Diario Libre, YouTube, ITLA y Escritorio remoto |
| 5 | Prueba la tarjeta **Escritorio remoto** y confirma que abra correctamente `https://miguel.local/RDWeb/webclient/` |

---

## 🔁 Comandos de mantenimiento (opcional)

<details>
<summary>Ver comandos útiles</summary>

```powershell
# Ver el estado del sitio
Get-Website -Name "miguel-site"

# Ver los enlaces (bindings) del sitio
Get-WebBinding -Name "miguel-site"

# Reiniciar el sitio
Restart-WebItem "IIS:\Sites\miguel-site"

# Ver las reglas de firewall relacionadas a HTTPS
Get-NetFirewallRule -DisplayName "*HTTPS*"

# Ver los certificados instalados en el almacén personal
Get-ChildItem -Path Cert:\LocalMachine\My
```
</details>

---

## ✅ Checklist final

- [x] Rol **Servidor Web (IIS)** instalado
- [x] Carpeta del sitio creada con la página personalizada (portal de accesos)
- [x] Documento predeterminado configurado
- [x] Certificado `Miguel.localssl` ya creado (documento anterior) — reutilizado aquí
- [x] Sitio creado en IIS Manager con enlace HTTPS en el puerto 443 usando `Miguel.localssl`
- [x] Certificado confiado en Windows10-1 (ya lo estaba)
- [x] Regla de Firewall habilitada para HTTPS (puerto 443)
- [x] Acceso probado desde `https://miguel.local/` sin errores de certificado, con el portal y todas sus tarjetas funcionando
