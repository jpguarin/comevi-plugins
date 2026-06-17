# Comevi · Gerencia General (plugin de prueba)

Plugin de ejemplo, con datos y conector **dummy**, para validar la integración
de plugins por área en un marketplace personal de Claude Cowork.

## Componentes

- **Skill**: `resumen-ejecutivo-comevi` — genera un resumen ejecutivo
  consolidado a partir de datos ficticios en `references/kpis-comevi.md`.
- **MCP**: `bi-dashboard-dummy` — conector simulado de un dashboard de BI.
  No apunta a ningún servidor real; su único propósito es mostrar cómo se
  declara un conector en `.mcp.json`.

## Configuración

La variable de entorno `COMEVI_BI_TOKEN` está referenciada en `.mcp.json`
pero no es necesaria para este ejercicio: como la URL es ficticia, el
conector no intentará autenticarse contra nada real. En un plugin de
producción, aquí documentarías el token real requerido.

## Uso

Pide, por ejemplo:

- "Dame el resumen ejecutivo de Comevi"
- "¿Cómo van los indicadores generales este mes?"
- "Hazme un reporte para gerencia"

## Nota sobre versionado

Este `plugin.json` no incluye el campo `version` a propósito: así, cada
commit nuevo en la rama `main` del repo se trata como una versión nueva y
el auto-update de Claude Code/Cowork lo detecta sin tener que subir el
número de versión manualmente.
