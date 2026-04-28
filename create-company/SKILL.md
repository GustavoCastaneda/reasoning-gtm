---
description: >
  Crea empresas en HubSpot usando las fichas de investigación generadas por
  el skill research. Se activa después del research cuando el usuario pide
  crear o subir los leads a HubSpot. Para cada empresa creada, actualiza el
  Google Doc en Drive con el link directo al registro en HubSpot.
---

# Crear Empresa en HubSpot — $ARGUMENTS

Toma las fichas de investigación disponibles en el contexto de esta sesión y crea una empresa en HubSpot por cada lead. No pidas confirmación entre leads — procesa todos de corrido.

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
Crea una nota en el registro de la empresa con el contenido completo de la ficha de investigación — industria, competidores, contacto, trigger para outreach.

### 2. Actualizar el Google Doc en Drive
Busca el Doc `[Empresa] — Lead Research` en la carpeta "Leads activos" y agrega al inicio:

```
🔗 HubSpot: [link directo al registro de la empresa]
📅 Creado: [fecha]
```

---

## OUTPUT AL TERMINAR

Entrega una tabla confirmando lo que se creó:

| Empresa | HubSpot creado | Link | Doc actualizado |
|---------|---------------|------|-----------------|
| [nombre] | ✅ | [link] | ✅ |