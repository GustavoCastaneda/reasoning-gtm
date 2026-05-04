---
description: >
  Genera el Email 1 de outreach para un lead usando un loop de refinamiento
  entre tres agentes especializados. Se activa cuando el usuario pide generar
  el outreach o el email para un lead después de que create-contact fue
  ejecutado. Toma la ficha de investigación del contexto de la sesión.
  Foco: un único email, no secuencia. Email 2/3 y LinkedIn se manejan después
  según evolucione la conversación con el prospecto.
allowed-tools:
  - Skill
  - Read
  - Edit
---

# Outreach — $ARGUMENTS

Toma la ficha de investigación del lead disponible en el contexto. Si se especifica un nombre de empresa en $ARGUMENTS, usa solo la ficha de ese lead — no proceses otros leads del contexto.

Si la ficha no está en el contexto de la sesión, busca el archivo local: `leads/[empresa-slug]/ficha.md` (slug = nombre de la empresa en minúsculas, espacios → guiones, sin acentos). Si el archivo tampoco existe, detente: "No encuentro la ficha de [empresa]. Corre primero `/research` o pega la ficha en el chat."

**Foco: solo Email 1.** No generes Email 2, Email 3 ni mensaje de LinkedIn — esos se trabajan después, con contexto fresco según haya o no respuesta del prospecto.

Ejecuta el loop de refinamiento completo antes de mostrar cualquier resultado en el chat. El founder solo ve el email cuando alcanza 8.5 o después de 3 iteraciones.

---

## FLUJO COMPLETO

### Paso 1 — Writer
Invoca al `writer-agent` con:
- La ficha de investigación del lead
- El archivo `outreach/references/outreach-rules.md`
- El archivo `outreach/references/voice-profile.md`

El Writer produce el primer draft con su autocrítica.

---

### Paso 2 — Data Validator (guardrail determinístico)
Invoca al `data-validator-agent` con:
- El draft del Writer
- El archivo `outreach/references/outreach-rules.md`

El Validator verifica que los **datos puntuales obligatorios** estén presentes (operaciones concretas, 80%, semanas/meses → días, industrias) y que NO haya datos fabricados (peer cases inventados, métricas no autorizadas).

```
¿VEREDICTO del Validator?
    PASS → continuar al Paso 3 (Critic + Judge en paralelo)
    FAIL — ¿el único chequeo fallido es el #12 (longitud)?
        SÍ (solo falló longitud) → Gate de longitud:
              Mostrar al founder:
              "⚠️ El email tiene [N] palabras (límite: 85). Todos los demás chequeos pasaron.
               A) Recortamos a ≤85 palabras — el Writer lo compacta
               B) Autorizas [N] palabras — el contenido operacional justifica la excepción"
              Si elige A → regresar al Paso 1 con gap de longitud. Máximo 2 retries.
              Si elige B → agregar [OVERRIDE-LENGTH: autorizado] al contexto y re-invocar
                           el Validator (ahora pasa Chequeo 12 como PASS override) →
                           continuar al Paso 3 (Critic + Judge en paralelo)
        NO (hay otros FAILs además de longitud) → regresar al Paso 1 con la lista de gaps.
           El Writer reescribe corrigiendo los datos faltantes. Después del rewrite, vuelve al Paso 2.
           Máximo 2 retries por iteración antes de continuar con warning.
```

---

### Paso 3 — Critic + Judge en paralelo

Invoca **simultáneamente** al `critic-agent` y al `judge-agent`. Ambos reciben:
- La ficha de investigación del lead
- El archivo `outreach/references/outreach-rules.md`
- El draft del Writer (con su autocrítica)
- El reporte del Validator (PASS — para que Critic confíe en los datos y se enfoque en calidad)

El Critic produce recomendaciones de tono, integración y hook.
El Judge produce calificación 1–10 desde la perspectiva del prospecto.

**Nota:** el Judge evalúa el draft como el prospecto lo leería — sin ver el reporte del Critic. El Writer sí recibe ambos reportes en la siguiente iteración.

---

### Paso 4 — Decisión del loop

```
¿Calificación del Judge ≥ 8.5?
    Sí → ir a Paso 5 (mostrar resultado)
    No → ¿iteraciones completadas < 3?
              Sí → Writer reescribe con AMBOS reportes (Critic + Judge) →
                   Validator nuevamente (Paso 2) → Critic+Judge en paralelo (Paso 3)
              No → ir a Paso 5 con el mejor draft alcanzado
```

El loop máximo es 3 iteraciones (cada iteración = Writer → Validator → [Critic+Judge simultáneos]). Si después de 3 iteraciones no se alcanza 8.5, se entrega el mejor draft con nota de calificación.

