---
description: >
  Genera el DM de LinkedIn post-conexión para un lead usando un loop de
  refinamiento entre tres agentes especializados. Se activa cuando el
  usuario pide el mensaje de LinkedIn después de que el prospecto aceptó
  la conexión. Toma la ficha de investigación del contexto o del archivo
  local. Output: texto en el chat para copiar y pegar directamente en
  LinkedIn — no crea draft en Gmail.
allowed-tools:
  - Skill
  - Read
  - Edit
---

# LinkedIn Outreach — $ARGUMENTS

Toma la ficha de investigación del lead disponible en el contexto. Si se especifica un nombre de empresa en $ARGUMENTS, usa solo la ficha de ese lead.

Si la ficha no está en el contexto de la sesión, busca el archivo local: `leads/[empresa-slug]/ficha.md` (slug = nombre de la empresa en minúsculas, espacios → guiones, sin acentos). Si el archivo tampoco existe, detente: "No encuentro la ficha de [empresa]. Corre primero `/research` o pega la ficha en el chat."

**Foco: DM post-conexión (Día 1).** No generes follow-ups ni mensajes de reconexión — esos se trabajan con contexto fresco si hay o no respuesta.

**Output: texto en el chat para copiar y pegar en LinkedIn directamente.** No creas draft en Gmail.

Ejecuta el loop de refinamiento completo antes de mostrar cualquier resultado en el chat. El founder solo ve el DM cuando alcanza 8.5 o después de 3 iteraciones.

---

## FLUJO COMPLETO

### Paso 1 — LinkedIn Writer

Invoca al `linkedin-writer-agent` con:
- La ficha de investigación del lead
- El archivo `outreach/references/linkedin-rules.md`

El Writer produce el primer draft con su autocrítica.

---

### Paso 2 — LinkedIn Critic

Invoca al `linkedin-critic-agent` con:
- La ficha de investigación del lead
- El archivo `outreach/references/linkedin-rules.md`
- El draft del Writer incluyendo su autocrítica

El Critic produce el reporte de revisión con recomendaciones.

---

### Paso 3 — LinkedIn Writer reescribe (si el Critic recomienda)

Si el Critic veredicta "Sí" a reescribir, invoca al `linkedin-writer-agent` nuevamente con:
- La ficha de investigación del lead
- El archivo `outreach/references/linkedin-rules.md`
- El draft anterior
- El reporte completo del Critic

Si el Critic veredicta "No" (draft está bien), salta directamente al Paso 4.

---

### Paso 4 — LinkedIn Judge

Invoca al `linkedin-judge-agent` con:
- La ficha de investigación del lead
- El archivo `outreach/references/linkedin-rules.md`
- El draft más reciente del Writer
- El reporte del Critic

El Judge produce la calificación y el veredicto desde la perspectiva del prospecto.

---

### Paso 5 — Decisión del loop

```
¿Calificación ≥ 8.5?
    Sí → ir a Paso 6
    No → ¿iteraciones < 3?
              Sí → regresar al Paso 2 con el feedback del Judge (Critic vuelve a revisar)
              No → ir al Paso 6 con el mejor draft alcanzado
```

El loop máximo es 3 iteraciones (cada iteración = Critic → Writer reescribe → Judge). Si después de 3 iteraciones no se alcanza 8.5, se entrega el mejor draft con nota de calificación.

---

### Paso 6 — Mostrar en el chat

Muestra el progreso y el resultado final:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
LINKEDIN DM: [Empresa] — [Contacto] | [Puesto]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PROCESO DE REFINAMIENTO
• Iteración 1: [calificación del Judge]
• Iteración 2: [calificación del Judge] (si aplica)
• Iteración 3: [calificación del Judge] (si aplica)
• Calificación final: [X.X] / 10

─────────────────────────────────────────

💬 DM POST-CONEXIÓN — Día 1

[cuerpo completo del DM — solo texto, listo para copiar y pegar]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
¿Apruebas este DM? Responde "sí" para confirmar.
```

---

### Paso 7 — Después del visto bueno

Cuando el usuario apruebe, ejecuta sin pedir más confirmación:

**HubSpot:**
- Cambia `hs_lead_status` a `ATTEMPTED_TO_CONTACT`
- Agrega nota: "LinkedIn DM Día 1 aprobado — [fecha]. Calificación final: [X.X]/10."

**Confirmar en el chat:**

| Empresa | DM aprobado | HubSpot | Corpus |
|---------|-------------|---------|--------|
| [nombre] | ✅ Texto copiado | ✅ ATTEMPTED_TO_CONTACT | ✅ Entrada #N |

---

### Paso 7.5 — Aprendizaje automático: agregar entrada al corpus de LinkedIn

Sin pedir confirmación, genera y agrega una entrada al corpus de LinkedIn inmediatamente después de confirmar la tabla del Paso 7.

**Qué extraer de la sesión:**

- **DM aprobado**: el que el usuario dijo "sí" — usa la versión final que el usuario aceptó.
- **Empresa**: preservar sin anonimizar.
- **Contacto**: reemplazar el nombre real con `[Contacto]`.
- **Calificación**: la del Judge sobre el draft más cercano al aprobado.
- **Principal aprendizaje**: qué cambió del primer draft al aprobado.

**Leer el corpus para obtener el número siguiente:**

Lee `outreach/references/linkedin-corpus.md` y encuentra la última entrada `## #N —`. La nueva entrada es `#N+1`. Si el corpus está vacío (solo tiene el índice), la primera entrada es `#1`.

**Actualizar el índice:**

Agrega una fila al índice en `outreach/references/linkedin-corpus.md`:
```
| #N+1 | [vertical] | [rol] | [patrón corto] | Aprobado |
```

**Formato de la entrada a agregar:**

```markdown
## #[N+1] — [Patrón corto de 3–6 palabras] ([vertical])

**Prospecto**: [Rol] en **[Empresa]** ([descripción de 1 línea del vertical]).

**DM aprobado por el [Judge / usuario] ([score]/10)**:

> [cuerpo verbatim — nombre del contacto → [Contacto]]

([N] palabras.)

**✅ Lo que funcionó**:
• [elemento específico]

**❌ Lo que se rechazó / Lo que el usuario corrigió**:
• [qué se detectó + cita del feedback si existe]

**📝 Lección**: [principio internalizable en 2–4 líneas]
```

**Dónde insertar**: usa `Edit` para agregar la entrada justo **antes** de la línea `## Cómo se agregan entradas nuevas` en `outreach/references/linkedin-corpus.md`.

**Si no hubo rechazos** (Judge aprobó en primera iteración): omite la sección `❌` y documenta solo `✅` + lección.

**Confirmar** actualizando la tabla del Paso 7 con `✅ Entrada #N+1` en la columna Corpus.
