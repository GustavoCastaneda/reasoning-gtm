---
title: "feat: Voice Profile (cerebro del founder) + aceleración del loop de outreach"
type: feat
status: completed
date: 2026-04-30
---

# feat: Voice Profile + aceleración del loop de outreach

## Overview

Dos problemas hoy: el flujo de 2 leads tarda ~20 minutos, y el Writer produce drafts genéricos que necesitan 2–3 iteraciones de Critic+Judge para llegar a 8.5. Ambos problemas tienen la misma solución raíz: el Writer no tiene suficiente contexto del founder para producir un mensaje de calidad a la primera.

Este plan resuelve ambos con tres cambios coordinados:

1. **Voice Profile** (`voice-profile.md`) — el "cerebro" del founder: frases características, patrones validados, correcciones acumuladas. Los Writers lo leen antes del corpus. Reduce iteraciones promedio de ~2.5 a ~1.2.
2. **Critic + Judge en paralelo** — en lugar de secuencial (Critic → Writer → Judge), se evalúan simultáneamente. Ahorra 1–2 min por iteración.
3. **Research paralelo para N>1 leads** en `/gtm` — investigación de todos los leads al mismo tiempo. Para 2 leads: de ~20 min secuenciales a ~10–12 min con paralelismo.

---

## Problem Statement

### Cuello de botella 1: Leads procesados en secuencia

El `/gtm` con 2 leads hace: Lead1 completo → Lead2 completo. Cada lead son 8–12 min de investigación + loop de outreach. Total: 16–24 min.

Si investigación de ambos leads corriera en paralelo (no dependen entre sí), se reduciría a ~10–12 min.

### Cuello de botella 2: Lecturas de archivos redundantes

Por cada iteración de outreach, cada agente lee independientemente `linkedin-rules.md` + `linkedin-corpus.md` (o `outreach-rules.md` + `email-corpus.md`). En 3 agentes × 2 archivos × 2 iteraciones = 12 lecturas de los mismos archivos. Cada lectura consume tiempo de tool call.

### Cuello de botella 3: Writer sin calibración del founder → múltiples iteraciones

El Writer lee reglas genéricas del canal y el corpus. Infiere el estilo del founder de los ejemplos. El resultado: primer draft mediocre que necesita 2–3 rondas de Critic+Judge. Cada ronda adicional son 2–4 min.

Si el Writer tiene acceso a un perfil explícito de la voz del founder (frases exactas, estructura preferida, correcciones acumuladas), el primer draft debería llegar a 8.5 directamente con más frecuencia.

---

## Proposed Solution

### Componente 1: `voice-profile.md` — el cerebro del founder

Archivo en `outreach/references/voice-profile.md`, disponible en `~/.claude/references/voice-profile.md` vía el symlink que ya crea setup.

Contiene:
- **Identidad del remitente**: nombre preferido en el mensaje, cargo, URL del producto
- **Frases características** (verbatim): las frases que Gustavo usa naturalmente y que han funcionado
- **Patrones estructurales validados**: la arquitectura exacta del DM y email que produce respuestas
- **Correcciones del usuario**: lo que explícitamente NO hacer (acumulado de sesiones)
- **Mensajes de referencia**: links a entradas del corpus con score ≥9.0

El voice profile NO es un template — es una calibración. Los agentes lo usan para alinear tono, no para copiar phrasings.

**Quién lo usa:**
- `linkedin-writer-agent`: lo lee en Paso 0 antes del corpus
- `writer-agent`: lo lee en Paso 0 antes del corpus
- No lo leen Critic, Judge ni Validator — ellos evalúan el output, no se calibran con el founder

**Cómo se actualiza:**
- Manualmente: el usuario edita el archivo directamente
- Semi-automáticamente: después de cada aprobación, el skill ofrece extraer el patrón ganador al voice profile (opcional, no bloqueante)
- El corpus sigue siendo la fuente de mensajes verbatim; el voice profile es el destilado de principios y frases

### Componente 2: Critic + Judge en paralelo

**Estado actual:**
```
Writer draft → Critic (1-2 min) → Writer rewrite → Judge (1-2 min) → [loop]
```

**Estado nuevo:**
```
Writer draft → [Critic + Judge simultáneos] → si Judge ≥8.5: aprobar
                                              → si Judge <8.5: Writer con ambos reportes → [loop]
```

El Judge no recibe el reporte del Critic en la evaluación paralela (lo cual está bien — el Judge simula al prospecto, quien no tiene el reporte del Critic). El Writer sí recibe ambos reportes para la siguiente iteración.

