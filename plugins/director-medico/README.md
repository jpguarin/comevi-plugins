# Comevi · Director Médico (plugin de prueba)

Plugin de ejemplo, con datos dummy, que combina dos tipos de referencia:
una estática (bundleada en el plugin) y una viva (un Google Sheet real en
Drive, leído en modo de solo lectura).

## Componentes

- **Skill**: `protocolos-clinicos-comevi`
  - Referencia estática: `references/guias-clinicas.md` (protocolos clínicos
    ficticios).
  - Referencia viva: Google Sheet **"Comevi - Indicadores Médicos (dummy)"**
    (ID `1Kdj1HOlAsJIeXAWWCt7zF9xxJvUASebA_R_gYEYDeIM`), leído a través del
    conector de Google Drive ya conectado en tu cuenta.

## Por qué este plugin no incluye un `.mcp.json`

A diferencia del plugin de Gerencia General (que declara un conector
custom `bi-dashboard-dummy` con su propia URL), Google Drive es un
**conector del directorio de Anthropic**: ya está conectado a nivel de tu
cuenta, no a nivel del plugin. En teoría, un `.mcp.json` puede declarar un
conector de directorio por nombre (sin URL) para que se ofrezca
automáticamente a usuarios que todavía no lo tienen conectado, pero no
verifiqué con certeza el nombre exacto de esa clave en la documentación
pública, así que preferí no inventarlo. Si más adelante quieres probar
esto con un colega que no tenga Drive conectado, podemos averiguar el
nombre correcto juntos antes de declararlo.

## Modo de lectura

El skill instruye explícitamente a Claude a nunca escribir ni editar ese
Google Sheet, aunque como eres el propietario del archivo técnicamente
podrías editarlo. Esta es una salvaguarda de comportamiento (en las
instrucciones del skill), no un bloqueo técnico de Google Drive. Si
quisieras un bloqueo técnico real, deberías compartir el archivo como
"Lector" con la cuenta que usa el conector, en vez de ser tú el propietario
con permisos de edición.

## Frescura de los datos dentro de la misma conversación

El skill está instruido para **nunca reutilizar una lectura anterior del
Sheet dentro del mismo chat**. Cada vez que vuelvas a preguntar algo sobre
indicadores médicos, Claude relee el Sheet en ese momento en lugar de confiar
en lo que recordaba de un turno anterior — así, si cambias un valor en Drive
a mitad de la conversación, la siguiente respuesta ya refleja el cambio.

Nota técnica: esto se logra con una instrucción explícita en el `SKILL.md`,
no con un hook. Los hooks de tipo comando corren en tu entorno local, pero
los conectores de Cowork (como Google Drive) solo son accesibles por Claude
durante su propio turno — un hook no puede llamarlos directamente sin
credenciales propias de Google, que quedan fuera del alcance de este
ejercicio dummy.

## Uso

Pide, por ejemplo:

- "¿Cuál es el protocolo de manejo de sepsis en Comevi?"
- "¿Cómo van los indicadores médicos de calidad este mes?"
- "Dame un resumen para la Dirección Médica"
