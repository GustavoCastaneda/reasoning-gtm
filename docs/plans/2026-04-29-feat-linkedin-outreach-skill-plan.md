---
title: "feat: /linkedin-outreach — mensajes DM de LinkedIn post-conexión"
type: feat
status: completed
date: 2026-04-29
---

# feat: /linkedin-outreach — mensajes DM de LinkedIn post-conexión

## Contexto

10 conexiones aceptadas en LinkedIn vs 0 respuestas por email. La causa es estructural: como founder, el perfil de LinkedIn ES la credibilidad — el prospecto ya te evaluó antes de leer una sola palabra. El email frío no tiene eso.

**Inversión del orden:** LinkedIn primero (DM post-conexión), email como refuerzo si no hay respuesta.

**Restricción crítica:** todo lo que ya existe (email outreach, agentes writer/critic/judge/data-validator, corpus de email, outreach-rules.md) se mantiene intacto sin ninguna modificación. Este es un skill completamente nuevo que corre en paralelo.

---

## Decisión de arquitectura

### Por qué skill nuevo + agentes propios (no modificar /outreach)

| Alternativa | Problema |
|---|---|
| Modificar `/outreach` para soportar LinkedIn | Rompe calibración de email: word count 75-85, subject line, 4-T framework, corpus de email todo es email-específico |
| Reusar writer/critic/judge con override instructions | Los agentes leen `outreach-rules.md` que tiene reglas de email hardcodeadas; crear conflicto de reglas en el mismo agente es frágil |
| **Skill nuevo + agentes propios** | Cero riesgo al email flow; formato correcto para LinkedIn desde el inicio; los principios (empatía, mecanismo, no jargon) se replican, el formato se adapta |

El setup script auto-detecta cualquier carpeta con `SKILL.md` y cualquier `agents/*.md` — no requiere modificar `setup`.

---

## Qué produce el skill

Un mensaje de LinkedIn DM para copiar y pegar directamente. No crea Gmail draft — el output es texto en el chat. El usuario copia y pega en LinkedIn.

**Contexto del mensaje:** el prospecto ya aceptó la conexión y ha visto el perfil del founder. El DM es el primer mensaje después de la aceptación, no un mensaje frío.

---

## Archivos a crear

| Archivo | Qué hace |
|---|---|
| `linkedin-outreach/SKILL.md` | Orquesta el loop: LinkedIn Writer → LinkedIn Critic → LinkedIn Judge |
| `agents/linkedin-writer-agent.md` | Escribe el DM según reglas de LinkedIn |
| `agents/linkedin-critic-agent.md` | Revisa el DM y da recomendaciones |
| `agents/linkedin-judge-agent.md` | Califica del 1-10 desde perspectiva del prospecto |
| `outreach/references/linkedin-rules.md` | Reglas de formato y tono específicas de LinkedIn DM |
| `outreach/references/linkedin-corpus.md` | Corpus vacío con estructura de índice lista (crece igual que el de email) |

## Archivos a modificar

| Archivo | Qué cambia |
|---|---|
| `agents/gtm-agent.md` | Agregar `/linkedin-outreach` a la tabla de skills y `linkedin-writer/critic/judge-agent` a la tabla de agentes |
| `gtm/SKILL.md` | Agregar frase de routing para "linkedin", "DM de LinkedIn", "mensaje de LinkedIn" |

**No se modifica nada más.** Email outreach flow intacto.

---

## Especificaciones del LinkedIn DM

### Diferencias vs email

| | Email (existente) | LinkedIn DM (nuevo) |
|---|---|---|
| Subject | 1-4 palabras, minúsculas | No aplica |
| Longitud | 75-85 palabras | 80-120 palabras (DM UI es más tolerante, pero el prospecto lee rápido) |
| Primera línea | T1: saludo + reconocimiento de rol | Hook: primera frase visible en preview (~100 chars antes de "Ver más") |
| Referencias al perfil | Solo datos del cargo/trayectoria | Puede mencionar post reciente de LinkedIn — es ESPERADO en este canal |
| Links | Prohibidos | Prohibidos (algoritmo de LinkedIn penaliza) |
| CTA | Value-framed + propósito explícito | Pregunta abierta conversacional — no propuesta de reunión todavía |
| Tono | Profesional B2B | Peer-to-peer en red social |
| Contexto | Desconocido en bandeja | Ya me conoce (aceptó la conexión, vio mi perfil) |

### Framework para el DM (adaptación del 4-T)

- **Hook** (T1): Primera línea enganchadora. Puede referenciar un post reciente o su rol específico. No "gracias por conectar" (genérico). No cumplido ("tu perfil es impresionante"). Algo concreto y relevante.
- **Empatía vertical** (T2): Reto del sector como hipótesis, no diagnóstico. Mismo principio que email — nunca cuestionador, siempre oportunidad.
- **Mecanismo + liberación** (T3): Qué hace Reasoning (explícito) + qué trabajo manual desaparece. Condensado — 1-2 líneas máximo.
- **CTA conversacional** (T4): Pregunta abierta que invita a continuar la conversación. Patrón: "¿aplica a lo que estás construyendo en [empresa]?" No pedir reunión todavía — ese es el siguiente mensaje si responde.

### Longitud

Target 90-110 palabras. El límite no es tan estricto como en email (no hay corte de preview de bandeja), pero LinkedIn DMs se leen en el teléfono y el prospecto tiene el dedo sobre el botón de archivar.

### El hook — regla crítica

LinkedIn muestra ~100 caracteres antes del "Ver más". Si el hook no engancha antes de ese corte, el prospecto no abre. La primera línea debe ser:
- Concreta y específica al prospecto (no genérica)
- Que genere curiosidad o reconocimiento ("eso aplica a lo que yo hago")
- Sin "Hola [nombre], vi que..." como apertura — eso es template y se ve como spam