Ahorro por iteración: 1–2 min (la latencia del agente que antes esperaba al otro terminar).

**Aplica a:**
- `linkedin-outreach/SKILL.md` — Pasos 2 y 4 se fusionan en un paso paralelo
- `outreach/SKILL.md` — después de que pasa el Validator, Critic + Judge corren simultáneos

### Componente 3: Research paralelo en /gtm

**Estado actual:** `/gtm` con N leads → research Lead1 → research Lead2 → ... → sequential

**Estado nuevo:** `/gtm` detecta N>1 leads → lanza N investigaciones en paralelo → espera todas → luego outreach secuencial

La investigación de cada lead es completamente independiente (no hay dependencias entre ellas). El outreach queda secuencial porque el founder necesita revisar y aprobar cada mensaje.

---

## Technical Approach

### Arquitectura del voice profile

```
outreach/references/
├── voice-profile.md        ← NUEVO — voz del founder (symlinkeado via setup)
├── email-corpus.md         ← Existente — mensajes verbatim de email
├── linkedin-corpus.md      ← Existente — mensajes verbatim de DM
├── outreach-rules.md       ← Existente — reglas de email
└── linkedin-rules.md       ← Existente — reglas de LinkedIn DM
```

El voice profile vive en `outreach/references/` y se sirve automáticamente via el symlink `~/.claude/references/` que setup ya crea. No requiere cambios en setup.

### Estructura de `voice-profile.md`

```markdown
# Voice Profile — Gustavo Castañeda

## Identidad del remitente
- Nombre en firma: Gustavo / "Gus" para prospectos con rapport previo
- Cargo completo: Founder & CEO de reasoning.so
- Self-intro estándar DM: "Soy Gustavo, Founder & CEO de reasoning.so" 
  o "Soy Founder & CEO de reasoning.so" (intercambiables)
- Self-intro estándar email: incluir cargo en firma, no en el cuerpo

## Frases características — usar verbatim cuando aplique
### Escape hatch (CTA de bajo riesgo)
- "sin tema" — forma corta
- "Si no aplica, sin tema." — forma completa
- "Son 15 min y si no aplica para lo que haces, sin tema." — con tiempo acotado

### Apertura de DM
- "Hola [Nombre]! ¿Cómo estás?" — apertura estándar
- "Hola [Nombre]! Como estas?" — variante informal (cuando hay rapport)

### Ancla del pain
- "estamos resolviendo un problema muy concreto"
- "el trabajo sigue siendo manual"
- "esta capa suele consumir semanas"

### Mecanismo + compresión
- "Lo que normalmente toma semanas o meses puede convertirse en días"
- "con más trazabilidad y menos carga operativa para el equipo"
- "con menos carga manual para el equipo"

### CTA validados en campo
- DM: "¿Te haría sentido verlo aplicado a [empresa] en una demo rápida de 15 min? Si no aplica, sin tema."
- DM: "¿Te haría sentido verlo? Son 15 min y si no aplica para lo que haces, sin tema."
- Email: [ver email-corpus.md para CTAs de email aprobados]

## Patrones estructurales validados

### DM LinkedIn post-conexión (5-T)
T0: Saludo cálido — "Hola [Nombre]! ¿Cómo estás?"
T1: Self-intro como Founder & CEO + trigger específico (empresa + rol + contexto observable)
T2: Pain sectorial — términos del negocio del prospecto, hipótesis de industria (nunca diagnóstico)
T3: Mecanismo (automatizamos: limpieza, deduplicación, homologación, gobernanza) + compresión temporal
T4: CTA con tiempo acotado (15 min) + escape hatch ("sin tema")

### Email 1 (4-T)
T1: Saludo + reconocimiento de rol/tenure (no datos de prensa ni funding)
T2: Hipótesis del vertical — reto del sector como oportunidad
T3: Mecanismo + compresión + qué carga manual desaparece
T4: CTA value-framed con propósito explícito

## Correcciones acumuladas — NO hacer

### Tono y apertura
- NO abrir self-intro con "ayudamos a empresas a…" — primero el problema
- NO usar hedges de inseguridad: "creo que tal vez", "me imagino que quizá", "estamos explorando"
- NO usar "reducción en nómina" en frío — usar "menos carga manual", "menos trabajo operativo"
- NO diagnosticar el stack/setup del prospecto — hipótesis de industria, nunca diagnóstico
- NO usar jerga técnica arquitectónica: "event streams", "feature stores", "data lakes"

### LinkedIn específico
- NO abrir DM con "Gracias por conectar" o cualquier cumplido vacío
- NO hacer trigger genérico sin empresa/rol específico ("vi tu perfil")
- NO CTA sin tiempo acotado ni escape hatch

### Email específico
- NO citar datos de funding, anuncios de prensa o logros de la empresa como hook
- NO prescribir en qué debería enfocarse el equipo después del mecanismo

## Mensajes de referencia de alta puntuación
(Actualizar después de cada aprobación ≥9.0)
- DM #4 (KAM Solar): self-intro integra value prop — ver linkedin-corpus.md #4
- DM #5 (FUNO): vertical pain ultra-específico para real estate — ver linkedin-corpus.md #5
- DM #6 (Gentera): pain con activos sectoriales de fintech + 80% integrado — ver linkedin-corpus.md #6
```

