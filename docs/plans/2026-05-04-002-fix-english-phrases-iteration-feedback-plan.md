---
title: "fix: English phrase variants in voice profile + lead iteration feedback"
type: fix
status: completed
date: 2026-05-04
---

# fix: English phrase variants en voice profile + feedback de iteraciones por lead

## Por qué existe este plan

El mismo lead (lululemon — Domenic Fayad) produce outputs diferentes dependiendo de si el agente lee los archivos de referencia de primera instancia o si recibe un prompt enriquecido con frases en inglés y lecciones de intentos previos.

Diferencias observadas:

| Elemento | Primera instancia (agente solo) | Plugin con contexto inyectado |
|----------|--------------------------------|-------------------------------|
| T0 | "Hi Domenic! How are you?" | "Hey Domenic! How's it going?" |
| Escape hatch | "If it's not a fit, no worries." | "If it's not relevant to what you're building, no worries." |
| T2 ángulo | Tiempo de limpieza manual | Calidad de señal upstream del modelo |
| T3 frase | "We're building..." (startup — incorrecto con T2 pain) | "Our platform automates..." (correcto para T2 con pain) |

Los primeros dos gaps son por frases en inglés ausentes en `voice-profile.md`.
El tercero fue inyectado como "LECCIÓN CLAVE DE INTENTOS ANTERIORES" manualmente — no vive en ningún archivo.
El cuarto es consecuencia de no tener las variantes T3 en inglés documentadas con la regla de cuándo usar cada una.

## Problema raíz

### Gap 1: voice-profile.md no tiene frases en inglés

El agente calibra leyendo `voice-profile.md`. Todas las frases características están en español. Para prospectos US/Canadá, el agente infiere equivalentes — y se equivoca en tono o phrasing específico validado en campo.

### Gap 2: No hay mecanismo para feedback por lead

Cuando ya hubo un intento de DM para un lead y el founder lo corrigió (ej: "el pain no es sobre tiempo manual sino sobre calidad de señal"), esa corrección desaparece. El siguiente intento empieza de cero y puede repetir el mismo error.

La "LECCIÓN CLAVE DE INTENTOS ANTERIORES" del plugin fue inyectada manualmente — no hay archivo donde viva ni forma de que el SKILL la pase automáticamente.

## Cambios propuestos

### 1. Nueva sección en voice-profile.md — frases en inglés

Agregar `## Frases en inglés (prospectos US/Canadá)` con equivalentes validados:

```
### T0 — Apertura
- `"Hey [Nombre]! How's it going?"` — estándar (peer-level, casual)
- `"Hey [Nombre]! How are you?"` — variante ligeramente más formal

### Escape hatch
- `"no worries"` — forma corta
- `"If it's not relevant to what you're building, no worries."` — forma completa
- `"If it doesn't apply to what you're working on, no worries."` — variante

### Pain connector (equivalente en inglés de "nos toca directo")
- `"it's something we work on directly"` — cierra T1, va directo a T3

### T3 variante simple (cuando T2 fue omitido o se usó pain connector)
- `"We're building a platform that simplifies and automates the entire data prep layer before analytics can run. It connects to your database and in seconds you get answers that today take hours of manual work."`
- Usar "We're building" — startup energy, solo cuando NO hubo T2 con pain.

### T3 variante con enumeración (solo después de T2 con pain sectorial)
- `"Our platform automates that layer — cleaning, deduplication, and harmonization — turning weeks of prep into days, with more traceability and less manual load on the team."`
- Usar "Our platform" — el pain de T2 ya estableció contexto. NO mezclar con "We're building".

### CTA canónico
- `"Would it make sense to see it applied to [empresa] in a quick 15-min demo? If it's not relevant to what you're building, no worries."`
- `"Would it make sense to see it? It's 15 min and if it doesn't apply to what you're building, no worries."`
```

### 2. Sección opcional `## Iteraciones previas` en ficha.md

Formato a agregar al final de cualquier ficha cuando ya hubo intentos:

```markdown
## Iteraciones previas

### Intento 1 — [fecha]
- **Ángulo probado**: [pain usado en el intento]
- **Resultado**: [aprobado / rechazado / corrección del founder]
- **Corrección**: [qué cambió exactamente]
- **Lección**: [principio que el agente debe recordar en el próximo intento]

### Ejemplo T2 validado para este perfil
"[frase que funcionó — verbatim o near-verbatim]"
```

Sección completamente opcional — fichas sin ella funcionan igual que hoy.

### 3. Actualizar SKILL.md — pasar contexto de iteraciones al agente

En el Paso 1 (invocar linkedin-dm-agent), si la ficha contiene `## Iteraciones previas`, incluir esa sección como contexto adicional:

```
Contexto adicional — lecciones de intentos previos con este lead:
[contenido de ## Iteraciones previas]
```

Sin esa sección en la ficha → comportamiento inalterado.

### 4. Actualizar linkedin-dm-agent.md PASO 0

Agregar como cuarta lectura en PASO 0:

**Cuarto — Contexto de iteraciones previas (si fue provisto por el SKILL):**
Si hay lecciones de intentos anteriores, tratarlas como la señal de calibración de mayor prioridad para este lead. Sobreponen las inferencias del corpus.

### 5. Documentar la corrección en ficha de lululemon

Agregar `## Iteraciones previas` a `leads/lululemon/ficha.md` con la lección capturada:
- Pain debe ser sobre **calidad de señal** que afecta confiabilidad del modelo, no tiempo de limpieza manual
- Especificidad ganadora nombra "Audience Science" y "behavioral events" — no retail genérico
- T2 como hipótesis del ROL, no del vertical en general
- Ejemplo T2 validado: "In that role, a lot of the work sits upstream of every model — making sure behavioral events across app, email, and paid channels are consistently defined before anything runs."

## Archivos a modificar

| Archivo | Cambio | Tipo |
|---------|--------|------|
| `outreach/references/voice-profile.md` | Nueva sección `## Frases en inglés` | Additive |
| `linkedin-outreach/SKILL.md` | Paso 1 pasa contexto de iteraciones si existe en ficha | Additive |
| `agents/linkedin-dm-agent.md` | PASO 0 consume contexto de iteraciones si fue provisto | Additive |
| `leads/lululemon/ficha.md` | Agregar `## Iteraciones previas` con lección capturada | Additive |

`linkedin-rules.md` no requiere cambios — las variantes T3 ya están documentadas; la especificación en inglés va en voice-profile.md.

## Criterios de aceptación

- [ ] `voice-profile.md` tiene sección `## Frases en inglés` con T0, escape hatch, pain connector, T3 simple, T3 enumeración y CTA canónico
- [ ] El agente genera "Hey [Nombre]! How's it going?" para prospectos US/Canadá sin inyección manual
- [ ] El agente genera el escape hatch correcto en inglés sin inyección manual
- [ ] T3 variante simple usa "We're building" / T3 variante con enumeración usa "Our platform" — sin mezcla
- [ ] `leads/lululemon/ficha.md` tiene `## Iteraciones previas` con la lección del intento anterior
- [ ] `SKILL.md` pasa el contenido de `## Iteraciones previas` al agente cuando existe en la ficha
- [ ] Un segundo intento con lululemon sin inyección manual produce output equivalente al del plugin

## Riesgo

Bajo. Todos los cambios son aditivos. El path sin iteraciones previas no cambia. Las frases en inglés son una nueva sección en voice-profile — no modifican las existentes en español.
