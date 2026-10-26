# Servidor MCP de Cisco Packet Tracer

Servidor **Model Context Protocol (MCP)** que permite a cualquier cliente con soporte MCP (OpenCode, Claude Code, Codex CLI, VS Code, Cursor) planificar, validar, generar y desplegar topologías de red completas directamente en **Cisco Packet Tracer** a partir de una instrucción en lenguaje natural.

**Ejemplo:** *"crea una red con 3 routers, OSPF y DHCP"* -> el servidor planifica, valida, genera configuraciones y despliega la topología en Packet Tracer en tiempo real.

Este repositorio contiene **todo lo necesario para replicar el servidor**: código fuente, catálogo de dispositivos, módulos, cables, pruebas y configuración de ejemplo para OpenCode.

---

## Tabla de contenidos

1. [Requisitos previos](#1-requisitos-previos)
2. [Instalar Python](#2-instalar-python)
3. [Instalar Cisco Packet Tracer](#3-instalar-cisco-packet-tracer)
4. [Obtener este repositorio](#4-obtener-este-repositorio)
5. [Instalar el servidor MCP](#5-instalar-el-servidor-mcp)
6. [Instalar OpenCode](#6-instalar-opencode)
7. [Configurar el MCP en OpenCode](#7-configurar-el-mcp-en-opencode)
8. [Verificar la instalación](#8-verificar-la-instalación)
9. [Primer uso](#9-primer-uso)
10. [Despliegue en vivo en Packet Tracer](#10-despliegue-en-vivo-en-packet-tracer)
11. [Solución de problemas](#11-solución-de-problemas)
12. [Estructura del repositorio](#12-estructura-del-repositorio)
13. [Atribución y licencia](#13-atribución-y-licencia)

---

## 1. Requisitos previos

| Requisito | Versión mínima | Notas |
|-----------|----------------|-------|
| Sistema operativo | Windows / macOS / Linux | Mismo comportamiento en los tres |
| Python | 3.11+ | Requerido |
| Cisco Packet Tracer | 8.2+ | Solo para despliegue en vivo (probado en 8.2 y 9.0) |
| Git | Cualquiera | Para clonar el repositorio |

No se necesitan claves, tokens ni variables de entorno. Este repositorio no contiene secretos.

---

## 2. Instalar Python

### Windows

1. Descargue el instalador desde [python.org/downloads](https://www.python.org/downloads/).
2. Ejecute el instalador y **marque** la opción **"Add Python to PATH"**.
3. Verifique en PowerShell:

```powershell
python --version
# Python 3.11.x o superior

pip --version
```

### macOS

```bash
brew install python@3.11
python3 --version
```

### Linux (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install python3.11 python3.11-venv python3-pip
python3.11 --version
```

---

## 3. Instalar Cisco Packet Tracer

1. Regístrese en [netacad.com](https://www.netacad.com/resources/lab-downloads) y descargue Cisco Packet Tracer (versión 8.2 o superior; se recomienda 9.0).
2. Instale siguiendo el asistente de la plataforma.
3. Abra Packet Tracer al menos una vez para confirmar que funciona.

> El despliegue en vivo requiere la extensión **MCP-BUILDER** incluida en este repositorio (`V3-MCP-BUILDER.pts`). Véase la sección [10. Despliegue en vivo](#10-despliegue-en-vivo-en-packet-tracer).

---

## 4. Obtener este repositorio

```bash
git clone https://github.com/oropezagabriel2004-blip/MCP-packet
cd MCP-packet
```

---

## 5. Instalar el servidor MCP

### 5.1. Crear un entorno virtual (recomendado)

**Windows (PowerShell):**

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install -e .
```

**macOS / Linux:**

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -e .
```

### 5.2. Instalar dependencias opcionales

**Herramientas offline de archivos `.pkt`/`.pka`** (opcional, requiere `twofish`):

```bash
pip install -e ".[offline]"
```

**Dependencias de desarrollo y pruebas** (opcional):

```bash
pip install -e ".[dev]"
```

### 5.3. Verificar la instalación del paquete

```bash
python -m packet_tracer_mcp --help
```

También se instala un script de consola `pt-mcp`:

```bash
pt-mcp --stdio
```

---

## 6. Instalar OpenCode

### Windows

OpenCode se distribuye como binario nativo. La forma más sencilla es con npm:

```powershell
npm install -g @opencode/cli
```

> Si no tiene Node.js instalado, descárguelo desde [nodejs.org](https://nodejs.org/) (versión LTS) y vuelva a ejecutar el comando.

Verifique:

```powershell
opencode --version
```

### macOS

```bash
brew install opencode 2>/dev/null || npm install -g @opencode/cli
opencode --version
```

### Linux

```bash
npm install -g @opencode/cli
opencode --version
```

> **Nota:** Windows no soporta gestores de paquetes para OpenCode. Use npm o descargue el binario standalone desde la página oficial.

---

## 7. Configurar el MCP en OpenCode

Este repositorio incluye un archivo `opencode.jsonc` en la raíz que registra el servidor MCP de Packet Tracer. El archivo usa la ruta `python` del sistema; para que funcione con el entorno virtual, modifique la ruta del intérprete.

### 7.1. Configuración por proyecto (recomendada)

Abra el repositorio en OpenCode y edite `opencode.jsonc` para usar el Python del entorno virtual:

**Windows:**

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "servers": {
      "packet-tracer": {
        "type": "local",
        "command": [
          "C:\\Users\\SU_USUARIO\\MCP-packet\\.venv\\Scripts\\python.exe",
          "-m",
          "packet_tracer_mcp",
          "--stdio"
        ]
      }
    }
  }
}
```

**macOS / Linux:**

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "servers": {
      "packet-tracer": {
        "type": "local",
        "command": [
          "/ruta/a/MCP-packet/.venv/bin/python",
          "-m",
          "packet_tracer_mcp",
          "--stdio"
        ]
      }
    }
  }
}
```

### 7.2. Configuración global (opcional)

Si desea usar el MCP desde cualquier directorio, regístrelo globalmente:

```bash
opencode mcp add packet-tracer --global -- .venv/Scripts/python.exe -m packet_tracer_mcp --stdio
```

> En Windows use la ruta absoluta al intérprete del entorno virtual.

### 7.3. Alternativa: usar el CLI de OpenCode

```bash
opencode mcp add packet-tracer -- npx -y example-mcp-server
```

Adapte el comando al intérprete de Python que tenga instalado el paquete.

---

## 8. Verificar la instalación

### 8.1. Probar el servidor manualmente

En una terminal, ejecute:

```bash
python -m packet_tracer_mcp --stdio
```

El servidor se iniciará en modo stdio y esperará entrada. Presione `Ctrl+C` para detenerlo.

### 8.2. Verificar desde OpenCode

1. Abra OpenCode en el directorio del repositorio.
2. Ejecute:

```bash
opencode mcp list
```

Debería aparecer `packet-tracer` con estado **Connected**.

### 8.3. Ejecutar las pruebas

```bash
python -m pytest tests/ -q
```

Resultado esperado:

```
389 passed in ~10s
```

### 8.4. Probar una herramienta

Desde OpenCode, pida al asistente:

> "Lista los dispositivos disponibles en Packet Tracer"

El asistente invocará `pt_list_devices` y mostrará los 74 modelos soportados.

---

## 9. Primer uso

### Ejemplo básico

Pida al asistente de OpenCode:

> "Crea una red con 2 routers, 2 switches, 4 PCs, DHCP y enrutamiento estático"

El servidor ejecutará el pipeline completo:

1. **Planificación** -> genera un `TopologyPlan` con dispositivos, enlaces, IPs, DHCP y rutas.
2. **Validación** -> verifica modelos, puertos, cables e IPs.
3. **Generación** -> produce el script JavaScript para Packet Tracer y las configuraciones IOS.
4. **Despliegue** -> envía los comandos al bridge HTTP (si está conectado) o exporta a archivos.

### Transporte HTTP (opcional)

Para compartir una sola instancia del servidor entre varios clientes:

```bash
python -m packet_tracer_mcp
# Servidor en http://127.0.0.1:39000/mcp
# Bridge en http://127.0.0.1:54321
```

---

## 10. Despliegue en vivo en Packet Tracer

El despliegue en vivo envía comandos directamente a una instancia de Packet Tracer en ejecución.

### 10.1. Instalar la extensión MCP-BUILDER

1. Copie el archivo `V3-MCP-BUILDER.pts` a la carpeta `extensions/` del **directorio de instalación** de Packet Tracer (no la carpeta por usuario):
   - **Windows:** `C:\Program Files\Cisco Packet Tracer <versión>\extensions\`
   - **macOS / Linux:** carpeta `extensions/` dentro de la instalación de Packet Tracer
2. Reinicie Packet Tracer.
3. En el primer inicio, Packet Tracer mostrará un aviso de **"load this extension?"** -> haga clic en **Yes**.
4. La ventana MCP-BUILDER se abrirá automáticamente. **Déjela visible** (puede moverla a una esquina); si se oculta, QtWebEngine congela los temporizadores y el polling se detiene.

> En Windows puede necesitar ejecutar Packet Tracer o copiar el archivo como administrador para escribir en `Program Files`.

### 10.2. Verificar la conexión

Desde OpenCode, pida al asistente:

> "Verifica el estado del bridge de Packet Tracer"

El asistente invocará `pt_bridge_status`. Debería reportar que el bridge está activo y PTBuilder está haciendo polling.

### 10.3. Método alternativo (sin extensión)

Si prefiere usar el builder integrado de Packet Tracer:

1. Vaya a **Extensions -> Builder Code Editor**.
2. Pegue este fragmento y haga clic en **Run**:

```javascript
/* PT-MCP Bridge */ window.webview.evaluateJavaScriptAsync("setInterval(function(){var x=new XMLHttpRequest();x.open('GET','http://127.0.0.1:54321/next',true);x.onload=function(){if(x.status===200&&x.responseText){$se('runCode',x.responseText)}};x.onerror=function(){};x.send()},500)");
```

> Use solo comentarios de bloque `/* */`. Los comentarios de línea `//` rompen el snippet porque Packet Tracer elimina los saltos de línea internamente.

---

## 11. Solución de problemas

| Síntoma | Causa | Solución |
|---------|-------|----------|
| `pt_bridge_status` reporta PT no conectado | QtWebEngine congela temporizadores de webviews ocultos | Mantenga la ventana MCP-BUILDER **visible** |
| La extensión `.pts` no aparece en PT 9.0 | PT 9.0 no escanea la carpeta de extensiones por usuario | Copie `V3-MCP-BUILDER.pts` a la carpeta `extensions/` del **directorio de instalación** |
| "Could not start the HTTP bridge on :54321" | Otro proceso ocupa el puerto 54321 | Detenga el proceso: `Get-NetTCPConnection -LocalPort 54321` en PowerShell |
| `addModule` devuelve `false` silenciosamente | El argumento `slot` fue un entero en lugar de cadena | Pase el slot como **cadena**: `"0/0"` para HWIC, `"0"` para NIM |
| Error "Invalid arguments for IPC call createLink" | `addLink()` sin el 5º argumento (tipo de cable) | Siempre pase el cable: `addLink("R1","Gig0/0","SW1","Gig0/1","straight")` |
| El snippet del bridge no hace polling | PTBuilder elimina saltos de línea; `//` comenta el resto | Use solo comentarios `/* */` |
| Instalar 2+ módulos con llamadas separadas agota el bridge | Cada llamada reinicia el dispositivo | Use `pt_install_modules_batch` |
| Claude Desktop (MSIX) no enciende el indicador MCP | El sandbox MSIX no expone `python` en PATH | Reemplace `"command": "python"` por la ruta absoluta del intérprete |
| `ModuleNotFoundError: No module named 'imp'` | `twofish` 0.3.0 importa `imp`, eliminado en Python 3.12 | El archivo `offline-tools/_twofish_compat.py` se carga automáticamente |
| Un proyecto exportado no aparece en el repositorio | `ProjectRepository` es relativo al directorio de trabajo del servidor | Inicie el servidor desde la raíz del repositorio |

---

## 12. Estructura del repositorio

```
MCP-packet/
├── src/packet_tracer_mcp/     # Código fuente del servidor MCP
│   ├── adapters/mcp/          # Registro de herramientas y recursos MCP
│   ├── application/           # Casos de uso y DTOs
│   ├── domain/                # Modelos, servicios y reglas de negocio
│   ├── infrastructure/        # Catálogo, generadores, ejecución, persistencia
│   ├── shared/                # Enums, constantes y utilidades
│   ├── server.py              # Instancia FastMCP
│   ├── settings.py            # Configuración del servidor
│   └── __main__.py            # Punto de entrada: python -m packet_tracer_mcp
├── tests/                     # 389 pruebas automatizadas
├── offline-tools/             # Utilidades para archivos .pkt/.pka/.pts
├── demo/                      # Imágenes de demostración
├── img/                       # Capturas de pantalla
├── docs/                      # Documentación adicional
├── scripts/                   # Scripts auxiliares
├── V3-MCP-BUILDER.pts         # Extensión de Packet Tracer para despliegue en vivo
├── opencode.jsonc             # Configuración de ejemplo para OpenCode
├── pyproject.toml             # Metadatos y dependencias del paquete
├── LICENSE                    # Licencia MIT
├── SECURITY.md                # Política de seguridad
└── README.md                  # Este archivo
```

### Resumen de capacidades

| Categoría | Cantidad |
|-----------|----------|
| Herramientas MCP | 71 |
| Recursos MCP | 5 |
| Modelos de dispositivos | 74 |
| Módulos de expansión | 150 |
| Tipos de cable | 15 |
| Plantillas de topología | 9 |
| Protocolos de enrutamiento | Static, OSPF, EIGRP, RIP |
| Pruebas automatizadas | 389 |

---

## 13. Atribución y licencia

Este proyecto se basa en **[MCP-Packet-Tracer](https://github.com/mex-i/MCP-Packet-Tracer)** (licencia MIT, autor: Mateo). Se conservan los avisos de licencia y autoría originales.

### Licencia

Este proyecto está licenciado bajo la **[Licencia MIT](LICENSE)**.

```
MIT License
Copyright (c) 2026 Mateo - Packet Tracer MCP Server
```

Puede usar, modificar y distribuir este software libremente, siempre que se mantenga el aviso de licencia.

### Créditos

- **Proyecto original:** [mex-i/MCP-Packet-Tracer](https://github.com/mex-i/MCP-Packet-Tracer)
- **Protocolo MCP:** [modelcontextprotocol.io](https://modelcontextprotocol.io)
- **Cisco Packet Tracer:** [netacad.com](https://www.netacad.com)

---

<div align="center">

**Construido con MCP - Impulsado por Pydantic - Se despliega en Cisco Packet Tracer**

</div>
