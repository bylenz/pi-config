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