### Integración en los writers

**`agents/linkedin-writer-agent.md` — Paso 0 actualizado:**

```markdown
### Antes de empezar (paso 0): consulta el voice profile y el corpus

1. Lee `~/.claude/references/voice-profile.md` — este es el perfil de voz del founder.
   Es tu calibración primaria: úsalo para alinear tono, frases y estructura ANTES de leer el corpus.
   Si la frase o patrón que necesitas está ahí, úsalo literalmente.
2. Lee la sección Índice de `~/.claude/references/linkedin-corpus.md`.
   Identifica 2–3 entradas relevantes por vertical/rol. Lee solo esas.
3. Si el corpus está vacío, el voice profile es suficiente para calibrar.
```

**`agents/writer-agent.md` — Paso 0 actualizado:**

```markdown
### Antes de empezar (paso 0): consulta el voice profile y el corpus

1. Lee `~/.claude/references/voice-profile.md` — calibración primaria del founder.
2. Lee el índice de `~/.claude/references/email-corpus.md`. Identifica entradas relevantes. Lee solo esas.
```

### Critic + Judge en paralelo — cambios en SKILL.md

**`linkedin-outreach/SKILL.md` — loop simplificado:**

```
Paso 1: LinkedIn Writer → draft
Paso 2: [LinkedIn Critic + LinkedIn Judge en paralelo]
         → Ambos reciben: ficha + linkedin-rules + draft
         → Critic: recomendaciones de mejora
         → Judge: calificación 1–10
Paso 3: ¿Judge ≥8.5? → aprobar
         ¿Judge <8.5 y iteraciones <3? → Writer reescribe con ambos reportes → regresa a Paso 2
         ¿iteraciones=3? → entregar mejor draft
```

**`outreach/SKILL.md` — loop simplificado:**

```
Paso 1: Writer → draft
Paso 2: Data Validator → PASS/FAIL
Paso 3: [Critic + Judge en paralelo] — solo si Validator dice PASS
         → Writer reescribe con ambos reportes si Judge <8.5
```

### Parallel research en /gtm

**`gtm/SKILL.md` — Paso 4 (modo multi-lead):**

Cuando la cadena incluye N>1 leads:

```
Fase A — Research en paralelo:
  Anuncia "Investigando N leads en paralelo..."
  Lanza N sub-agentes de /research simultáneos (uno por lead).
  Espera a que todos terminen antes de avanzar.

Fase B — HubSpot en paralelo (si aplica):
  Lanza N sub-agentes de /create-company → /create-contact simultáneos.
  Espera a que todos terminen.

Fase C — Outreach secuencial:
  El outreach se procesa de a un lead a la vez.
  Anuncia el orden: "Procesando outreach para [Lead1], luego [Lead2]..."
  El founder revisa y aprueba cada mensaje antes de continuar.
```

Para la paralelización se requiere agregar `Agent` a los `allowed-tools` de `gtm/SKILL.md`.

---

## Fases de implementación

### Fase 1: Voice Profile (impacto en calidad — 2–3 horas)

**Archivos:**
- [x] Crear `outreach/references/voice-profile.md` con el perfil completo de Gustavo
- [x] Actualizar `agents/linkedin-writer-agent.md` — Paso 0 lee voice-profile.md primero
- [x] Actualizar `agents/writer-agent.md` — Paso 0 lee voice-profile.md primero
- [x] Correr `./setup` para propagar (el symlink ya existe, los agents se reinstalan con los nuevos paths)

**Criterio de éxito:** El Writer produce en su primer draft frases reconocibles de Gustavo sin que el Critic tenga que corregirlas.

