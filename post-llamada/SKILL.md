---
description: >
  Procesa el transcript de una llamada en Granola y agrega la información al
  expediente acumulativo del cliente en HubSpot. Se activa cuando el usuario
  menciona que acaba de tener una llamada o reunión con un cliente. Recibe el
  nombre de la empresa y el propósito de la llamada. Formato de invocación:
  "Empresa | propósito de la llamada" (ej: "Financiera ABC | llamada de mapeo").
---

# Post-Llamada — $ARGUMENTS

Formato esperado: `[Empresa] | [propósito de la llamada]`

Ejemplos válidos:
- `Financiera ABC | llamada de mapeo`
- `Distribuidora XYZ | demo`
- `Retail Norte | llamada con el CTO`
- `Empresa X | revisión de piloto`

Si no recibes el propósito, pregunta antes de continuar: "¿Cuál fue el propósito de esta llamada?"

---

## PASO 1 — Buscar el transcript en Granola

Busca en Granola el transcript más reciente asociado a la empresa. Si no encuentras ninguno, avisa: "No encontré un transcript reciente para [empresa] en Granola. ¿Puedes confirmar el nombre o pegarlo directamente?"

---

## PASO 2 — Extraer información de la llamada

Del transcript, extrae lo relevante según el propósito indicado. No todos los campos aplican a todas las llamadas — extrae lo que se mencionó:

**Participantes**
- Quién estuvo en la llamada (nombres y roles)
- Si apareció alguien nuevo, ¿cuál es su rol en la decisión?

**Stack tecnológico** (si se mencionó)
- Nuevas herramientas o fuentes identificadas
- Cambios respecto a lo que ya se sabía

**Flujo de datos** (si se mencionó)
- Nuevos detalles sobre cómo operan hoy
- Tiempo en plomería actualizado

**El dolor** (si se mencionó)
- Nuevos puntos de dolor o confirmación de los existentes
- Cambios en urgencia

**Complejidad operacional** (si se mencionó)
- Nuevas sucursales, líneas de negocio o sistemas identificados

**Reacción** (si aplica)
- Cómo reaccionaron a lo que se presentó
- Preguntas u objeciones nuevas

**Siguiente paso acordado**
- Qué se acordó al final de esta llamada
- Timeline

**Citas directas relevantes**
- Frases textuales del prospecto

---

## PASO 3 — Actualizar HubSpot

En el registro de la empresa (Portal ID: 51399355):

### 1. Cambiar `hs_lead_status` según el propósito:
| Propósito | Status |
|-----------|--------|
| Llamada de mapeo | `OPEN` |
| Demo | `DEMO_DONE` |
| Revisión de piloto | `IN_PROGRESS` |
| Llamada de seguimiento | mantener el status actual |
| Otro | mantener el status actual |

### 2. Agregar nota nueva — no reemplazar las anteriores
Cada llamada es una entrada nueva en el expediente. Formato de la nota:

```
[LLAMADA — propósito | fecha]

Participantes: [lista]

[campos relevantes extraídos del transcript]

Siguiente paso: [acuerdo] — [timeline]
```

---

## PASO 4 — Actualizar el Google Doc en Drive

1. **Localiza el URL del Doc** vía la nota inicial del registro de la empresa en HubSpot — busca la línea `📄 Google Doc: [URL]` que dejó `/create-company` cuando se creó la empresa.
2. **Abre ese Doc específico (por URL/ID, no por nombre)** y agrega una nueva sección AL FINAL:

```
## [Propósito de la llamada] — [fecha]
[contenido del expediente de esta llamada]
```

**Reglas estrictas:**
- No borres ni modifiques las secciones anteriores — solo agrega al final.
- NO crees un Doc nuevo bajo ninguna circunstancia. Si la nota de HubSpot no tiene la línea `📄 Google Doc:` o el URL no resuelve, **detente y reporta**: "No encuentro el Google Doc del lead [empresa]. Verifica que `/research` y `/create-company` se hayan corrido para esta empresa."
- No busques por nombre en Drive como fallback — eso generaba duplicados.

---

## PASO 5 — Mostrar en el chat

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[PROPÓSITO] — [Empresa]
Fecha: [fecha del transcript]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PARTICIPANTES
• [nombre] | [rol]

[secciones relevantes extraídas]

SIGUIENTE PASO
• [acuerdo] — [timeline]

CITAS DIRECTAS
• "[cita]"
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
HubSpot: ✅ Nota agregada | Status: [nuevo status]
Google Doc: ✅ Sección agregada
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```