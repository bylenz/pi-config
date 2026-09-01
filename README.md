# Configuración personal de Pi

Backup portable de `~/.pi/agent`. El repositorio es privado: **aun así no contiene credenciales**.

## Incluido

- `settings.json`: modelo por defecto, tema, paquetes y overrides de subagentes.
- `mcp.json`: servidor MCP local de Engram (`/opt/homebrew/bin/engram`), cargado bajo demanda.
- `extensions/custom-header.ts`: sustituye la cabecera inicial por el logo `tre`.
- `extensions/prompt-snippets/`: extensión `alt+s` / `/snippets` para activar reglas de prompt por mensaje.
  - **Session kickoff**: conoce el proyecto y pide alineación antes de trabajar.
  - **Orchestrator mode**: delega exploración y trabajo mecánico a subagentes.
  - **Ask questions**: pregunta hasta aclarar el encargo.
  - **Verify, don't assume**: verifica hechos críticos.
  - **Delegate exploration**: conserva contexto delegando exploración dirigida.
  - **Diagnose, don't fix**: investiga y propone, sin modificar código.

## Paquetes gestionados

`settings.json` es la fuente de verdad: Pi instala los paquetes desde sus orígenes. `npm/package.json` y `npm/package-lock.json` son estado generado por esa instalación, por eso no se versionan.

| Paquete | Uso |
| --- | --- |
| `pi-mcp-adapter` | Carga `mcp.json` y conecta los servidores MCP. |
| `pi-web-access` | Búsqueda web y lectura de contenido remoto. |
| `ponytail` (Git) | Prioriza soluciones mínimas y evita sobreingeniería. |
| `rpiv-ask-user-question` | Preguntas estructuradas al usuario. |
| `pi-subagents` | Delegación y coordinación de subagentes. |
| `gentle-engram` | Memoria persistente entre sesiones. |
| `rpiv-todo` | Lista de tareas de Pi. |
| `pi-image-tools` | Herramientas de imágenes. |
| `pi-usage-meters` | Indicadores de uso y coste. |

## Subagentes

Los proporciona `pi-subagents` y están disponibles de inmediato:

| Agente | Modelo | Úsalo cuando quieras... |
| --- | --- | --- |
| `scout` | `openai-codex/gpt-5.6-luna` | Reconocimiento rápido del código local: archivos relevantes, puntos de entrada, flujo de datos, riesgos. |
| `researcher` | `zai/glm-5.3` | Investigación web/documentación con fuentes y un informe breve. |
| `worker` | `zai/glm-5.3` | Trabajo de implementación. Edita archivos, valida y escala las decisiones no aprobadas en vez de adivinar. |
| `reviewer` | `openai-codex/gpt-5.6-sol` | Revisión de código y correcciones menores contra la tarea/plan, tests, casos límite y simplicidad. |
| `oracle` | `openai-codex/gpt-5.6-sol` | Una segunda opinión antes de actuar. Cuestiona supuestos sin editar. |
| `delegate` | `zai/glm-5.3` | Un delegado general ligero que se comporta casi como la sesión padre. |

Regla práctica: `scout` antes de entender el código, `researcher` antes de confiar en hechos externos, `worker` para implementar, `reviewer` para comprobar y `oracle` cuando la decisión en sí parece arriesgada.

### Flujo: planificar → validar → delegar en paralelo

1. **Investigar** — «Usa el subagente `researcher` para investigar X y darme un informe breve con fuentes.»
2. **Planificar** — «Con ese informe, propón un plan de implementación."
3. **Validar el plan** — «Pide al `oracle` una segunda opinión sobre este plan: riesgos, supuestos dudosos y qué falta.» → ajusta el plan según el veredicto.
4. **Delegar en paralelo** — «Ejecuta el plan: lanza varios `worker` en paralelo con `runs.all`, uno por tarea independiente del plan."
5. **Revisar** — «Pasa al `reviewer` el diff completo contra el plan: tests, casos límite y simplicidad.»

> Combina con los snippets: `orchestrator-mode` (prepend) para que la sesión no lea código ella misma, y `delegate-exploration` (append) al pedir la exploración previa.

## Análisis de imágenes con zai MCP server

### Español

El modelo principal no admite imágenes directamente. Para analizar una imagen, usa el servidor MCP `zai-mcp-server` (configurado en `mcp.json`, carga bajo demanda):

1. Pide: «analiza esta imagen con zai mcp server» y pega la ruta del portapapeles (`/var/folders/.../pi-clipboard-*.png`).
2. Pi llama a `analyze_image` con `image_source` (ruta o URL) y `prompt` (qué extraer).

Herramientas disponibles:

