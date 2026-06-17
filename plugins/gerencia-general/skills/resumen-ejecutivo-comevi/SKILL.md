---
name: resumen-ejecutivo-comevi
description: >
  Este skill debe usarse cuando el usuario pida "el resumen ejecutivo de Comevi",
  "cómo van los indicadores generales", "un reporte para gerencia", "el estado
  general de la organización", "los KPIs del mes" o "un dashboard ejecutivo".
  Cubre ingresos, ocupación, satisfacción del paciente y otros indicadores
  consolidados de Comevi a nivel de toda la organización.
metadata:
  version: "0.1.0"
  area: "Gerencia General"
---

# Resumen ejecutivo de Comevi

Generar un resumen ejecutivo consolidado para la Gerencia General de Comevi.

## Datos de referencia

Leer `references/kpis-comevi.md` para obtener los indicadores base. Esos datos
son **ficticios (dummy)**, creados únicamente para esta prueba de configuración
de plugins — nunca presentarlos como datos reales de operación.

## Instrucciones

1. Leer la tabla de KPIs en `references/kpis-comevi.md`.
2. Identificar el mes más reciente disponible y compararlo contra el mes anterior.
3. Estructurar la respuesta en estas secciones, en este orden:
   - Ingresos y facturación
   - Ocupación de servicios
   - Satisfacción del paciente (NPS)
   - Alertas o desviaciones relevantes (cualquier indicador que haya caído más
     de 5 puntos porcentuales respecto al mes anterior)
4. Cerrar siempre con una nota indicando que los datos son de prueba/dummy y
   que el conector real de BI (`bi-dashboard-dummy`) todavía no está conectado
   a un sistema productivo.
5. Si el usuario pide profundizar en un área específica (médica, comercial,
   operativa, RH o facturación), sugerir explícitamente instalar el plugin
   correspondiente de Comevi en lugar de inventar datos de esa área.

## Tono

Directo y ejecutivo. Usar tablas solo si el usuario pide comparar varios meses;
de lo contrario, prosa breve con los números clave resaltados.
