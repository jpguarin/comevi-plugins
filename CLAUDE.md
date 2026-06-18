# CLAUDE.md — Comevi Plugins Marketplace

Este archivo es leído automáticamente por Claude Code al abrir este directorio.
Contiene todo el contexto necesario para continuar el desarrollo sin repetir
decisiones ya tomadas.

---

## Qué es este proyecto

Repositorio de un **marketplace personal de plugins para Claude Cowork/Code**,
organizado por áreas de la empresa ficticia **Comevi**. Es un proyecto de
aprendizaje (datos y conectores dummy) para entender la integración de plugins
por área en un marketplace personal.

Cada área tiene su propio plugin independiente. Los 6 plugins están
pre-declarados en `marketplace.json`; solo faltan por construir los últimos 4.

---

## Estado actual

| Plugin | Carpeta | Estado |
|--------|---------|--------|
| Gerencia General | `plugins/gerencia-general` | ✅ Completo |
| Director Médico | `plugins/director-medico` | ✅ Completo |
| Gerente Comercial | `plugins/gerente-comercial` | ❌ Pendiente |
| Operaciones | `plugins/operaciones` | ❌ Pendiente |
| Recursos Humanos | `plugins/recursos-humanos` | ❌ Pendiente |
| Facturación / EPS | `plugins/facturacion-eps` | ❌ Pendiente |

El archivo `.claude-plugin/marketplace.json` en la raíz ya tiene declarados
los 6 plugins con sus rutas relativas correctas. No hay que tocarlo.

---

## Estructura obligatoria de cada plugin

```
plugins/<nombre-area>/
├── .claude-plugin/
│   └── plugin.json          ← manifiesto (SIN campo version — ver decisiones)
├── skills/
│   └── <nombre-skill>/
│       ├── SKILL.md          ← instrucciones para Claude + frontmatter
│       └── references/
│           └── *.md          ← datos/documentación dummy estáticos
└── README.md                ← documentación del plugin
```

Si el plugin tiene un conector MCP custom (no de directorio Anthropic):

```
├── .mcp.json                ← declaración del conector con su URL
```

---

## Reglas que NO debes cambiar

### 1. Nunca agregar `version` en `plugin.json`
Todos los `plugin.json` de este repo omiten el campo `version` deliberadamente.
Si lo agregas, el auto-update de Claude Code solo detectará cambios cuando
subas manualmente ese número. Sin ese campo, cada commit nuevo en `main` cuenta
como versión nueva.

### 2. El `description` del frontmatter del `SKILL.md` son frases gatillo
Ese campo es lo que Claude lee para decidir cuándo activar el skill. Deben
ser frases específicas en tercera persona, no una descripción técnica. Ver
los dos plugins completos como referencia.

### 3. Los conectores de directorio Anthropic (Google Drive, Gmail, etc.)
no van en `.mcp.json`
`.mcp.json` es solo para conectores custom con su propia URL. Los conectores
del directorio de Anthropic ya están conectados a nivel de cuenta del usuario
y se referencian directamente en las instrucciones del `SKILL.md`.

### 4. Reglas para manipulación de Google Drive
Para fuentes en Google Drive, los skills siguen estas reglas:
1. **Siempre usar `read_file_content`** para leer el archivo. No usar
   `get_file_metadata` ni `download_file_content`.
2. **Siempre hacer el request en cada turno** que requiera datos del archivo,
   ya que la información es dinámica y puede cambiar con frecuencia entre turnos.
3. **Nunca responder con números recordados de un turno anterior**; siempre
   leer el archivo fresco en el turno actual.
4. **Los archivos son de solo lectura.** No usar ninguna herramienta de
   escritura o edición sobre ellos (aunque el usuario actual sea propietario
   y técnicamente pueda editarlos, los skills nunca deben modificarlos).

---

## Recursos externos ya creados

| Recurso | Detalle |
|---------|---------|
| Google Sheet (Director Médico) | "Comevi - Indicadores Médicos (dummy)" |
| ID del Sheet | `1Kdj1HOlAsJIeXAWWCt7zF9xxJvUASebA_R_gYEYDeIM` |
| Propietario | jpguarin@gmail.com |
| URL | https://docs.google.com/spreadsheets/d/1Kdj1HOlAsJIeXAWWCt7zF9xxJvUASebA_R_gYEYDeIM/edit |

---

## Qué construir en cada plugin pendiente

Cada plugin pendiente debe tener exactamente:

1. **`plugin.json`** — copia el de `gerencia-general` como base, cambiando
   `name` y `description`.
2. **`SKILL.md`** — con frontmatter (`name`, `description` con frases gatillo,
   `metadata.area`), instrucciones de cómo responder, fuentes que leer y tono.
3. **Una referencia estática** — un `.md` con datos dummy relevantes al área
   en `skills/<nombre>/references/`.
4. **`README.md`** — documentando componentes, uso y cualquier decisión
   particular del plugin.

Si el plugin tiene una fuente externa en Drive (como Director Médico), agregar
la instrucción de metadata-first en el `SKILL.md` y documentarlo en el README.

---

## Cómo instalar el marketplace (contexto del usuario)

El usuario tiene Claude Code CLI instalado. El marketplace se instala con:

```bash
/plugin marketplace add TU-USUARIO/comevi-plugins
/plugin install comevi-<area>@comevi-plugins
```

Para forzar actualización tras un push:

```bash
/plugin marketplace update comevi-plugins
```

**Importante**: `/plugin marketplace` es un comando de Claude Code CLI, **no**
un slash command del chat de Cowork. En Cowork, los plugins se instalan
manualmente desde la UI de plugins, no por CLI.

---

## Referencia rápida de plugins completados

### gerencia-general
- Skill: `resumen-ejecutivo-comevi`
- Referencias: `references/kpis-comevi.md` (estática, datos mensuales ficticios)
- Conector MCP dummy: `bi-dashboard-dummy` (URL ficticia en `.mcp.json`)
- Patrón de referencia: **solo estática**

### director-medico
- Skill: `protocolos-clinicos-comevi`
- Referencias: `references/guias-clinicas.md` (estática) + Google Sheet en
  Drive (viva, estrategia metadata-first)
- Sin `.mcp.json` (Drive es conector de directorio Anthropic, ya conectado)
- Patrón de referencia: **estática + Drive vivo con metadata-first**

---

## Decisiones de arquitectura

Ver `docs/decisiones.md` para el razonamiento completo detrás de cada
decisión. Este CLAUDE.md solo contiene las reglas; ese archivo contiene el
"por qué".