| Herramienta | Uso |
| --- | --- |
| `analyze_image` | Análisis general: extrae texto, tablas y estructura de cualquier imagen. |
| `extract_text_from_screenshot` | Solo OCR/reconocimiento de texto de capturas. |
| `ui_to_artifact` | Convierte capturas de UI en artefactos (HTML, componentes, código). |
| `ui_diff_check` | Compara dos capturas y detecta diferencias visuales. |
| `diagnose_error_screenshot` | Diagnostica errores y stack traces de capturas. |
| `understand_technical_diagram` | Explica diagramas técnicos (arquitectura, flujos). |
| `analyze_data_visualization` | Analiza gráficos y visualizaciones de datos. |
| `analyze_video` | Analiza contenido de vídeo. |

### English

The main model cannot view images directly. To analyze an image, use the `zai-mcp-server` MCP server (configured in `mcp.json`, loaded on demand):

1. Ask: "analyze this image using zai mcp server" and paste the clipboard path (`/var/folders/.../pi-clipboard-*.png`).
2. Pi calls `analyze_image` with `image_source` (path or URL) and `prompt` (what to extract).

Available tools:

| Tool | Use |
| --- | --- |
| `analyze_image` | General analysis: extract text, tables, and structure from any image. |
| `extract_text_from_screenshot` | OCR/text recognition from screenshots only. |
| `ui_to_artifact` | Turn UI screenshots into artifacts (HTML, components, code). |
| `ui_diff_check` | Compare two screenshots and spot visual differences. |
| `diagnose_error_screenshot` | Diagnose errors and stack traces from screenshots. |
| `understand_technical_diagram` | Explain technical diagrams (architecture, flows). |
| `analyze_data_visualization` | Analyze charts and data visualizations. |
| `analyze_video` | Analyze video content. |

## Prompt snippets (`alt+s` / `/snippets`)

La extensión `extensions/prompt-snippets/` añade reglas de prompt que se activan por mensaje. Cada snippet es una instrucción pequeña e independiente; se fusionan en tu mensaje al enviarlo y los toggles se resetean tras cada envío.

**Cómo usarlos:**

1. Pulsa **alt+s** o ejecuta **/snippets** con el editor abierto.
2. Navega con `↑`/`↓`, activa con `espacio`, aplica con `enter` (o `esc` para cancelar).
3. `tab` previsualiza el snippet resaltado (nombre, colocación, orden y cuerpo completo).
4. Los activos aparecen como widget sobre el editor: `↑ prepend:` (antes de tu texto, color acento) y `↓ append:` (después, color aviso).
5. Al enviar, se fusionan: grupo *prepend* (ordenado por `order`) → tu texto → grupo *append* (ordenado por `order`).

**Snippets disponibles** (`snippets/*.md`, se re-escanean en cada apertura y envío — editar surte efecto sin `/reload`):

| Snippet | Colocación (orden) | Qué hace |
| --- | --- | --- |
| `session-kickoff` | prepend (10) | Explora el proyecto y reporta antes de empezar; no trabaja hasta alinear el siguiente paso. |
| `ask-questions` | append (10) | Pregunta hasta estar 100% seguro de qué hacer; no actúa hasta confirmación. |
| `verify-not-assume` | append (20) | Verifica hechos críticos en vez de asumir; pregunta si no puede verificar. |
| `orchestrator-mode` | prepend (30) | Sesión de orquestación pura: delega trabajo mecánico (exploración, lectura, implementación) a subagentes y conserva su contexto. |
| `delegate-exploration` | append (30) | Mantén el contexto ligero: delega la exploración del código a subagentes con preguntas concretas; lee archivos solo para verificar lo crítico. |
| `diagnose-report` | append (40) | Solo lectura: investiga y diagnostica el problema, reporta hallazgos y fix propuesto sin aplicar cambios. |

**Crear uno nuevo:** archivo `.md` en `snippets/` con frontmatter `name`, `description`, `placement` (`prepend`/`append`, por defecto `append`) y `order` (número, por defecto `9999`).


## Restaurar en otra máquina

```bash
npm install -g --ignore-scripts @earendil-works/pi-coding-agent
mkdir -p ~/.pi
git clone git@github.com:bylenz/pi-config.git ~/.pi/agent

jq -r '.packages[] | if type == "string" then . else .source end' \
  ~/.pi/agent/settings.json |
while IFS= read -r package; do
  pi install "$package"
done
```

Abre `pi` y usa `/login` para autenticar de nuevo. Instala Engram si quieres usar el MCP configurado.

## No incluido

`auth.json`, sesiones, perfiles, cachés, historial, paquetes descargados y decisiones de confianza son estado local o contienen secretos; están excluidos por `.gitignore`.