### Fase 2: Critic + Judge en paralelo (velocidad — 1–2 horas)

**Archivos:**
- [x] Actualizar `linkedin-outreach/SKILL.md` — reescribir loop para evaluar en paralelo
- [x] Actualizar `outreach/SKILL.md` — reescribir loop para evaluar en paralelo (post-Validator)

**Criterio de éxito:** Un loop de DM que antes tomaba Critic(2min) → Writer(2min) → Judge(2min) = 6 min ahora toma [Critic+Judge paralelo](2min) → Writer(2min) → [Critic+Judge paralelo](2min) = 4 min.

### Fase 3: Parallel research en /gtm (velocidad para N>1 leads — 1 hora)

**Archivos:**
- [x] Actualizar `gtm/SKILL.md` — Paso 4 con modo multi-lead paralelo
- [x] Agregar `Agent` a `allowed-tools` en `gtm/SKILL.md`

**Criterio de éxito:** 2 leads en paralelo completan su investigación en ~5 min en lugar de ~10 min secuenciales.

### Fase 4: Auto-actualización del Voice Profile (opcional, futuro)

**Qué hace:** Después de cada aprobación ≥9.0, el skill extrae el patrón ganador y ofrece actualizarlo en voice-profile.md.

**Por qué es opcional:** El corpus ya captura los mensajes verbatim. El voice profile puede actualizarse manualmente revisando el corpus. La auto-actualización es conveniente pero no crítica para el valor inicial.

**Archivos:**
- [ ] Actualizar `outreach/SKILL.md` Paso 9.5 — ofrecer actualización de voice profile
- [ ] Actualizar `linkedin-outreach/SKILL.md` Paso 7.5 — ídem

### Fase 5: Multi-usuario (opcional, futuro)

**Contexto:** El usuario mencionó que el plugin debería soportar múltiples founders con estilos distintos.

**Arquitectura propuesta:**
```
outreach/references/voice-profiles/
  gustavo.md       ← perfil de Gustavo
  [otro-founder].md
voice-profile.md   ← symlink al perfil activo, o archivo standalone
```

**Setup con detección de usuario:**
```bash
# En setup: detectar git user y crear symlink al perfil correcto
GIT_USER=$(git config user.name | tr '[:upper:]' '[:lower:]' | tr ' ' '-')
if [ -f "$INSTALL_DIR/outreach/references/voice-profiles/${GIT_USER}.md" ]; then
  ln -snf "$INSTALL_DIR/outreach/references/voice-profiles/${GIT_USER}.md" \
          "$INSTALL_DIR/outreach/references/voice-profile.md"
fi
```

Los agentes siempre leen `voice-profile.md` — el setup resuelve qué perfil está activo.

---

## Alternative Approaches Considered

### A: Embed voice profile en los agents (descartado)
Meter el perfil directamente en el system prompt de cada agent. Pros: siempre disponible sin file read. Cons: cuando Gustavo actualiza el perfil, habría que correr setup para re-copiar los agents. La versión externa permite live editing.

### B: Extender el corpus con sección de estilo (descartado)
Agregar `## Estilo del Founder` al final de cada corpus file. Pros: un archivo menos. Cons: el corpus crece con ejemplos y la sección de estilo se pierde entre el ruido. El voice profile separado es más fácil de mantener y más rápido de leer.

### C: Voice profile como parte del SKILL.md (descartado)
Hardcodear frases en el SKILL.md que las pasa a los agentes. Cons: actualizar el estilo requeriría editar todos los SKILL.md. Centralizar en un archivo es más sostenible.

### D: Merge Critic y Judge en un solo agente (descartado)
Un agente CriticJudge que evalúa Y califica en un solo paso. Pros: menos API calls. Cons: el Critic y el Judge tienen perspectivas distintas (revisor técnico vs prospecto). Mezclarlos reduce la calidad de cada evaluación. La paralelización captura el beneficio de velocidad sin sacrificar perspectivas.

---

## Acceptance Criteria

### Fase 1 — Voice Profile
- [ ] `outreach/references/voice-profile.md` existe con al menos: identidad, frases características, patrones estructurales, correcciones acumuladas, 3+ mensajes de referencia
- [ ] `linkedin-writer-agent.md` lee voice-profile.md en Paso 0 antes del corpus
- [ ] `writer-agent.md` lee voice-profile.md en Paso 0 antes del corpus
- [ ] Los agents instalados (`~/.claude/agents/`) tienen la ruta absoluta correcta después de correr `./setup`
- [ ] Una prueba de `/linkedin-outreach` con una ficha real produce en iteración 1 frases características de Gustavo ("sin tema", "estamos resolviendo un problema muy concreto") sin que el Critic las tenga que corregir

