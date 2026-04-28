---
description: >
  Orquestador GTM — entry point del flujo completo en lenguaje natural.
  Toma instrucciones tipo "investiga este lead", "mete esto a HubSpot",
  "corre el flujo completo hasta el draft en mi gmail", "procesa la
  llamada de X", "brief para cotizar a Y", y encadena los skills
  granulares correctos en orden. Acepta tabla pegada, archivo CSV o
  XLSX, o nombres sueltos. Úsalo cuando quieras hacer cualquier cosa de
  GTM sin elegir skill manualmente.
allowed-tools:
  - Skill
  - Read
  - Bash
  - AskUserQuestion
---

# /gtm — Orquestador GTM

Entry point del flujo completo de Reasoning Labs. Recibe `$ARGUMENTS` + cualquier contenido pegado o archivo adjunto, decide qué sub-skills llamar y en qué orden, y los ejecuta secuencialmente.

**Regla #0 — solo ejecuta lo que el usuario pidió.** No auto-encadenes hasta `/outreach` si solo pidió "investiga". Identifica el stop point antes de actuar.

---

## Paso 1 — Parsear intención + payload

Lee `$ARGUMENTS` y todo lo que el usuario haya pegado o adjuntado en el mensaje. En paralelo, identifica dos cosas:

### A) **Stop point** — hasta qué skill llegar

| Frase del usuario (o equivalente semántico) | Cadena |
|---|---|
| "investiga", "research", "haz research", "investígame este lead" | `/research` (stop) |
| "mete a HubSpot", "crea empresas", "crea en HubSpot", "investiga + HubSpot" | `/research` → `/create-company` → `/create-contact` |
| "outreach", "prepara secuencia", "manda outreach", "flujo completo", "hasta outreach", **"hasta el draft en mi gmail"**, "hasta gmail", "hasta el draft" | `/research` → `/create-company` → `/create-contact` → `/outreach` |
| "procesa la llamada", "post-llamada", "actualiza con la demo", "guarda la llamada de X" | `/post-llamada` |
| "brief", "prepárame para cotizar", "voy a cotizar X", "tengo reunión con X" | `/lead-brief` |

Reglas de mapeo:
- "hasta el draft en mi gmail" / "hasta gmail" / "hasta el draft" → equivale a llegar a `/outreach` (es el skill que produce los Gmail drafts).
- "flujo completo" sin contexto adicional → cadena `/research` → `/create-company` → `/create-contact` → `/outreach`.
- Si el usuario nombra explícitamente un skill ("solo /research", "no quiero outreach todavía") → respeta el límite.
- Si pide algo más allá del último skill ("manda los emails de verdad", "envía") → aclara que `/outreach` solo crea drafts en Gmail, no los envía.
- Si la frase no mapea claro a ninguna fila → **brincar a Paso 3 (Confirmación)**.

### B) **Formato del payload**

| Formato detectado | Cómo identificarlo |
|---|---|
| **Tabla markdown pegada** | El mensaje contiene líneas con `|` y headers como `empresa\|contacto\|puesto\|email\|linkedin` |
| **Archivo CSV** | `$ARGUMENTS` o el mensaje menciona una ruta terminada en `.csv`, o hay un adjunto `.csv` |
| **Archivo XLSX/XLS** | Ruta terminada en `.xlsx` o `.xls` |
| **Nombre suelto de empresa o lead** | Texto corto sin tabla ni archivo (ej. "Financiera ABC") |
| **Nada** | Solo verbo, sin payload |

---

## Paso 2 — Normalizar payload

Solo aplica si la cadena requiere `/research` (las cadenas que arrancan con investigación).

### Si el payload es **tabla markdown pegada**
Pásala tal cual al primer skill de la cadena. No la transformes.

### Si el payload es **CSV** (ruta de archivo o adjunto)
1. Léelo con `Read` (si la ruta es absoluta) o `Bash`: `cat "<ruta>"`.
2. Valida que las 5 columnas requeridas existan: `empresa`, `contacto`, `puesto`, `work email` (o `email`), `linkedin`. Tolera variaciones de case y espacios.
3. Si **falta alguna columna** → para inmediatamente y dile al usuario:
   ```
   El CSV tiene <columnas que sí trae> pero le faltan: <columnas faltantes>.
   /research necesita las 5 para procesar el lead. ¿Puedes regenerar el archivo o agregarlas?
   ```
4. Si todas las columnas existen → conviértelo a tabla markdown:
   ```bash
   python3 -c "
   import csv, sys
   with open('<ruta>') as f:
       rows = list(csv.reader(f))
   header = rows[0]
   print('| ' + ' | '.join(header) + ' |')
   print('|' + '|'.join(['---'] * len(header)) + '|')
   for r in rows[1:]:
       print('| ' + ' | '.join(r) + ' |')
   "
   ```
5. Pasa la tabla resultante al primer skill de la cadena.

