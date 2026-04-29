---
description: >
  Crea contactos en HubSpot y los vincula a su empresa. Se activa después de
  create-company o cuando el usuario pide crear o subir un contacto. Siempre
  requiere empresa + contacto — si solo llega un contacto, pregunta a qué
  empresa pertenece antes de continuar.
---

# Crear Contacto en HubSpot — $ARGUMENTS

Crea el contacto en HubSpot y vincúlalo a su empresa. No pidas confirmación — procesa de corrido.

Portal ID: **51399355**

---

## REGLA PRINCIPAL

Si recibes un contacto sin empresa asociada, detente y pregunta:
> "¿A qué empresa pertenece este contacto?"

No continues hasta tener la empresa.

---

## FLUJO

### 1. Verificar si la empresa existe en HubSpot
Busca la empresa por nombre o dominio.

- **Existe** → usa ese registro para vincular el contacto
- **No existe** → avisa: "La empresa [nombre] no está en HubSpot. Corre primero `create-company` y luego regresa aquí."

### 2. Crear el contacto

Mapea los datos de la ficha a estas propiedades:

| Propiedad HubSpot | Fuente |
|---|---|
| `firstname` / `lastname` | Nombre del contacto |
| `jobtitle` | Puesto |
| `email` | Work email |
| `linkedin_profile_url` | URL de LinkedIn (custom property — ver nota abajo) |
| `lifecyclestage` | Siempre `lead` al crear |

> **Nota sobre LinkedIn URL**: `linkedin_profile_url` es la custom property estándar para URLs de perfil en HubSpot. Si el portal 51399355 no tiene esta propiedad creada, el API call devolverá error en ese campo — en ese caso omite el campo LinkedIn y continúa con el resto (no detener el flujo). La propiedad se crea en HubSpot → Configuración → Propiedades → Contactos → Nueva propiedad → tipo Texto de una línea, nombre interno `linkedin_profile_url`.
>
> **¿Por qué no `hs_linkedin_ad_last_used_date`?** Esa es una propiedad built-in de HubSpot para registrar la última vez que se usó un anuncio de LinkedIn — es un timestamp, no un campo de texto para URLs. Los valores guardados ahí no aparecen como links en el perfil del contacto.

### 3. Vincular el contacto a la empresa
Asocia el contacto recién creado al registro de la empresa en HubSpot.

### 4. Agregar nota en el contacto
Crea una nota con los datos del contacto de la ficha de investigación:
- Responsabilidades reales
- Tiempo en el puesto
- Publicación reciente relevante
- Señal de dolor identificada
- Trigger para outreach

### 5. Actualizar el Google Doc en Drive

Toma el URL del Doc del campo `GOOGLE DOC > URL` de la ficha que dejó `/research`. **Abre ese Doc específico (por URL/ID, no por nombre)** y agrega al inicio, justo debajo del bloque de empresa que dejó `/create-company`:

```
🔗 HubSpot Contacto: [link directo al contacto en HubSpot]
📅 Contacto creado: [fecha]
```

**Reglas estrictas:**
- NO crees un Doc nuevo bajo ninguna circunstancia.
- Si no tienes el URL en el contexto pero la empresa ya existe en HubSpot, busca la empresa en HubSpot y lee la línea `📄 Google Doc: ...` de la nota inicial — ese es el URL.
- Si después de eso aún no tienes URL, detente y reporta: "No encuentro el Google Doc del lead [empresa]. Confirma con el usuario antes de continuar."
- No busques por nombre en Drive como fallback.

---

## OUTPUT AL TERMINAR

| Contacto | Empresa | HubSpot creado | Vinculado | Doc actualizado |
|----------|---------|----------------|-----------|-----------------|
| [nombre] | [empresa] | ✅ | ✅ | ✅ |