---

## Loop del skill

```
/linkedin-outreach
  ├── Paso 1 — LinkedIn Writer (genera el DM)
  ├── Paso 2 — LinkedIn Critic (revisa, da recomendaciones)
  ├── [Si critic recomienda reescritura] → Paso 1 otra vez
  └── Paso 3 — LinkedIn Judge (califica 1-10, mínimo 8.5)
      ├── ≥8.5 → entrega el DM al founder para copiar y pegar
      └── <8.5 → nueva iteración (máximo 3 iteraciones totales)
```

**Sin data-validator en MVP.** El judge del LinkedIn loop valida inline (mecanismo presente, no jargon, no fabricación). Se puede agregar después si el corpus crece y aparecen patrones problemáticos recurrentes.

---

## `outreach/references/linkedin-rules.md` — estructura

El archivo debe cubrir:

1. **Contexto del canal** — por qué LinkedIn DM es diferente a email (el prospect ya te conoce, vio tu perfil)
2. **Formato obligatorio** — no subject, longitud 80-120 palabras, hook en primera línea
3. **Hook rules** — qué funciona, qué se siente template
4. **Empatía vertical** — mismos principios que email: hipótesis, no diagnóstico; oportunidad, no problema
5. **Mecanismo** — obligatorio igual que en email: el prospecto debe poder articular qué hace Reasoning
6. **T3 closer** — mismo principio: descripción de qué desaparece, no prescripción de qué hacer después
7. **CTA conversacional** — pregunta, no propuesta; no pedir reunión todavía
8. **Prohibiciones** — "gracias por conectar", cumplidos genéricos, links, jerga arquitectónica, workflow inventado, prescripción de refocus
9. **Palabras prohibidas** — replicar la lista del email (solution, potenciar, empoderar, sinergia, transformar, etc.)
10. **Calibración de longitud del hook** — primera línea ≤100 chars para caber en el preview

---

## `outreach/references/linkedin-corpus.md` — estructura inicial

Inicia vacío con el mismo formato que el corpus de email: índice arriba, entradas numeradas abajo. Los agentes de LinkedIn leen el índice primero, saltan a entradas por ID.

```markdown
# LinkedIn DM Corpus — Reasoning Labs

## Índice
| ID | Vertical | Rol | Patrón | Estado |
|----|----------|-----|--------|--------|
(vacío — se llena con la primera iteración aprobada)

## Cómo usar este corpus
[mismo patrón que email-corpus.md]
```

---

## Actualización de routing tables (OBLIGATORIO — regla de CLAUDE.md)

### `agents/gtm-agent.md`

**Tabla de skills** — agregar fila:
```
| `/linkedin-outreach` | Después de que el prospecto aceptó la conexión en LinkedIn | Nombre de empresa (toma la ficha del contexto) |
```

**Tabla de agentes** — agregar filas:
```
| `linkedin-writer-agent` | Escribe el DM de LinkedIn post-conexión | El skill `linkedin-outreach` automáticamente |
| `linkedin-critic-agent` | Revisa el DM y da recomendaciones | El skill `linkedin-outreach` automáticamente |
| `linkedin-judge-agent` | Califica del 1-10 como si fuera el prospecto en LinkedIn | El skill `linkedin-outreach` automáticamente |
```

### `gtm/SKILL.md`

**Tabla de routing (Paso 1A)** — agregar fila:
```
| "linkedin", "DM de linkedin", "mensaje de linkedin", "outreach por linkedin" | `/linkedin-outreach` (stop) |
```

---

## Acceptance Criteria

- [x] `linkedin-outreach/SKILL.md` existe y orquesta el loop writer→critic→judge
- [x] `agents/linkedin-writer-agent.md` produce DMs de 80-120 palabras sin subject, con hook en primera línea
- [x] `agents/linkedin-critic-agent.md` evalúa hook, empatía, mecanismo, CTA y tono peer-to-peer
- [x] `agents/linkedin-judge-agent.md` califica desde perspectiva de prospecto en LinkedIn, min 8.5
- [x] `outreach/references/linkedin-rules.md` existe con reglas específicas de LinkedIn DM
- [x] `outreach/references/linkedin-corpus.md` existe con estructura de índice vacía
- [x] `agents/gtm-agent.md` tiene `/linkedin-outreach` en tabla de skills y los 3 agentes en tabla de agentes
- [x] `gtm/SKILL.md` tiene routing para frases de LinkedIn en Paso 1A
- [x] `outreach/SKILL.md` y agentes de email NO se modificaron
- [x] El output del skill es texto en el chat (copy-paste), no un Gmail draft
- [x] `./setup` instala el nuevo skill y los 3 agentes automáticamente (sin modificar setup)

---

## Dependencias

- Requiere que `/research` haya corrido (la ficha del lead debe existir en contexto o en `leads/[slug]/ficha.md`)
- No requiere que `/create-company` ni `/create-contact` hayan corrido antes
- No requiere integración con LinkedIn API

---

## Sources

- `outreach/SKILL.md` — estructura del loop a replicar/adaptar
- `agents/writer-agent.md` — estructura del agente de email a adaptar para LinkedIn
- `agents/gtm-agent.md:142-165` — tabla de skills y agentes a actualizar
- `gtm/SKILL.md:30-45` — tabla de routing a actualizar
- `outreach/references/outreach-rules.md` — principios a reutilizar (tono, mecanismo, no jargon)
- `outreach/references/email-corpus.md` — estructura de corpus a replicar para LinkedIn
- `setup` — wildcard copy confirma que no requiere modificación
