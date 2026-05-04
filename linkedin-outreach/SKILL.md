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
- El archivo `outreach/references/voice-profile.md`

El Writer produce el primer draft con su autocrítica.

---

### Paso 2 — LinkedIn Critic + LinkedIn Judge en paralelo

Invoca **simultáneamente** al `linkedin-critic-agent` y al `linkedin-judge-agent`. Ambos reciben:
- La ficha de investigación del lead
- El archivo `outreach/references/linkedin-rules.md`
- El draft más reciente del Writer (con su autocrítica)

El Critic evalúa calidad, tono, mecanismo y cumplimiento de reglas.
El Judge evalúa el impacto desde la perspectiva del prospecto y asigna calificación 1–10.

**Nota:** el Judge evalúa el draft como el prospecto lo vería — sin ver el reporte del Critic. Esto es correcto: el prospecto tampoco lo vería. El Writer sí recibe ambos reportes en la siguiente iteración.

---

### Paso 3 — Decisión del loop

```
¿Calificación del Judge ≥ 8.5?
    Sí → ir a Paso 4 (entregar)
    No → ¿iteraciones completadas < 3?
              Sí → LinkedIn Writer reescribe con AMBOS reportes (Critic + Judge) → regresar a Paso 2
              No → ir a Paso 4 con el mejor draft alcanzado
```

El loop máximo es 3 iteraciones (cada iteración = [Critic+Judge simultáneos] → Writer reescribe si aplica). Si después de 3 iteraciones no se alcanza 8.5, se entrega el mejor draft con nota de calificación.

**Al reescribir**, invoca al `linkedin-writer-agent` con:
- La ficha de investigación del lead
- El archivo `outreach/references/linkedin-rules.md`
- El archivo `outreach/references/voice-profile.md`
- El draft anterior
- El reporte completo del Critic
- El reporte completo del Judge (incluye calificación y mejoras específicas)

---

### Paso 4 — Mostrar en el chat

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

### Paso 5 — Después del visto bueno

Cuando el usuario apruebe, ejecuta sin pedir más confirmación:

**HubSpot:**
- Cambia `hs_lead_status` a `ATTEMPTED_TO_CONTACT`
- Agrega nota: "LinkedIn DM Día 1 aprobado — [fecha]. Calificación final: [X.X]/10."

**Confirmar en el chat:**

| Empresa | DM aprobado | HubSpot | Corpus |
|---------|-------------|---------|--------|
| [nombre] | ✅ Texto copiado | ✅ ATTEMPTED_TO_CONTACT | ✅ Entrada #N |

---

### Paso 5.5 — Aprendizaje automático: agregar entrada al corpus de LinkedIn

Sin pedir confirmación, genera y agrega una entrada al corpus de LinkedIn inmediatamente después de confirmar la tabla del Paso 5.

**Qué extraer de la sesión:**

- **DM aprobado**: el que el usuario dijo "sí" — usa la versión final que el usuario aceptó.
- **Empresa**: preservar sin anonimizar.
- **Contacto**: reemplazar el nombre real con `[Contacto]`.
- **Calificación**: la del Judge sobre el draft más cercano al aprobado.
- **Principal aprendizaje**: qué cambió del primer draft al aprobado. Si Judge aprobó en primera iteración, documenta qué funcionó.

**Leer el corpus para obtener el número siguiente:**

Lee `outreach/references/linkedin-corpus.md` y encuentra la última entrada `## #N —`. La nueva entrada es `#N+1`.

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

**Confirmar** actualizando la tabla del Paso 5 con `✅ Entrada #N+1` en la columna Corpus.

---

### Paso 5.7 — Oferta de actualizar el Voice Profile (si score ≥9.0)

Si el Judge aprobó con calificación ≥9.0 en la primera o segunda iteración, pregunta al usuario:

```
¿Quieres actualizar el voice profile con el patrón de este DM?
El DM obtuvo [X.X]/10 en iteración [N] — hay un patrón ganador que vale capturar.
Responde "sí" para agregar las frases clave a voice-profile.md.
```

Si el usuario dice sí, agrega en `outreach/references/voice-profile.md`:
- En "Mensajes de referencia de alta puntuación": una fila nueva con el corpus ID y por qué es referencia
- En "Frases características" si hay frases nuevas que no estaban documentadas