### Si el payload es **XLSX/XLS**
1. Convertir con pandas:
   ```bash
   python3 -c "
   import pandas as pd
   df = pd.read_excel('<ruta>')
   print(df.to_markdown(index=False))
   " 2>&1
   ```
2. Si pandas falla (no instalado) → dile al usuario: `pandas no está disponible — exporta el archivo a CSV y vuelve a intentar.`
3. Si la conversión funciona → aplica la misma validación de columnas que en CSV.

### Si el payload es **nombre suelto**
- Si la cadena arranca con `/research` → no es válido, pide la tabla: `Para investigar necesito la tabla con empresa, contacto, puesto, email y LinkedIn. ¿Me la pegas?`
- Si la cadena es `/post-llamada` o `/lead-brief` → pásalo como `$ARGUMENTS` directo (esos skills aceptan nombre de empresa).

### Si **no hay payload** y la cadena lo requiere
Brinca a Paso 3 (Confirmación) para preguntar qué quiere el usuario.

---

## Paso 3 — Confirmar si la intención es ambigua

Si la frase del usuario no mapea claro a una fila, o no hay payload cuando se requiere, usa `AskUserQuestion` con una sola pregunta y estas 5 opciones:

| Header | Description |
|---|---|
| Investigar leads nuevos | `/research` desde tabla/CSV/xlsx — fichas de cada lead + Doc en Drive |
| Investigar + crear en HubSpot | `/research` → `/create-company` → `/create-contact` |
| Flujo completo hasta outreach | `/research` → empresas → contactos → drafts en Gmail |
| Procesar una llamada | `/post-llamada` — agrega transcript de Granola al expediente |
| Brief pre-cotización | `/lead-brief` — consolida HubSpot + Granola + Calendar para una empresa |

No avances al Paso 4 sin una respuesta. Si la respuesta requiere payload (las primeras 3 opciones), pídelo.

---

## Paso 4 — Ejecutar la cadena

Para cada skill en la cadena, en orden estricto:

1. Anuncia al usuario qué skill vas a invocar y por qué (1 línea cada uno).
2. Invoca el skill via `Skill` tool. Para el primer skill, pasa el payload normalizado en `args`. Para los siguientes, NO pases args — leen las fichas del contexto de conversación que dejó el skill anterior.
3. Espera a que termine y el output esté en el contexto antes de seguir.
4. **No insertes confirmaciones entre skills.** Los sub-skills tienen sus propios gates donde importa: `/outreach` pide aprobación antes de crear los Gmail drafts; `/post-llamada` antes de actualizar HubSpot. Respeta esos gates pero no añadas extras.

### Excepciones que cortan la cadena

- Si `/research` no encuentra suficiente info para un lead → ese lead se marca como incompleto pero la cadena sigue con los demás (research ya itera internamente).
- Si `/create-company` falla porque la empresa ya existe en HubSpot → no es error, se reutiliza el record existente y la cadena continúa.
- Si `/create-contact` se queja de que falta empresa → para la cadena para ese lead específico y reporta.
- Si `/outreach` detecta que el judge no llega a 8.5 después de 3 iteraciones → entrega lo que tenga + reporta el score, no bloquea la cadena.

---

## Paso 5 — Resumen final

Al terminar todos los skills de la cadena, presenta al usuario una tabla compacta:

```
| Skill | Status | Output |
|-------|--------|--------|
| /research | ✅ | 3 fichas + 3 Google Docs en "Leads activos" |
| /create-company | ✅ | 3 empresas creadas en HubSpot |
| /create-contact | ✅ | 3 contactos creados y vinculados |
| /outreach | ✅ | 3 secuencias en Gmail drafts (scores: 9.0, 8.7, 8.5) |
```

Y un cierre de 1–2 líneas con qué falta por hacer manualmente (ej. "Los drafts están en tu Gmail — revísalos antes de mandar.").

---

## Reglas heredadas del gtm-agent

Estas reglas vienen de `agents/gtm-agent.md` y aplican a todo lo que `/gtm` orqueste:

- Máximo 10 leads por sesión en `/research`.
- Si un contacto llega sin empresa, pregunta a qué empresa pertenece antes de crear.
- Outreach necesita mínimo 8.5/10 del Judge — máximo 3 iteraciones.
- `/post-llamada` acumula, nunca reemplaza información existente en el expediente.
- Idioma: español para LATAM, inglés para US — nunca mezclar idiomas en el mismo mensaje de outreach.
- Antes de cualquier acción sobre un lead conocido, consulta HubSpot, Granola y Calendar para no escribir sobre información obsoleta.

## Nota de mantenimiento

La tabla de routing y las reglas anteriores son un **mirror compacto** de `agents/gtm-agent.md` líneas 122–172. Si se agrega un skill nuevo o se cambia una regla allá, hay que actualizar este SKILL.md en paralelo.
