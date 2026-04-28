---
description: >
  Genera la secuencia de outreach para un lead usando un loop de refinamiento
  entre tres agentes especializados. Se activa cuando el usuario pide generar
  el outreach o los emails para un lead después de que create-contact fue
  ejecutado. Toma la ficha de investigación del contexto de la sesión.
---

# Outreach — $ARGUMENTS

Toma la ficha de investigación del lead disponible en el contexto. Si se especifica un nombre de empresa en $ARGUMENTS, usa solo la ficha de ese lead — no proceses otros leads del contexto.

Ejecuta el loop de refinamiento completo antes de mostrar cualquier resultado en el chat. El founder solo ve el email cuando alcanza 8.5 o después de 3 iteraciones.

---

## FLUJO COMPLETO

### Paso 1 — Writer
Invoca al `writer-agent` con:
- La ficha de investigación del lead
- El archivo `outreach/references/outreach-rules.md`

El Writer produce el primer draft con su autocrítica.

---

### Paso 2 — Critic
Invoca al `critic-agent` con:
- La ficha de investigación del lead
- El archivo `outreach/references/outreach-rules.md`
- El draft del Writer incluyendo su autocrítica

El Critic produce el reporte de revisión con recomendaciones.

---

### Paso 3 — Writer reescribe
Invoca al `writer-agent` nuevamente con:
- La ficha de investigación del lead
- El archivo `outreach/references/outreach-rules.md`
- El draft anterior
- El reporte completo del Critic

El Writer produce el draft revisado incorporando las recomendaciones.

---

### Paso 4 — Judge
Invoca al `judge-agent` con:
- La ficha de investigación del lead
- El archivo `outreach/references/outreach-rules.md`
- El draft revisado del Writer
- El reporte del Critic

El Judge produce la calificación y el veredicto.

---

### Paso 5 — Decisión del loop

```
¿Calificación ≥ 8.5?
    Sí → ir a Paso 6
    No → ¿iteraciones < 3?
              Sí → regresar al Paso 2 con el feedback del Judge
              No → ir al Paso 6 con el mejor draft alcanzado
```

El loop máximo es 3 iteraciones. Si después de 3 iteraciones no se alcanza 8.5, se entrega el mejor draft con nota de calificación.

---

### Paso 6 — Mostrar en el chat

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

─────────────────────────────────────────

📧 EMAIL 2 — Día 3 (reply al hilo)
Subject: Re: [subject]

[cuerpo]

─────────────────────────────────────────

📧 EMAIL 3 — Break-up Día 14 (reply al hilo)
Subject: Re: [subject]

[cuerpo]

─────────────────────────────────────────

💼 LINKEDIN — Mensaje (después de aceptar conexión)

[cuerpo]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
¿Apruebas esta secuencia? Responde "sí" para crear los drafts en Gmail.
```

---

### Paso 7 — Después del visto bueno

Cuando el usuario apruebe, ejecuta sin pedir más confirmación:

**Gmail:**
- Crea Email 1 como draft con destinatario: work email del contacto
- Crea Email 2 como draft con subject "Re: [subject]"
- Crea Email 3 como draft con subject "Re: [subject]"

**HubSpot:**
- Cambia `hs_lead_status` a `ATTEMPTED_TO_CONTACT`
- Agrega nota: "Secuencia de outreach aprobada — [fecha]. Calificación final: [X.X]/10. Drafts en Gmail."

**Confirmar en el chat:**

| Empresa | Drafts en Gmail | HubSpot |
|---------|----------------|---------|
| [nombre] | ✅ 3 emails | ✅ ATTEMPTED_TO_CONTACT |