### Fase 2 — Critic + Judge en paralelo
- [ ] `linkedin-outreach/SKILL.md` invoca Critic y Judge simultáneamente (no en secuencia)
- [ ] Writer recibe ambos reportes para la reescritura
- [ ] El Judge sigue calificando 1–10 con mínimo 8.5 (criterio sin cambios)
- [ ] Un loop completo (Writer + [Critic+Judge] + Writer si hay reescritura) toma <5 min en condiciones normales

### Fase 3 — Parallel research
- [ ] `gtm/SKILL.md` con 2 leads lanza 2 investigaciones simultáneas
- [ ] El tiempo de investigación de 2 leads no supera 1.5x el tiempo de investigar 1 lead solo
- [ ] Los resultados de ambas investigaciones llegan completos antes de continuar al outreach

---

## Success Metrics

| Métrica | Antes | Target |
|---------|-------|--------|
| Tiempo total para 2 leads (research → outreach) | ~20 min | <12 min |
| Iteraciones promedio por lead | ~2.5 | <1.5 |
| % de drafts que pasan Judge en iteración 1 | ~20% | >50% |
| Frases de Gustavo en primer draft sin intervención de Critic | 0–1 | 3–4 |

---

## Dependencies & Risks

### Dependencias
- `setup` ya crea el symlink `~/.claude/references/` — no necesita cambios para Fase 1
- Los agents ya usan rutas absolutas post-setup (fix aplicado hoy) — voice-profile.md se servirá automáticamente

### Riesgos

**Riesgo 1: Voice profile demasiado rígido → outputs menos creativos**
Mitigación: El voice profile da frases características, no un template completo. El corpus sigue siendo la fuente de inspiración de patrones. La instrucción en el agent debe ser explícita: "el voice profile calibra el tono, no dicta el texto".

**Riesgo 2: Critic + Judge paralelo reduce calidad si el Judge necesitaba contexto del Critic**
Mitigación: En la práctica, el Judge simula al prospecto — el prospecto nunca leyó el reporte del Critic. El Judge no necesita ese contexto para evaluar el impacto del mensaje. El Writer sí necesita ambos reportes para la reescritura.

**Riesgo 3: Parallel research en /gtm consume más tokens simultáneos**
Mitigación: 2 leads en paralelo son 2 investigaciones independientes, no 10. El riesgo de rate limit es bajo. Si aparece, el fallback es volver al modo secuencial.

**Riesgo 4: Voice profile desactualizado vs corpus (divergencia)**
Mitigación: El voice profile debe revisarse cada 5–10 entradas nuevas al corpus. Agregar un recordatorio en el Paso 9.5 / 7.5 del skill: "el voice profile tiene X semanas sin actualización — considera revisarlo".

---

## Files to Create / Modify

| Archivo | Acción | Fase |
|---------|--------|------|
| `outreach/references/voice-profile.md` | **CREAR** | 1 |
| `agents/linkedin-writer-agent.md` | Actualizar Paso 0 | 1 |
| `agents/writer-agent.md` | Actualizar Paso 0 | 1 |
| `linkedin-outreach/SKILL.md` | Refactorizar loop — Critic+Judge paralelo | 2 |
| `outreach/SKILL.md` | Refactorizar loop — Critic+Judge paralelo | 2 |
| `gtm/SKILL.md` | Agregar modo multi-lead paralelo + `Agent` en allowed-tools | 3 |

**No se modifican:** corpus files, rules files, judge/critic agents (su lógica interna no cambia — solo cómo se invocan), setup (ya funciona para voice-profile via symlink existente).

---

## Sources & References

- `outreach/references/linkedin-corpus.md` — entradas #3–#6 son la fuente de verdad del estilo de Gustavo
- `outreach/references/email-corpus.md` — ídem para emails
- `agents/linkedin-writer-agent.md:38–45` — Paso 0 actual (a reemplazar)
- `agents/writer-agent.md:17` — referencia a outreach-rules (a extender con voice-profile)
- `linkedin-outreach/SKILL.md:31–86` — loop actual a refactorizar
- `outreach/SKILL.md:27–127` — loop actual a refactorizar
- `gtm/SKILL.md:129–141` — Paso 4 actual a extender con modo multi-lead
