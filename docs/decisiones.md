# Decisiones de arquitectura — Comevi Plugins

Registro de todas las decisiones técnicas tomadas durante el desarrollo
inicial de este proyecto (sesión en Claude.ai, junio 2026). Sirve como
referencia para no revertir decisiones ya razonadas.

---

## D1 — Omitir `version` en `plugin.json`

**Decisión:** Ningún `plugin.json` de este repo incluye el campo `version`.

**Por qué:** Claude Code/Cowork detecta una actualización de plugin comparando
el último commit del repo contra el último commit sincronizado. Si `version`
existe, el auto-update solo se activa cuando ese string cambia manualmente —
es decir, el desarrollador tiene que acordarse de subirlo en cada PR. Sin ese
campo, cada commit nuevo en `main` cuenta automáticamente como versión nueva,
lo cual es el comportamiento deseado para un flujo de desarrollo ágil.

**Cuándo cambiar esto:** si en algún momento se necesita estabilidad de
versión explícita (ej. múltiples usuarios instalando versiones distintas de
un plugin en producción), agregar `version` y gestionar semver manualmente.

---

## D2 — Plan Personal (Pro/Max): el auto-update NO es instantáneo al hacer push

**Decisión:** Documentar y aceptar que el ciclo push → actualización en
Cowork no es instantáneo para cuentas personales.

**Por qué:** En planes Team/Enterprise con un admin, existe un webhook que
dispara la sincronización inmediatamente al hacer merge de un PR. En cuentas
personales ese webhook no existe — la actualización ocurre al iniciar una
sesión nueva (auto-update en segundo plano) o al ejecutar manualmente
`/plugin marketplace update comevi-plugins` en la CLI de Claude Code.

**Implicación práctica:** el flujo de desarrollo correcto es push → forzar
`/plugin marketplace update` → nueva conversación para que el skill cargue
la versión nueva. No esperar actualización automática dentro de una sesión
ya abierta.

---

## D3 — `/plugin marketplace` es CLI de Claude Code, no un slash command de Cowork

**Decisión:** Nunca documentar `/plugin marketplace add/update` como comandos
disponibles en el chat de Cowork.

**Por qué:** Se descubrió en la sesión de desarrollo que Cowork interpreta
`/plugin` como un intento de invocar un skill llamado "plugin" y devuelve
"Unknown skill: plugin". Los comandos de gestión de marketplaces solo existen
en la CLI de Claude Code (terminal). Traerlos a la UI de Cowork está
pendiente como mejora en la roadmap de Anthropic pero no existe aún.

---

## D4 — Conectores de directorio Anthropic vs. conectores custom en `.mcp.json`

**Decisión:** `.mcp.json` solo se crea cuando el plugin tiene un conector
con su propia URL. Los conectores del directorio de Anthropic (Google Drive,
Gmail, Slack, etc.) NO van en `.mcp.json`.

**Por qué:** Un conector de directorio (como Google Drive) ya está conectado
a nivel de cuenta del usuario en Claude.ai/Cowork. Para usarlo, basta con
que las instrucciones del `SKILL.md` lo referencien explícitamente y le
indiquen a Claude que use esas herramientas. Declararlo en `.mcp.json` sin
URL no cumple ninguna función verificada, y poner una URL inventada crearía
un conector roto que confundiría al usuario.

**Ejemplo de la diferencia:**
- `gerencia-general` tiene `.mcp.json` con `bi-dashboard-dummy` porque es
  un conector custom con URL propia (ficticia en este caso).
- `director-medico` NO tiene `.mcp.json` porque usa Google Drive, que ya
  está conectado a nivel de cuenta.

---

## D5 — Estrategia metadata-first para leer Google Drive

**Decisión:** Para cualquier fuente de Google Drive, el skill siempre llama
primero a `get_file_metadata` (sin `excludeContentSnippets`) antes de
decidir si descarga el contenido completo.

**Por qué — tres razones:**

1. **Frescura:** el `modifiedTime` de la metadata permite saber si el archivo
   cambió desde la última lectura en el mismo chat. Si no cambió, se reutiliza
   lo ya leído sin gastar tokens adicionales.

2. **Eficiencia:** para archivos pequeños/medianos, `get_file_metadata` ya
   trae el contenido completo en el campo `contentSnippet` en una sola llamada
   más liviana que `read_file_content`.

3. **Costo real medido (Sheet de Director Médico, junio 2026):**
   - `read_file_content` → 607 caracteres (tabla markdown limpia)
   - `download_file_content` → 668 caracteres (base64, +36% overhead)
   - `get_file_metadata` con contentSnippet → ~mismo contenido que
     read_file_content pero en una sola llamada que además trae modifiedTime.

**Flujo completo:**
```
get_file_metadata (sin excludeContentSnippets)
  → ¿modifiedTime igual al último visto en este chat?
      SÍ → usar lo que ya se leyó, no gastar más tokens
      NO (o primera vez) → usar contentSnippet como fuente de verdad
          → ¿contentSnippet parece truncado?
              SÍ → escalar a read_file_content
              NO → responder con el contentSnippet
```

**Regla derivada:** nunca usar `download_file_content` para leer contenido
que Claude necesita entender — es base64, Claude lo tiene que "decodificar"
mentalmente, y siempre ocupa más tokens que el texto plano equivalente.

---

## D6 — "Solo lectura" en Drive es behavioral, no técnico

**Decisión:** La restricción de no editar el Google Sheet del skill de
Director Médico se implementa como instrucción en el `SKILL.md`, no como
control técnico de permisos de Google Drive.

**Por qué:** El propietario del archivo (jpguarin@gmail.com) siempre tendrá
permisos de edición sobre sus propios archivos, sin importar cómo esté
configurado el plugin. Un bloqueo técnico real requeriría compartir el archivo
como "Lector" con una cuenta de servicio separada, lo cual está fuera del
alcance de este ejercicio dummy.

**Para producción:** si se necesita un bloqueo técnico real, crear una cuenta
de servicio con acceso de solo lectura y conectar el plugin a esa cuenta, no
a la cuenta propietaria del archivo.

---

## D7 — Referencias estáticas vs. referencias vivas

**Decisión:** Las referencias estáticas (`.md` bundleados en el plugin) se
leen una vez por conversación y no se refrescan. Las referencias vivas
(Google Drive u otras fuentes externas) usan la estrategia metadata-first
(D5) en cada uso dentro de la misma conversación.

**Por qué:** Refrescar referencias estáticas en cada turno no tiene sentido
— el contenido solo puede cambiar si alguien hace un commit al repo y el
plugin se actualiza, lo cual requiere una sesión nueva de todas formas
(D2). Gastar tokens en releer un `.md` que no puede haber cambiado dentro
del mismo chat es desperdicio puro.

**Regla práctica:** si el contenido puede cambiar a mitad de una conversación
(como un Sheet que alguien puede editar en tiempo real), usar referencia viva
con Drive. Si el contenido solo cambia con un deploy del plugin, usar `.md`
estático y olvidarse del refresco.
