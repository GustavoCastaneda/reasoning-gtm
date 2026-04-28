---
description: >
  Genera un resumen ejecutivo del lead antes de una cotización o reunión
  importante. Se activa cuando el usuario menciona que va a cotizar, que
  tiene una reunión próxima, o pide prepararse para una llamada con un cliente.
  Consulta HubSpot, Granola y Google Calendar para consolidar todo el contexto
  acumulado sobre el lead.
---

# Lead Brief — $ARGUMENTS

Consolida toda la información disponible sobre "$ARGUMENTS" y genera un resumen ejecutivo para entrar preparado a la llamada de cotización.

Ejecuta los 3 pasos de búsqueda en paralelo antes de generar el brief.

---

## PASO 1 — Consultar HubSpot

Busca la empresa "$ARGUMENTS" en HubSpot y extrae:
- Estado actual del deal (`hs_lead_status`)
- Todas las notas del registro — especialmente el expediente post-demo
- Historial de interacciones (emails, llamadas, actividad)
- Contactos asociados y sus roles

---

## PASO 2 — Consultar Granola

Busca todos los transcripts asociados a "$ARGUMENTS" y extrae:
- Fecha y tipo de cada llamada
- Puntos clave de cada conversación
- Citas directas relevantes del prospecto

---

## PASO 3 — Consultar Google Calendar

Busca reuniones próximas o recientes con personas de "$ARGUMENTS":
- Fecha y hora de la próxima reunión
- Participantes confirmados
- Tipo de reunión

---

## PASO 4 — Generar el brief

Con toda la información recopilada, entrega este resumen en el chat:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
LEAD BRIEF — [Empresa]
Preparado: [fecha y hora]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

EL CONTACTO
• Champion: [nombre] | [puesto]
• Decision maker: [nombre] | [puesto] (si se conoce)
• Empresa: [nombre] | [industria] | [tamaño estimado]

PRÓXIMA REUNIÓN
• [fecha y hora] — [tipo de reunión] — [participantes]

HISTORIAL
• [fecha] — [tipo de interacción] — [resumen 1 línea]
• [fecha] — [tipo de interacción] — [resumen 1 línea]

STACK TECNOLÓGICO
• Bases de datos: [dato]
• BI: [dato]
• ERP: [dato]
• Complejidad estimada: [Alta / Media / Baja]

EL DOLOR
• [punto de dolor 1]
• [punto de dolor 2]
• Urgencia: [Alta / Media / Baja]

REACCIÓN A LA DEMO
• [resumen de cómo reaccionaron]
• Objeciones levantadas: [lista]

EL EQUIPO DE DATOS
• Tamaño: [dato]
• Roles: [dato]
• Decision maker confirmado: [nombre y título]

CITAS DIRECTAS DEL PROSPECTO
• "[cita 1]"
• "[cita 2]"

CONTEXTO PARA LA COTIZACIÓN
• Siguiente paso acordado: [dato]
• Objeciones que pueden surgir: [lista]
• Lo que más les importó de la demo: [dato]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Si algún campo no tiene información disponible, escríbelo como "Sin dato" — no inventes información.