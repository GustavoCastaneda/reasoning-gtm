---
description: >
  Genera el Email 1 de outreach para un lead usando un loop de refinamiento
  entre tres agentes especializados. Se activa cuando el usuario pide generar
  el outreach o el email para un lead después de que create-contact fue
  ejecutado. Toma la ficha de investigación del contexto de la sesión.
  Foco: un único email, no secuencia. Email 2/3 y LinkedIn se manejan después
  según evolucione la conversación con el prospecto.
---

# Outreach — $ARGUMENTS

Toma la ficha de investigación del lead disponible en el contexto. Si se especifica un nombre de empresa en $ARGUMENTS, usa solo la ficha de ese lead — no proceses otros leads del contexto.

**Foco: solo Email 1.** No generes Email 2, Email 3 ni mensaje de LinkedIn — esos se trabajan después, con contexto fresco según haya o no respuesta del prospecto.

Ejecuta el loop de refinamiento completo antes de mostrar cualquier resultado en el chat. El founder solo ve el email cuando alcanza 8.5 o después de 3 iteraciones.

---

## FLUJO COMPLETO

### Paso 1 — Writer
Invoca al `writer-agent` con:
- La ficha de investigación del lead
- El archivo `outreach/references/outreach-rules.md`

El Writer produce el primer draft con su autocrítica.

---

### Paso 2 — Data Validator (guardrail determinístico)
Invoca al `data-validator-agent` con:
- El draft del Writer
- El archivo `outreach/references/outreach-rules.md`

El Validator verifica que los **datos puntuales obligatorios** estén presentes (operaciones concretas, 80%, semanas/meses → días, industrias) y que NO haya datos fabricados (peer cases inventados, métricas no autorizadas).

```
¿VEREDICTO del Validator?
    PASS → continuar al Paso 3 (Critic)
    FAIL → regresar al Paso 1 con la lista de gaps. El Writer reescribe corrigiendo
           específicamente los datos faltantes. Después del rewrite, vuelve al Paso 2.
           Máximo 2 retries por iteración antes de continuar a Critic con warning.
```

---

### Paso 3 — Critic
Invoca al `critic-agent` con:
- La ficha de investigación del lead
- El archivo `outreach/references/outreach-rules.md`
- El draft del Writer incluyendo su autocrítica
- El reporte del Validator (PASS — para que el Critic confíe en presencia de datos y se enfoque en calidad)

El Critic produce el reporte de revisión con recomendaciones de tono, integración, hook.

---

### Paso 4 — Writer reescribe
Invoca al `writer-agent` nuevamente con:
- La ficha de investigación del lead
- El archivo `outreach/references/outreach-rules.md`
- El draft anterior
- El reporte completo del Critic

El Writer produce el draft revisado incorporando las recomendaciones.

---

### Paso 5 — Data Validator (segundo gate)
Invoca al `data-validator-agent` otra vez con el draft revisado.

```
¿VEREDICTO?
    PASS → continuar al Paso 6 (Judge)
    FAIL → regresar al Paso 4 con la lista de gaps. Máximo 2 retries.
```

Esto evita que el Critic introduzca cambios que rompen la presencia de datos puntuales.

---

### Paso 6 — Judge
Invoca al `judge-agent` con:
- La ficha de investigación del lead
- El archivo `outreach/references/outreach-rules.md`
- El draft revisado del Writer
- El reporte del Critic

El Judge produce la calificación y el veredicto desde la perspectiva del prospecto.

---

### Paso 7 — Decisión del loop

```
¿Calificación ≥ 8.5?
    Sí → ir a Paso 8
    No → ¿iteraciones < 3?
              Sí → regresar al Paso 3 con el feedback del Judge (Critic vuelve a revisar)
              No → ir al Paso 8 con el mejor draft alcanzado
```

El loop máximo es 3 iteraciones (cada iteración = Critic → Writer reescribe → Validator → Judge). Si después de 3 iteraciones no se alcanza 8.5, se entrega el mejor draft con nota de calificación.

---

### Paso 8 — Mostrar en el chat

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

### Paso 9 — Después del visto bueno

Cuando el usuario apruebe, ejecuta sin pedir más confirmación:

**Gmail:**
- Crea **un único draft** con el Email 1: destinatario = work email del contacto, subject + cuerpo aprobados.

**HubSpot:**
- Cambia `hs_lead_status` a `ATTEMPTED_TO_CONTACT`
- Agrega nota: "Email 1 de outreach aprobado — [fecha]. Calificación final: [X.X]/10. Draft en Gmail."

**Confirmar en el chat:**

| Empresa | Draft en Gmail | HubSpot |
|---------|----------------|---------|
| [nombre] | ✅ Email 1 | ✅ ATTEMPTED_TO_CONTACT |