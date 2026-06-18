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

## D5 — Estrategia de lectura directa para Google Drive

**Decisión:** Para cualquier fuente de Google Drive, el skill siempre usa
`read_file_content` en cada turno que necesite los datos. No se usa
`get_file_metadata` ni `download_file_content`.

**Por qué:** Las fuentes en Drive son datos dinámicos que pueden cambiar entre
turnos de la misma conversación (alguien puede editar el Sheet mientras habla
con el skill). Cualquier estrategia de caché basada en `modifiedTime` añade
complejidad para un beneficio marginal en tokens, y abre la puerta a responder
con datos desactualizados. Leer siempre con `read_file_content` garantiza que
cada respuesta refleja el estado real del archivo en ese momento.

**Por qué no `download_file_content`:** devuelve el contenido en base64,
que Claude tiene que "decodificar" mentalmente y ocupa más tokens que el
texto plano que entrega `read_file_content`.

**Reglas derivadas (todas en SKILL.md y CLAUDE.md):**
- Siempre `read_file_content`, nunca `get_file_metadata` ni
  `download_file_content`.
- Hacer el request en cada turno, sin excepción.
- Nunca responder con números recordados de un turno anterior.
- Los archivos son de solo lectura: ningún skill debe escribir o editar sobre
  ellos (ver D6 para el razonamiento de por qué esto es behavioral, no
  técnico).

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
(Google Drive u otras fuentes externas) se leen con `read_file_content`
en cada turno dentro de la misma conversación (ver D5).

**Por qué:** Refrescar referencias estáticas en cada turno no tiene sentido
— el contenido solo puede cambiar si alguien hace un commit al repo y el
plugin se actualiza, lo cual requiere una sesión nueva de todas formas
(D2). Gastar tokens en releer un `.md` que no puede haber cambiado dentro
del mismo chat es desperdicio puro.

**Regla práctica:** si el contenido puede cambiar a mitad de una conversación
(como un Sheet que alguien puede editar en tiempo real), usar referencia viva
con Drive. Si el contenido solo cambia con un deploy del plugin, usar `.md`
estático y olvidarse del refresco.