**Al reescribir**, invoca al `writer-agent` con:
- La ficha de investigación del lead
- El archivo `outreach/references/outreach-rules.md`
- El archivo `outreach/references/voice-profile.md`
- El draft anterior
- El reporte completo del Critic
- El reporte completo del Judge (incluye calificación y mejoras específicas)

Después del rewrite, el Validator debe aprobar el nuevo draft antes de pasar a Critic+Judge nuevamente. Si el Validator falla (fuera del gate de longitud): 2 retries máximo.

---

### Paso 5 — Mostrar en el chat

Muestra el progreso y el resultado final:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
OUTREACH: [Empresa] — [Contacto] | [Puesto]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PROCESO DE REFINAMIENTO
• Iteración 1: [calificación del Judge]
• Iteración 2: [calificación del Judge] (si aplica)
• Iteración 3: [calificación del Judge] (si aplica)
• Calificación final: [X.X] / 10

─────────────────────────────────────────

📧 EMAIL 1 — Día 1
Subject: [subject]

[cuerpo]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
¿Apruebas este email? Responde "sí" para crear el draft en Gmail.
```

---

### Paso 6 — Después del visto bueno

Cuando el usuario apruebe, ejecuta sin pedir más confirmación:

**Gmail:**
- Crea **un único draft** con el Email 1: destinatario = work email del contacto, subject + cuerpo aprobados.

**HubSpot:**
- Cambia `hs_lead_status` a `ATTEMPTED_TO_CONTACT`
- Agrega nota: "Email 1 de outreach aprobado — [fecha]. Calificación final: [X.X]/10. Draft en Gmail."

**Confirmar en el chat:**

| Empresa | Draft en Gmail | HubSpot | Corpus |
|---------|----------------|---------|--------|
| [nombre] | ✅ Email 1 | ✅ ATTEMPTED_TO_CONTACT | ✅ Entrada #N |

---

### Paso 6.5 — Aprendizaje automático: agregar entrada al corpus

Sin pedir confirmación, genera y agrega una entrada al corpus de outreach inmediatamente después de confirmar la tabla del Paso 9.

**Qué extraer de la sesión:**

- **Email aprobado**: el que el usuario dijo "sí" o "aprobado" — puede diferir del draft que el Judge calificó si el usuario hizo correcciones post-judge. Usa siempre la versión final que el usuario aceptó.
- **Empresa**: preservar sin anonimizar (el grounding del vertical lo requiere).
- **Contacto**: reemplazar el nombre real del contacto con `[Contacto]` en el cuerpo del email.
- **Calificación**: la del Judge sobre el draft más cercano al aprobado.
- **Principal aprendizaje**: qué cambió del primer draft al aprobado. Puede ser: corrección del Critic, mejora del Judge, o corrección post-judge del usuario. Si hubo varias iteraciones, captura la corrección más importante.
- **Si el usuario corrigió algo post-judge**: documenta el término o frase específica corregida + la razón que dio el usuario.

**Leer el corpus para obtener el número siguiente:**

Lee `outreach/references/email-corpus.md` y encuentra la última entrada `## #N —`. La nueva entrada es `#N+1`.

**Formato de la entrada a agregar:**

```markdown
## #[N+1] — [Patrón corto de 3–6 palabras] ([vertical])

**Prospecto**: [Rol] en **[Empresa]** ([descripción de 1 línea del vertical]).

**Email aprobado por el [Judge / usuario] ([score]/10)**:

> Subject: [subject]
>
> [cuerpo verbatim — nombre del contacto → [Contacto]]

([N] palabras[si excedió 85: " — el usuario autorizó exceder el límite para [razón específica]"].)

**✅ Lo que funcionó**:
• [elemento específico que funcionó — T1 limpio, T2 con riqueza operacional, etc.]

**❌ Lo que se rechazó / Lo que el usuario corrigió**:
• [qué se detectó + cita del feedback del Critic/Judge/usuario si está en el contexto]

**📝 Lección**: [principio internalizable en 2–4 líneas — qué debe saber el writer/critic en la próxima sesión para este patrón]
```

**Dónde insertar**: usa `Edit` para agregar la entrada justo **antes** de la línea `## Cómo se agregan entradas nuevas` al final del corpus.

**Si no hubo rechazos en el loop** (el Judge aprobó en primera iteración): omite la sección `❌` y documenta solo `✅` + lección.

**Confirmar en el chat** actualizando la tabla del Paso 9 con `✅ Entrada #N+1` en la columna Corpus.