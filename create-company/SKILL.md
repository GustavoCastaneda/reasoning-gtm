---
description: >
  Crea empresas en HubSpot usando las fichas de investigación generadas por
  el skill research. Se activa después del research cuando el usuario pide
  crear o subir los leads a HubSpot. Para cada empresa creada, actualiza el
  Google Doc en Drive con el link directo al registro en HubSpot.
---

# Crear Empresa en HubSpot — $ARGUMENTS

Toma las fichas de investigación disponibles en el contexto de esta sesión y crea una empresa en HubSpot por cada lead. No pidas confirmación entre leads — procesa todos de corrido.

Si una ficha no está en el contexto de la sesión, busca su archivo local: `leads/[empresa-slug]/ficha.md` (slug = nombre de la empresa en minúsculas, espacios → guiones, sin acentos). Si el archivo tampoco existe, reporta el lead como pendiente y continúa con los demás.

Portal ID: **51399355**
Object type: **Company**

---

## POR CADA LEAD, CREA LA EMPRESA EN HUBSPOT

Mapea los datos de la ficha a estas propiedades:

| Propiedad HubSpot | Fuente |
|---|---|
| `name` | Nombre de la empresa |
| `domain` | Dominio web (inferido del email o de la investigación) |
| `description` | Campo "Qué hacen" de la ficha |
| `industry` | Campo "Sector" de la ficha |
| `city` | Ciudad (si se encontró en la investigación) |
| `country` | País (si se encontró en la investigación) |
| `numberofemployees` | Tamaño estimado de la ficha |
| `hs_lead_status` | Siempre `NEW` al crear |

Si algún campo no está disponible en la ficha, déjalo vacío — no inventes datos.

---

## DESPUÉS DE CREAR CADA EMPRESA

### 1. Agregar nota en HubSpot
Crea una nota en el registro de la empresa con:
- El contenido completo de la ficha de investigación (industria, competidores, contacto, trigger para outreach).
- **Una línea final fija**: `📄 Google Doc: [URL del Doc tomado del campo GOOGLE DOC > URL de la ficha]`. Esto permite que `/post-llamada` y `/lead-brief` (en sesiones posteriores) recuperen el URL desde HubSpot sin buscar en Drive.

### 2. Actualizar el Google Doc en Drive

Toma el URL del Doc del campo `GOOGLE DOC > URL` de la ficha que dejó `/research` en el contexto. **Abre ese Doc específico (por URL/ID, no por nombre)** y agrega al inicio:

```
🔗 HubSpot Empresa: [link directo al registro de la empresa en HubSpot]
📅 Empresa creada: [fecha]
```

**Reglas estrictas:**
- NO crees un Doc nuevo bajo ninguna circunstancia.
- Si el campo `GOOGLE DOC > URL` no está en la ficha o el URL no resuelve, busca la línea `Path local:` en la ficha — ese archivo `.md` tiene el URL de Drive en su sección GOOGLE DOC. Lee el archivo con `leads/[empresa-slug]/ficha.md` y extrae el URL de ahí.
- Si después de eso aún no tienes URL, **detente y reporta**: "No encuentro el Google Doc del lead [empresa]. Corre primero `/research` o pásame el URL del Doc."
- No busques por nombre en Drive — eso generaba duplicados.

---

## OUTPUT AL TERMINAR

Entrega una tabla confirmando lo que se creó:

| Empresa | HubSpot creado | Link | Doc actualizado |
|---------|---------------|------|-----------------|
| [nombre] | ✅ | [link] | ✅ |