---
description: >
  Genera el DM de LinkedIn post-conexión para un lead. Invoca al
  linkedin-dm-agent (agente único que escribe, se auto-evalúa y
  reescribe si aplica). Output: texto en el chat para copiar y pegar
  directamente en LinkedIn — no crea draft en Gmail.
allowed-tools:
  - Skill
  - Read
  - Edit
---

# LinkedIn Outreach — $ARGUMENTS

Toma la ficha de investigación del lead disponible en el contexto. Si se especifica un nombre de empresa en $ARGUMENTS, usa solo la ficha de ese lead.

Si la ficha no está en el contexto de la sesión, busca el archivo local: `leads/[empresa-slug]/ficha.md` (slug = nombre de la empresa en minúsculas, espacios → guiones, sin acentos). Si el archivo tampoco existe, detente: "No encuentro la ficha de [empresa]. Corre primero `/research` o pega la ficha en el chat."

**Foco: DM post-conexión (Día 1).** No generes follow-ups ni mensajes de reconexión.

**Output: texto en el chat para copiar y pegar en LinkedIn directamente.** No creas draft en Gmail.

---

## FLUJO COMPLETO

### Paso 0 — Detectar modo del DM

Antes de invocar al agente, determina el modo:

**MODO COMPLETO** (default): El founder no ha contactado al prospecto aún. El DM incluye T0 (saludo) + T1 + T2 + T3 + T4.

**MODO POST-RESPUESTA**: El founder ya envió el saludo y recibió respuesta. El DM incluye T1 + T2 + T3 + T4 — sin T0.

**Cómo detectar** (revisar $ARGUMENTS y el mensaje del usuario):
- "ya le mandé el saludo", "ya me respondió", "ya conectamos y respondió", "post-respuesta", "ya respondió el saludo" → **MODO POST-RESPUESTA**
- $ARGUMENTS contiene `post-respuesta` o `pitch` → **MODO POST-RESPUESTA**
- Sin contexto de conversación previa → **MODO COMPLETO**

Anuncia el modo en 1 línea antes de invocar al agente:
- `"Generando DM completo (saludo + pitch) para [Empresa]."`
- `"Generando mensaje de pitch post-respuesta para [Empresa] — sin saludo inicial."`

---

### Paso 1 — Invocar linkedin-dm-agent

Invoca al `linkedin-dm-agent` con:
- La ficha de investigación del lead
- El archivo `outreach/references/linkedin-rules.md`
- El archivo `outreach/references/voice-profile.md`
- **El modo detectado en Paso 0**: COMPLETO o POST-RESPUESTA
- **Contexto de iteraciones previas** (si aplica): si la ficha contiene una sección `## Iteraciones previas`, incluirla como contexto adicional al agente con el prefijo: `"Contexto adicional — lecciones de intentos previos con este lead: [contenido]"`

El agente escribe el DM, se auto-evalúa, reescribe una vez si score < 8.5, y entrega el DM con el bloque SELF-EVALUATION incluido.

---

### Paso 2 — Mostrar en el chat

Presenta el resultado al founder:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
LINKEDIN DM: [Empresa] — [Contacto] | [Puesto]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[Modo: COMPLETO / POST-RESPUESTA]
Score: [X.X] / 10[⚠️ si < 8.5]
[Si hubo reescritura: "Reescribí porque: [razón]"]

─────────────────────────────────────────

💬 DM POST-CONEXIÓN — Día 1

[cuerpo completo del DM]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
¿Apruebas este DM? Responde "sí" para confirmar.
```

Si score < 8.5, incluye nota: `"⚠️ Score debajo del umbral (8.5). El agente no pudo mejorarlo en la reescritura — revisa antes de enviar o pide ajustes específicos."`

---

### Paso 3 — Después del visto bueno

Cuando el usuario apruebe, ejecuta sin pedir más confirmación:

**HubSpot:**
- Cambia `hs_lead_status` a `ATTEMPTED_TO_CONTACT`
- Agrega nota: "LinkedIn DM Día 1 aprobado — [fecha]. Score: [X.X]/10."

**Confirmar en el chat:**

| Empresa | DM aprobado | HubSpot | Corpus |
|---------|-------------|---------|--------|
| [nombre] | ✅ Texto listo | ✅ ATTEMPTED_TO_CONTACT | ✅ Entrada #N |

---

### Paso 3.5 — Aprendizaje automático: agregar entrada al corpus

Sin pedir confirmación, agrega una entrada al corpus de LinkedIn inmediatamente después de confirmar la tabla del Paso 3.

**Qué extraer:**
- **DM aprobado**: versión final que el usuario aceptó.
- **Empresa**: preservar sin anonimizar.
- **Contacto**: reemplazar nombre real con `[Contacto]`.
- **Score**: el del agente sobre el draft aprobado.
- **Principal aprendizaje**: qué cambió del primer draft al aprobado. Si el agente reescribió, qué falló. Si el usuario corrigió algo post-agente, qué y por qué.

**Leer el corpus para obtener el número siguiente:**
Lee `outreach/references/linkedin-corpus.md` y encuentra la última entrada `## #N —`. La nueva entrada es `#N+1`.

**Actualizar el índice** con una fila nueva:
```
| #N+1 | [vertical] | [rol] | [patrón corto] | Aprobado |
```

**Formato de la entrada:**

```markdown
## #[N+1] — [Patrón corto de 3–6 palabras] ([vertical])

**Prospecto**: [Rol] en **[Empresa]** ([descripción de 1 línea del vertical]).

**DM aprobado por el [agente / usuario] ([score]/10)**:

> [cuerpo verbatim — nombre del contacto → [Contacto]]

([N] palabras.)

**✅ Lo que funcionó**:
• [elemento específico]

**❌ Lo que se rechazó / Lo que el usuario corrigió** (si aplica):
• [qué se detectó + razón]

**📝 Lección**: [principio internalizable en 2–4 líneas]
```

**Dónde insertar**: usa `Edit` para agregar la entrada justo **antes** de la línea `## Cómo se agregan entradas nuevas` en `outreach/references/linkedin-corpus.md`.

**Si no hubo rechazos** (el agente aprobó en primera iteración): omite la sección `❌`.

**Confirmar** actualizando la tabla del Paso 3 con `✅ Entrada #N+1` en la columna Corpus.

---

### Paso 3.7 — Oferta de actualizar el Voice Profile (si score ≥ 9.0)

Si el agente aprobó con score ≥ 9.0 en la primera iteración, pregunta al usuario:

```
¿Quieres actualizar el voice profile con el patrón de este DM?
Score: [X.X]/10 en primera iteración — hay un patrón ganador que vale capturar.
Responde "sí" para agregar las frases clave a voice-profile.md.
```

Si el usuario dice sí, agrega en `outreach/references/voice-profile.md`:
- En "Mensajes de referencia de alta puntuación": una fila nueva con el corpus ID y por qué es referencia.
- En "Frases características" si hay frases nuevas no documentadas.
