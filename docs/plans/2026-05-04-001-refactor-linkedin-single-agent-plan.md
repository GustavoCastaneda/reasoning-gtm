---
title: "refactor: LinkedIn outreach — agente único con self-review (eliminar loop Writer→Critic→Judge)"
type: refactor
status: active
date: 2026-05-04
---

# refactor: LinkedIn outreach — agente único con self-review

## Summary

El loop actual Writer → [Critic + Judge paralelos] → reescritura × 3 genera 6–9 invocaciones de agente para producir un DM de 60–120 palabras. Este refactor lo reemplaza con un único `linkedin-dm-agent` que escribe, se auto-evalúa contra los criterios del Critic y el Judge, se reescribe una vez si no llega a 8.5, y entrega el DM con calificación. Total: 1–2 llamadas fijas en lugar de hasta 9.

---

## Problem Frame

El corpus acumulado (12 entradas), el voice profile con correcciones acumuladas, y las reglas en `linkedin-rules.md` ya codifican lo que el Critic y el Judge harían. Cada iteración del loop externa agrega latencia, costo de tokens, y overhead de contexto sin mejorar dramáticamente un mensaje de 8–10 líneas. El bottleneck no es la falta de perspectivas externas — es que las reglas no están suficientemente internalizadas en quien escribe.

---

## Requirements

- R1. El flujo de LinkedIn DM debe completarse en máximo 2 invocaciones de agente (escribe → reescribe si aplica).
- R2. El agente único debe incorporar los criterios de evaluación del Critic (8 ejes) y las reglas especiales del Judge (4 reglas de techo) como instrucciones internas.
- R3. El agente debe auto-scorear el DM antes de entregarlo. Si score < 8.5: reescribe una vez internamente.
- R4. El output al founder debe incluir el DM + score + T2/T3 elegidos (reemplaza el bloque de proceso del loop anterior).
- R5. El skill `/linkedin-outreach` debe simplificarse: Paso 0 (modo) → Paso 1 (invocar agente) → Paso 2 (aprobación) → Paso 3 (HubSpot + corpus).
- R6. Los tres agentes anteriores (writer, critic, judge) deben archivarse — no eliminarse — para referencia futura.
- R7. El `setup` script debe instalar el nuevo agente y desinstalar los tres anteriores.

---

## Scope Boundaries

- Este refactor aplica únicamente al flujo de LinkedIn DM — no toca `/outreach` (email), ni sus agentes (writer-agent, critic-agent, judge-agent).
- No cambia los archivos de referencia: `linkedin-rules.md`, `voice-profile.md`, `linkedin-corpus.md`.
- No cambia el Paso 5.5 (corpus automático) ni el Paso 5.7 (oferta de actualizar voice profile) del skill.
- El score mínimo de aprobación (8.5) no cambia — solo desaparece el loop externo.
- No se agrega lógica de follow-up ni secuencias — sigue siendo solo Día 1.

### Deferred to Follow-Up Work

- Aplicar el mismo patrón de agente único al flujo de Email 1 (outreach): potencialmente colapsar writer-agent + critic-agent + judge-agent de email en un único agente con self-review. Eso es una decisión separada — el email tiene un data-validator determinístico que agrega complejidad distinta.

---

## Context & Research

### Relevant Code and Patterns

