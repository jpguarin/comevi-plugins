# Comevi Plugins (marketplace personal de prueba)

Repositorio de prueba para aprender a integrar un marketplace personal de
plugins en Claude Cowork. Todos los datos y conectores son **dummy/ficticios**.

## Plugins incluidos

- `comevi-gerencia-general` — listo
- `comevi-director-medico` — próximamente
- `comevi-gerente-comercial` — próximamente
- `comevi-operaciones` — próximamente
- `comevi-recursos-humanos` — próximamente
- `comevi-facturacion-eps` — próximamente

## Cómo se actualiza

Los `plugin.json` de este repo omiten el campo `version` a propósito: cada
commit nuevo en `main` cuenta como una versión nueva, así Cowork puede
detectar cambios automáticamente sin tener que subir un número de versión
a mano.
