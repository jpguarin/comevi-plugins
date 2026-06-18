---
name: protocolos-clinicos-comevi
description: >
  Este skill debe usarse cuando el usuario pida "protocolo clínico", "guía de
  manejo", "criterios de referencia", "indicadores médicos", "cómo van los
  indicadores de calidad asistencial" o "resumen para la Dirección Médica" de
  Comevi. Cubre guías clínicas internas y los indicadores médicos consolidados
  (readmisión, eventos adversos, sepsis, infecciones, satisfacción).
metadata:
  version: "0.1.0"
  area: "Director Médico"
---

# Protocolos clínicos e indicadores médicos de Comevi

## Advertencia general

Todo el contenido que usa este skill (guías y datos numéricos) es **ficticio**,
creado únicamente para probar la configuración de plugins en Claude Cowork.
Nunca debe presentarse ni usarse como guía clínica real ni como base para
decisiones de atención a pacientes reales.

## Fuentes de este skill

Este skill combina dos tipos de referencia distintos a propósito, para
demostrar ambos patrones:

1. **Referencia estática (archivo bundleado)**: `references/guias-clinicas.md`,
   con protocolos clínicos de ejemplo. Se lee directamente del plugin, igual
   que cualquier archivo de referencia.

2. **Referencia viva (Google Sheets vía conector de Google Drive)**: la hoja
   de cálculo **"Comevi - Indicadores Médicos (dummy)"**
   (ID: `1Kdj1HOlAsJIeXAWWCt7zF9xxJvUASebA_R_gYEYDeIM`), que vive en Google
   Drive y no está empaquetada en el plugin. Para leerla, usar las
   herramientas del conector de Google Drive ya conectado (por ejemplo,
   `search_files` por el nombre exacto, o `get_file_metadata` /
   `download_file_content` directamente con el ID anterior).

## Instrucciones

1. Si la pregunta es sobre protocolos, guías de manejo o criterios clínicos
   cualitativos: leer `references/guias-clinicas.md`.
2. Si la pregunta es sobre indicadores médicos cuantitativos (readmisión,
   sepsis, eventos adversos, infecciones, satisfacción): leer la hoja de
   cálculo de Google Drive descrita arriba usando el conector de Google
   Drive. No asumir ni inventar los números; si el conector no está
   disponible o el archivo no se encuentra, decirlo explícitamente en vez de
   inventar datos.
3. **Estrategia eficiente en tokens para verificar si los datos cambiaron:**
   a. Llamar primero a `get_file_metadata` con el ID de arriba, **sin**
      `excludeContentSnippets`. Esta llamada es más liviana que
      `read_file_content` o `download_file_content`, y para un archivo de
      este tamaño ya devuelve el contenido completo en el campo
      `contentSnippet`, junto con `modifiedTime`.
   b. Si ya se consultó este archivo antes en la misma conversación, comparar
      el `modifiedTime` nuevo contra el que se vio la última vez:
      - Si es **igual**, los datos no cambiaron: se puede responder con lo
        que ya se leyó, sin necesidad de re-analizar nada adicional.
      - Si es **distinto** (o es la primera vez en este chat), usar el
        `contentSnippet` de esta respuesta como la fuente de verdad actual.
   c. Solo si el `contentSnippet` luce truncado o incompleto (por ejemplo, el
      archivo creció mucho más de lo esperado), complementar con
      `read_file_content` para traer el contenido íntegro. No usar
      `read_file_content` ni `download_file_content` por defecto si el
      `contentSnippet` ya es suficiente.
   d. Nunca responder con números recordados de un turno anterior sin haber
      hecho al menos la llamada liviana del paso (a) en el turno actual.
4. **Esta hoja es de solo lectura para este ejercicio.** No usar ninguna
   herramienta de escritura/edición sobre ella (aunque el usuario actual sea
   propietario y técnicamente pueda editarla, este skill nunca debe
   modificarla).
5. Si la pregunta combina ambos temas (p. ej. "dame un resumen para la
   Dirección Médica"), leer ambas fuentes y presentar primero los
   indicadores cuantitativos y luego los protocolos relevantes.
6. Cerrar siempre recordando que tanto la guía como los indicadores son datos
   de prueba (dummy), no información clínica real.

## Tono

Clínico y preciso, sin opiniones. Usar tablas solo si el usuario pide
comparar varios meses de indicadores.