- `agents/linkedin-writer-agent.md` — instrucciones de escritura actuales: paso 0 (calibración con voice profile + corpus), instrucciones antes de escribir (#1–13), checklist de auto-revisión, formato de output con autocrítica.
- `agents/linkedin-critic-agent.md` — 8 ejes de evaluación: apertura/trigger, uso de ficha, adherencia a reglas, impacto, claridad del mecanismo, tono empático vs presuntuoso, T3 prescriptivo, riesgos.
- `agents/linkedin-judge-agent.md` — 5 tests de perspectiva del prospecto + 4 reglas especiales de techo (trigger genérico → máx 6.0, claridad de oferta → máx 7.5, respeto al peer → máx 7.0, CTA sin escape hatch → máx 7.5).
- `linkedin-outreach/SKILL.md` — orquestador actual: Paso 0 (modo) → Paso 1 (Writer) → Paso 2 (Critic + Judge paralelos) → Paso 3 (decisión loop) → Paso 4 (mostrar) → Paso 5 (aprobación + HubSpot) → Paso 5.5 (corpus) → Paso 5.7 (voice profile).
- `outreach/references/voice-profile.md`, `outreach/references/linkedin-rules.md`, `outreach/references/linkedin-corpus.md` — fuentes de verdad sin cambios.
- `setup` — instala agentes en `~/.claude/agents/` y reescribe paths a absolutos con `sed`.

### Institutional Learnings

- El corpus de 12 entradas y el voice profile con correcciones acumuladas ya codifican los patrones que el Critic y el Judge detectarían. La función real del loop era corregir al Writer cuando sus instrucciones eran insuficientes — ahora que las instrucciones son más completas y el corpus es más rico, la corrección puede ocurrir dentro del mismo agente.
- La enumeración técnica (limpieza, deduplicación, homologación) sin T2 previo y el T2 forzado eran los errores más frecuentes. Están explícitamente documentados en las instrucciones del writer tras el refactor anterior — el agente único heredará esas correcciones.

---

## Key Technical Decisions

- **Merge completo (Writer + Critic + Judge)**: en lugar de tener un writer que escribe y un evaluador separado, el agente único internaliza las 8 secciones del Critic y las 4 reglas especiales del Judge como criterios de auto-evaluación. La independencia del Judge (evaluar sin ver la construcción) se sacrifica a cambio de eficiencia — la compensación es aceptable porque las reglas ya son explícitas y el corpus provee calibración.

- **1 reescritura interna máximo**: si el score inicial < 8.5, el agente reescribe una vez con su propio feedback como contexto. No hay segundo Critic ni segundo Judge externo. Si después de la reescritura sigue < 8.5, entrega con nota — el founder decide. Esto evita el loop infinito manteniendo un safety net.

- **Score obligatorio en output**: el agente siempre reporta su auto-score. Esto permite al founder calibrar el nivel de confianza y es la señal equivalente a la calificación del Judge anterior.

- **Archivado, no eliminación**: los tres agentes anteriores se mueven a `agents/archive/`. Son referencia histórica y pueden consultarse si hay regresión o si se aplica el mismo patrón al email.

- **Setup script**: el script actual usa `cp` + `sed` para instalar agentes. El nuevo agente se instala igual; los tres archivados se desinstalan eliminando sus copias en `~/.claude/agents/`.

---

## High-Level Technical Design

> *Ilustra el approach propuesto — guía de dirección para revisión, no especificación de implementación.*

```
FLUJO ACTUAL (6–9 invocaciones):
───────────────────────────────
Skill detecta modo
  → linkedin-writer-agent (lee 3 archivos, escribe draft)
  → [linkedin-critic-agent ∥ linkedin-judge-agent] (leen 2 archivos c/u)
  → Si score < 8.5 y iter < 3:
      → linkedin-writer-agent (reescribe con reportes)
      → [linkedin-critic-agent ∥ linkedin-judge-agent]
      → ... (hasta 3 veces)
  → Mostrar al founder

FLUJO NUEVO (1–2 invocaciones):
────────────────────────────────
Skill detecta modo
  → linkedin-dm-agent
      Fase A: lee voice-profile + linkedin-rules + corpus (index + 2-3 entradas)
      Fase B: escribe DM según instrucciones del writer
      Fase C: auto-evalúa contra los 8 ejes del Critic + las 4 reglas del Judge
      Fase D: si score < 8.5 → reescribe una vez con self-feedback
      Fase E: entrega DM + score + metadatos de T2/T3
  → Mostrar al founder
  → Si aprueba → HubSpot + corpus (sin cambios)
```

**Estructura del output del agente único:**

```
[LINKEDIN DM — Iteración 1 / Iteración 2 tras reescritura]

DM POST-CONEXIÓN — [COMPLETO / POST-RESPUESTA]

[cuerpo del DM]

─────────────────────────────────────
SELF-EVALUATION
• Modo: [COMPLETO / POST-RESPUESTA]
• T2 elegido: [OMITIDO+"nos toca directo" / 1 oración pain / pain+80%]
• T3 variante: [SIMPLE / CON-ENUMERACIÓN]
• Trigger específico: [sí — "cita"] / [no — reescribir]
• Mecanismo declarado: [sí — "cita"] / [no — reescribir]
• CTA con tiempo+escape hatch: [sí] / [no — penalización aplicada]
• Palabras: [N]
• Score: [X.X] / 10
• [Si se reescribió: "Reescribí porque: [razón en 1 línea]"]
• [Si score < 8.5 tras reescritura: "⚠️ Debajo del umbral — revisar antes de enviar"]
```

---

## Implementation Units

- U1. **Crear `agents/linkedin-dm-agent.md` — agente único con write + self-critique + self-score**

**Goal:** Crear el agente que reemplaza writer + critic + judge para el flujo de LinkedIn DM.

**Requirements:** R1, R2, R3, R4

**Dependencies:** Ninguna — puede crearse independientemente antes de modificar el skill.

**Files:**
- Create: `agents/linkedin-dm-agent.md`

**Approach:**
- El agente lee: `outreach/references/voice-profile.md` + `outreach/references/linkedin-rules.md` + índice + entradas relevantes del corpus (mismo patrón que el writer actual).
- Sección "Escribir el DM": incorpora las instrucciones #1–13 del writer actual + el checklist de preguntas finales del writer.
- Sección "Auto-evaluar": los 8 ejes del Critic codificados como checklist interno. Los 4 tests del Judge como preguntas de perspectiva del prospecto. Las 4 reglas especiales de techo (trigger genérico → máx 6.0, etc.) como reglas de scoring.
- Sección "Decisión y reescritura": si score ≥ 8.5 → va directo al output. Si score < 8.5 → genera nuevo draft con el self-feedback como contexto adicional, luego va al output.
- Output: DM + bloque SELF-EVALUATION con score, T2/T3, y metadatos.
- El agente recibe en su invocación: ficha + modo (COMPLETO o POST-RESPUESTA). Si modo = POST-RESPUESTA: omite T0 en el DM.

**Patterns to follow:**
- `agents/linkedin-writer-agent.md` — estructura del paso 0 (calibración con voice profile + corpus), instrucciones antes de escribir, formato de output.
- `agents/linkedin-critic-agent.md` — los 8 ejes de evaluación (copiar como checklist interno).
- `agents/linkedin-judge-agent.md` — los 4 tests + 4 reglas especiales (copiar como scoring rubric interno).

**Test scenarios:**
- Test expectation: none — este es un archivo de instrucciones para agente, no código ejecutable. La verificación es cualitativa: invocar `/linkedin-outreach` con una ficha de prueba y confirmar que el output incluye DM + SELF-EVALUATION con todos los campos.

**Verification:**
- El archivo existe en `agents/linkedin-dm-agent.md`.
- Contiene secciones: "Escribir el DM", "Auto-evaluar" (con 8 ejes + 4 tests + 4 reglas), "Decisión y reescritura", "Output".
- El bloque SELF-EVALUATION en el output incluye: Modo, T2 elegido, T3 variante, Trigger específico, Mecanismo declarado, CTA con escape hatch, Palabras, Score.

---

- U2. **Simplificar `linkedin-outreach/SKILL.md` — eliminar loop, invocar agente único**

**Goal:** Reescribir el skill para que use el agente único. Pasos: 0 (modo) → 1 (agente) → 2 (mostrar) → 3 (aprobación + HubSpot) → 3.5 (corpus) → 3.7 (voice profile si ≥ 9.0).

**Requirements:** R5

**Dependencies:** U1 (el agente debe estar definido antes de que el skill lo referencie).

**Files:**
- Modify: `linkedin-outreach/SKILL.md`

**Approach:**
- Eliminar los pasos actuales 1 (Writer), 2 (Critic+Judge), 3 (decisión del loop), 4 (mostrar) — 4 pasos colapsan en 2.
- Nuevo Paso 1: invocar `linkedin-dm-agent` con ficha + modo. El agente entrega DM + score.
- Nuevo Paso 2 (antes Paso 4): mostrar al founder el DM con su score y formato. Pedir aprobación.
- Los pasos de aprobación (HubSpot, corpus, voice profile) se renumeran pero no cambian en lógica.
- El bloque de "PROCESO DE REFINAMIENTO" en el output al founder se simplifica: ya no muestra "Iteración 1: X.X, Iteración 2: X.X" — muestra "Score: X.X / 10" y si hubo reescritura interna, "Reescribí porque: [razón]".

**Patterns to follow:**
- `linkedin-outreach/SKILL.md` actual — mantener los pasos de aprobación (HubSpot, corpus, voice profile) sin cambios.
- `outreach/SKILL.md` — referencia de cómo un skill invoca un único agente y espera su output.

**Test scenarios:**
- Test expectation: none — es un archivo de instrucciones de skill, no código. Verificación cualitativa con invocación real.

**Verification:**
- El skill tiene máximo 4 pasos numerados (0, 1, 2, 3/3.5/3.7).
- No hay referencias a `linkedin-writer-agent`, `linkedin-critic-agent`, ni `linkedin-judge-agent`.
- Hay referencia explícita a `linkedin-dm-agent`.
- El paso de HubSpot y corpus está intacto.

---

- U3. **Archivar los tres agentes anteriores y actualizar `setup`**

**Goal:** Mover writer + critic + judge de LinkedIn a `agents/archive/`, desinstalar sus copias en `~/.claude/agents/`, e instalar `linkedin-dm-agent`.

**Requirements:** R6, R7

**Dependencies:** U1 (el nuevo agente debe existir antes de que setup lo instale).

**Files:**
- Create: `agents/archive/` (directorio si no existe)
- Create: `agents/archive/linkedin-writer-agent.md`
- Create: `agents/archive/linkedin-critic-agent.md`
- Create: `agents/archive/linkedin-judge-agent.md`
- Modify: `setup` (actualizar listas de agentes a instalar/desinstalar)

**Approach:**
- Copiar los 3 archivos a `agents/archive/` antes de cualquier otro cambio (preservar historial).
- En el script `setup`: agregar `linkedin-dm-agent.md` a la lista de agentes a instalar. Agregar lógica para eliminar `~/.claude/agents/linkedin-writer-agent.md`, `~/.claude/agents/linkedin-critic-agent.md`, `~/.claude/agents/linkedin-judge-agent.md` si existen (limpieza de instalaciones previas).
- Eliminar `agents/linkedin-writer-agent.md`, `agents/linkedin-critic-agent.md`, `agents/linkedin-judge-agent.md` del directorio principal (ya están archivados).
- La directiva `git rm` para los archivos eliminados debe ejecutarse explícitamente — no dejar archivos zombies en el repo.

**Patterns to follow:**
- `setup` actual — ver cómo instala agentes con `cp` + `sed` para reescritura de paths a absolutos.

**Test scenarios:**
- Test expectation: none — verificación operacional: después de correr `./setup`, `ls ~/.claude/agents/linkedin-*` debe mostrar solo `linkedin-dm-agent.md`.

**Verification:**
- `agents/archive/linkedin-writer-agent.md` existe.
- `agents/archive/linkedin-critic-agent.md` existe.
- `agents/archive/linkedin-judge-agent.md` existe.
- `agents/linkedin-writer-agent.md` NO existe en el directorio principal.
- `agents/linkedin-critic-agent.md` NO existe en el directorio principal.
- `agents/linkedin-judge-agent.md` NO existe en el directorio principal.
- `~/.claude/agents/linkedin-dm-agent.md` existe después de correr `./setup`.
- `~/.claude/agents/linkedin-writer-agent.md` NO existe después de correr `./setup`.

---

## System-Wide Impact

- **Interaction graph:** Solo afecta el flujo `/linkedin-outreach`. El skill `/gtm` cuando rutea a linkedin-outreach no cambia — el resultado visible al usuario es el mismo (DM para copiar-pegar). Los agentes de email (`writer-agent`, `critic-agent`, `judge-agent`) no se tocan.
- **Error propagation:** Si el agente único falla o no llega a 8.5 tras la reescritura, el skill entrega lo que tiene con nota de advertencia `⚠️` — misma lógica de fallback que el loop anterior.
- **State lifecycle risks:** Ninguno — no hay estado persistente entre los pasos del agente único. El corpus y HubSpot se actualizan solo después de la aprobación del founder, igual que antes.
- **API surface parity:** El output visible al founder (DM + score) es equivalente al actual. El bloque "PROCESO DE REFINAMIENTO" se simplifica — si el founder dependía de ver "Iteración 1: X.X / Iteración 2: X.X", ahora solo ve el score final y si hubo reescritura interna.
- **Unchanged invariants:** El umbral de 8.5 no cambia. El corpus automático (Paso 5.5) no cambia. La actualización de HubSpot (Paso 5/3) no cambia. El modo POST-RESPUESTA (Paso 0) no cambia.

---

## Risks & Dependencies

| Riesgo | Mitigación |
|--------|------------|
| El agente único puede tener un sistema prompt muy largo (merge de 3 agentes) y los LLMs pueden omitir criterios cuando hay demasiadas instrucciones | Organizar el prompt en secciones claras y secuenciales (escribir → evaluar → decidir). El orden explícito reduce omisiones más que la longitud total. |
| La auto-evaluación es menos independiente que la perspectiva del Judge externo — el agente puede "validar" lo que acaba de escribir | Las 4 reglas especiales de techo (trigger genérico, claridad, respeto al peer, CTA) son mecánicas y no dependen de perspectiva — se aplican como checklist binario antes del scoring subjetivo. |
| Si el agente se reescribe solo, puede introducir regresiones en aspectos que ya estaban bien | La reescritura interna tiene acceso a todo el contexto (draft original + self-critique) — el agente puede ser instructed a preservar lo que ya funcionaba y corregir solo lo que identificó. |
| Tres agentes en `~/.claude/agents/` desaparecen — si hay otras sesiones abiertas que los referencien, pueden romper | El archivo `linkedin-outreach/SKILL.md` actualizado elimina las referencias. El setup limpia la instalación global. Riesgo mínimo dado que estos agentes solo se invocan desde ese skill. |

---

## Sources & References

- `agents/linkedin-writer-agent.md` — instrucciones actuales del writer
- `agents/linkedin-critic-agent.md` — 8 ejes de evaluación a internalizar
- `agents/linkedin-judge-agent.md` — rubric de scoring y 4 reglas especiales a internalizar
- `linkedin-outreach/SKILL.md` — skill actual a simplificar
- `setup` — script de instalación de agentes
