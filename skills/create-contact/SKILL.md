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
| `hs_linkedin_ad_last_used_date` | URL de LinkedIn |
| `lifecyclestage` | Siempre `lead` al crear |

### 3. Vincular el contacto a la empresa
Asocia el contacto recién creado al registro de la empresa en HubSpot.

### 4. Agregar nota en el contacto
Crea una nota con los datos del contacto de la ficha de investigación:
- Responsabilidades reales
- Tiempo en el puesto
- Publicación reciente relevante
- Señal de dolor identificada
- Trigger para outreach

---

## OUTPUT AL TERMINAR

| Contacto | Empresa | HubSpot creado | Vinculado |
|----------|---------|---------------|-----------|
| [nombre] | [empresa] | ✅ | ✅ |