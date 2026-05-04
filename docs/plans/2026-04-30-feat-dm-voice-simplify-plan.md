---
title: "feat: DM voice — simplify T2/T3, add two-step mode, add corpus #7–#12"
type: feat
status: completed
date: 2026-04-30
---

# feat: DM voice — simplify T2/T3, add two-step mode, add corpus #7–#12

## Diagnóstico — por qué los DMs siguen sonando a vendedor

A pesar de múltiples correcciones al framework, los agentes generan DMs que Gustavo rechaza con "me los sigue dando como si fuera un vendedor". El análisis de las conversaciones reales que han generado respuesta revela **tres causas raíz**:

### Causa 1 — T2 obligatorio fuerza un párrafo que en campo se omite

La regla actual dice: "T2 — Empatía vertical (1–2 líneas): reto del sector como hipótesis general". Los agentes la cumplen produciendo frases como:

> "En organizaciones como Bitso, que operan cripto, remesas y stablecoins en seis países, los equipos de datos suelen pasar la mayor parte del tiempo…"

Los DMs que SÍ generaron respuesta usan exactamente una de estas dos variantes:
- **Variante A** ("nos toca directo"): omiten T2 completamente y en cambio conectan el trigger directamente a la propuesta con la frase `"es un tema que nos toca directo"`.
- **Variante B** (pain vertical): 1 oración que nombra los activos de datos del sector, sin elaborar. Ejemplo: `"Vi que lideras Data, Analytics & AI en Gentera. En reasoning estamos resolviendo un problema muy concreto: los equipos de datos pasan hasta el 80% de su tiempo limpiando y conciliando información."` — el 80% ancla el pain, y el T2 termina ahí.

### Causa 2 — T3 usa enumeración técnica que el founder no usa

La regla actual: `"Nuestra plataforma automatiza esa capa: limpieza, deduplicación, homologación y gobierno de datos."` Los agentes la cumplen siempre con los 4 verbos.

Los DMs que generaron respuesta usan dos variantes completamente distintas:
- **Variante simple** (más usada): `"Nosotros estamos construyendo una plataforma que simplifica y automatiza toda la generación de analítica y BI. Se conecta a tu base de datos y en segundos tienes respuestas que hoy toman horas de trabajo manual."` — cero enumeración técnica, resultado en lugar de verbos de proceso.
- **Variante enumerada** (solo cuando hay T2 de pain sectorial): los 4 verbos aparecen en KAM Solar, FUNO, Gentera — pero nunca sin el T2 que los antecede.

La enumeración técnica no es el patrón canónico — es una variante que solo funciona cuando el T2 ya pintó el contexto.

### Causa 3 — "estamos construyendo" vs "Nuestra plataforma"

Los DMs con mejor respuesta usan `"estamos construyendo una plataforma"` — energía de startup. Los agentes usan `"Nuestra plataforma automatiza"` — energía de producto corporativo. Esta distinción es tonal pero acumulativa.

### Causa 4 — Los agentes no soportan la estrategia real de dos pasos

La estrategia de Gustavo en campo:
1. **Paso 1 (Día 0)**: Enviar solo el saludo → `"Hola [Nombre]! ¿Cómo estás?"` — esperar respuesta
2. **Paso 2 (Respuesta recibida)**: Enviar el pitch completo sin repetir el saludo

El skill actual genera siempre el DM completo con T0+T1+T2+T3+T4. No existe modo para generar solo el mensaje de pitch post-respuesta (sin el T0 que ya se envió).

---

## Archivos afectados

| Archivo | Cambio |
|---------|--------|
| `outreach/references/voice-profile.md` | Agregar frases reales + restructurar T2 como opcional |
| `outreach/references/linkedin-rules.md` | T2 opcional, T3 dos variantes, "nos toca directo" canon |
| `agents/linkedin-writer-agent.md` | Simplificar instrucciones T2/T3, agregar pregunta de modo en checklist |
| `outreach/references/linkedin-corpus.md` | Agregar entradas #7–#12 de conversaciones reales |
| `linkedin-outreach/SKILL.md` | Agregar modo post-respuesta (dos pasos) |

---

## Cambios detallados

### 1. `outreach/references/voice-profile.md`

**Sección "Frases características"** — agregar en subsección "Conector de trigger" (nueva):
```
### Conector de trigger (T1 → T2 o T1 → T3 directo)
- `es un tema que nos toca directo` — puente que reemplaza T2 completo; posiciona al founder como par, no como vendedor
- `Me llamó la atención tu perfil porque [TEMA] es un tema que nos toca directo`
```

**Agregar subsección "Señales de startup"**:
```
### Señales de startup (preferir sobre lenguaje corporativo)
- `estamos construyendo una plataforma que...` — energía de founder activo
- `No te quito mucho tiempo` — señal de respeto, baja fricción de entrada; úsalo cuando el DM va directo al pitch sin mucho preámbulo
```

**Sección "Patrones estructurales validados — DM LinkedIn"** — reescribir T2 y T3:

T2 actual (incorrecto):
```
T2 — Empatía vertical (1–2 líneas)
Pain sectorial con los términos del negocio del prospecto. Hipótesis de industria, nunca diagnóstico.
Usar activos específicos del sector: "inmuebles, contratos, inquilinos" (RE), "cartera dispersa, múltiples ERPs" (fintech), etc.
```

T2 nuevo:
```
T2 — Empatía vertical [OPCIONAL — solo una de tres variantes]
Variante A (más simple): omitir T2 completamente; conectar T1 a T3 con "es un tema que nos toca directo" al final de T1.
Variante B (1 oración): nombrar el pain con los activos de datos del sector, sin elaborar. Una oración máximo.
Variante C (pain + 80%): nombrar los activos del sector + anclar con el 80%. Usar cuando el rol del prospecto es directamente de datos (Head of Data, Director de Analytics).
Si el DM ya tiene una self-intro con value prop integrada (patrón KAM Solar), T2 es redundante — omitir.
```

T3 actual (incorrecto):
```
T3 — Mecanismo + Compresión + Liberación (2 líneas)
"Nuestra plataforma automatiza esa capa: limpieza, deduplicación, homologación y gobierno de datos.
Lo que normalmente toma semanas o meses puede convertirse en días, con menos carga operativa para el equipo."
```

T3 nuevo:
```
T3 — Mecanismo + Output (2 variantes igualmente válidas)
Variante simple (sin T2): "Nosotros estamos construyendo una plataforma que simplifica y automatiza toda la generación de analítica y BI. Se conecta a tu base de datos y en segundos tienes respuestas que hoy toman horas de trabajo manual."
Variante con enumeración (después de T2 con pain sectorial): "Nuestra plataforma automatiza esa capa — limpieza, homologación y gobierno de datos — para convertir semanas de trabajo en días, con trazabilidad."
Regla: si hubo T2, puede usarse enumeración. Si no hubo T2, usar variante simple. Nunca ambas variantes juntas.
```

**Sección "Mensajes de referencia de alta puntuación"** — agregar entradas #7–#12 después de validar el plan.

---

### 2. `outreach/references/linkedin-rules.md`

**Sección "FRAMEWORK DM (5-T)"** — editar T2 y T3:

T2 actual:
```
**T2 — Empatía vertical** (1–2 líneas)
Reto del sector como **hipótesis general**, no diagnóstico específico del prospecto...
```

T2 nuevo:
```
**T2 — Empatía vertical** [OPCIONAL]
Tres variantes — elige UNA o ninguna:
- **Variante A — "nos toca directo"**: Al final de T1, agrega `"es un tema que nos toca directo"`. Elimina T2 completamente. El DM va directo a T3. Usar cuando el trigger de T1 es suficientemente específico para que el prospecto entienda el fit.
- **Variante B — pain en 1 oración**: Una oración que nombra los activos de datos del sector del prospecto. Hipótesis general, nunca diagnóstico. Usar cuando el vertical es menos obvio y necesitas puente.
- **Variante C — pain con 80%**: Nombrar activos del sector + anclar con "los equipos de datos pasan hasta el 80% de su tiempo limpiando y conciliando información". Usar cuando el rol del prospecto es directamente de datos (Head of Data, CDO, Director de Analytics).

Si la self-intro ya integra la value prop (patrón #4 KAM Solar): omitir T2 completamente.
```

T3 actual:
```
**T3 — Mecanismo + Compresión + Liberación** (2 líneas)
Los tres elementos en orden:
1. Qué hace Reasoning (explícito, con verbos concretos: limpieza, deduplicación, homologación, gobernanza)
2. Compresión temporal (semanas/meses → días)
3. Qué trabajo manual desaparece — descriptivo, **nunca prescriptivo**...
```

T3 nuevo:
```
**T3 — Mecanismo + Output** (1–2 líneas)
Dos variantes igualmente válidas — elegir según qué usó T2:

**Variante simple** (usar cuando T2 fue omitido o "nos toca directo"):
> "Nosotros estamos construyendo una plataforma que simplifica y automatiza toda la generación de analítica y BI. Se conecta a tu base de datos y en segundos tienes respuestas que hoy toman horas de trabajo manual."
Energía de startup activa ("estamos construyendo"), output en resultado final, sin enumeración técnica.

**Variante con enumeración** (usar después de T2 con pain sectorial):
> "Nuestra plataforma automatiza esa capa — limpieza, homologación y gobierno de datos — para convertir semanas de trabajo en días, con trazabilidad y menos carga operativa para el equipo."
Solo válida después de T2. Si no hubo T2, usar variante simple.

**Prohibido**: usar ambas variantes en el mismo DM. Prohibido usar enumeración sin T2 previo.
```

**Agregar frase "nos toca directo" a la sección de TRIGGER**:
```
**Conector especial — "nos toca directo"**
La frase `"es un tema que nos toca directo"` cierra T1 y elimina la necesidad de T2. Posiciona al founder como par trabajando en el mismo problema, no como vendedor buscando un cliente. Es el conector más eficiente validado en campo: nombra el rol → dice que les aplica directamente → va a T3.
Ejemplo: "Vi que llevas el revenue analytics en Church & Dwight. Me llamó la atención tu experiencia porque es un tema que nos toca directo."
```

**Sección "FORMATO"** — actualizar conteo de palabras:

Actual: `90–140 palabras. Target 100–120.`
Nuevo: `60–120 palabras. Target 80–100. El DM de pitch post-respuesta (sin T0) puede ser tan corto como 60 palabras.`

---

### 3. `agents/linkedin-writer-agent.md`

**Instrucción antes de escribir #5** — reemplazar:

Actual:
```
5. **Confirma que el DM enuncia el mecanismo de Reasoning explícitamente** — no solo el problema. Si no aparece una frase del tipo "Nuestra plataforma automatiza X" / "automatizamos la capa de limpieza, deduplicación, homologación", reescribe antes de entregar
```

Nuevo:
```
5. **Confirma que el DM enuncia el mecanismo de Reasoning**. Dos variantes válidas:
   - Simple: "estamos construyendo una plataforma que simplifica y automatiza toda la generación de analítica y BI. Se conecta a tu base de datos y en segundos tienes respuestas que hoy toman horas de trabajo manual."
   - Con enumeración (solo si hubo T2 con pain sectorial): "automatizamos esa capa — limpieza, homologación y gobierno de datos — para convertir semanas en días."
   Si no aparece ninguna de las dos, reescribe. Si usaste variante con enumeración sin T2 previo, reescribe con variante simple.
```

**Instrucción antes de escribir #6** — reemplazar:

Actual:
```
6. **Nombra el problema antes que la solución.** La self-intro no abre con "ayudamos a empresas a…" — primero el problema concreto que el prospecto reconoce, luego el mecanismo de Reasoning
```

Nuevo:
```
6. **T2 es opcional — elige la variante correcta para el prospecto**:
   - Si el trigger de T1 es específico y el rol del prospecto hace el fit obvio: usa `"es un tema que nos toca directo"` al final de T1 y ve directo a T3.
   - Si el vertical necesita contexto: usa 1 oración de pain con los activos del sector.
   - Si el rol del prospecto es directamente de datos (Head of Data, CDO): usa el 80% ancla.
   - Si ya integraste la value prop en la self-intro (patrón KAM Solar): omite T2.
   No fuerces T2 si el DM fluye mejor sin él.
```

**Agregar al checklist de autocrítica**:
```
• Modo del DM: [COMPLETO (T0+T1+T2+T3+T4) / POST-RESPUESTA (T1+T2+T3+T4 sin saludo)]
• T2 elegido: [OMITIDO + "nos toca directo" / 1 oración pain / pain + 80%] — [cita la línea o "omitido"]
• T3 variante: [SIMPLE / CON-ENUMERACIÓN] — [cita la línea del mecanismo]
• Energía de startup: ["estamos construyendo" usado / "Nuestra plataforma" — revisar]
```

---

### 4. `outreach/references/linkedin-corpus.md`

Agregar entradas #7–#12 **antes** de la línea `## Cómo se agregan entradas nuevas`. Una por cada conversación real que generó respuesta:

| ID | Conversación | Patrón |
|----|-------------|--------|
| #7 | Francisco/Nestlé (feedback angle corto, respuesta con pain exacto) | Feedback angle ultra-corto → respuesta que detalla el pain |
| #8 | Miguel/Church & Dwight (trigger de rol + "nos toca directo" → demo) | "nos toca directo" como conector T1→T3 |
| #9 | Dani PhD (muy suave, compartió PDF, feedback angle) | Feedback angle para perfil académico/consultor senior |
| #10 | Luis/Petco ("No te quito mucho tiempo" + trigger de innovación) | Frase de bajo compromiso + trigger de rol de innovación |
| #11 | Sebas/MercadoLibre (respuesta amigable, ya tiene herramienta interna) | Conversación de dos pasos: saludo → pitch; respuesta de "no fit ahora" |
| #12 | Tomás/Ben & Frank (consultó CFO, no apto pero respondió cortés) | Patrón que genera respuesta aunque no haya fit |

Las entradas deben documentar:
- El DM enviado (verbatim del mensaje de pitch, omitiendo nombre real)
- La respuesta recibida (para las conversaciones donde aplica — contexto del patrón)
- Lo que funcionó vs lo que no
- La lección para los agentes

**Actualizar el Índice** con las filas #7–#12.

---

### 5. `linkedin-outreach/SKILL.md`

**Agregar Paso 0 — Detectar modo** antes del Paso 1 actual:

```markdown
### Paso 0 — Detectar modo del DM

Antes de invocar al Writer, determina si el usuario está en el flujo de:

**MODO COMPLETO** (default): El founder no ha contactado al prospecto aún. Genera el DM con T0 (saludo) + T1 + T2/T3/T4.

**MODO POST-RESPUESTA**: El founder ya envió el saludo ("Hola [Nombre]! ¿Cómo estás?") y recibió respuesta. Ahora necesita el mensaje de pitch. Genera T1 + T2/T3/T4 — sin T0 (no repetir el saludo).

**Cómo detectar el modo:**
- Si el usuario menciona "ya le mandé el saludo", "ya me respondió el saludo", "ya conectamos y me respondió" → MODO POST-RESPUESTA
- Si el argumento incluye `post-respuesta`, `pitch`, `ya respondió` → MODO POST-RESPUESTA
- Default sin contexto → MODO COMPLETO

**Anunciar el modo detectado al usuario** en 1 línea antes de invocar al Writer:
- Modo completo: "Generando DM completo (saludo + pitch) para [Empresa]."
- Modo post-respuesta: "Generando mensaje de pitch post-respuesta para [Empresa] — sin saludo inicial."

Pasar el modo al `linkedin-writer-agent` en el prompt de invocación.
```

**Actualizar la instrucción al Writer en Paso 1** para incluir el modo:

```
Invoca al `linkedin-writer-agent` con:
- La ficha de investigación del lead
- El archivo `outreach/references/linkedin-rules.md`
- El archivo `outreach/references/voice-profile.md`
- **El modo detectado**: COMPLETO (incluir T0) o POST-RESPUESTA (omitir T0 — el saludo ya se envió)
```

---

## Acceptance Criteria

- [ ] `voice-profile.md` contiene "nos toca directo", "Me llamó la atención tu perfil porque", "No te quito mucho tiempo" y "estamos construyendo" en frases características
- [ ] `voice-profile.md` documenta T2 como opcional con las 3 variantes
- [ ] `voice-profile.md` documenta T3 con dos variantes (simple vs enumeración) y la regla de cuándo usar cada una
- [ ] `linkedin-rules.md` T2 es explícitamente opcional con las 3 variantes documentadas
- [ ] `linkedin-rules.md` T3 tiene las 2 variantes documentadas con la regla de cuándo usar enumeración
- [ ] `linkedin-writer-agent.md` instrucciones de T2/T3 reflejan opcionalidad y variantes
- [ ] `linkedin-writer-agent.md` autocrítica incluye campo "T2 elegido" y "T3 variante"
- [ ] `linkedin-corpus.md` tiene entradas #7–#12 con las conversaciones reales
- [ ] `linkedin-corpus.md` índice actualizado con filas #7–#12
- [ ] `linkedin-outreach/SKILL.md` tiene Paso 0 de detección de modo
- [ ] Corriendo `/linkedin-outreach` con un lead de fintech produce un DM que usa "nos toca directo" en lugar de un párrafo de T2 elaborado (o usa pain vertical en 1 oración)
- [ ] Corriendo `/linkedin-outreach post-respuesta [empresa]` produce un DM sin saludo inicial

---

## Notas de implementación

- Editar los archivos en este orden: `linkedin-rules.md` → `voice-profile.md` → `linkedin-writer-agent.md` → `linkedin-corpus.md` → `linkedin-outreach/SKILL.md`
- Las entradas #7–#12 del corpus requieren que el usuario confirme los textos verbatim de los DMs reales antes de documentarlos — si no están disponibles en la sesión, pedir al usuario que los pegue
- No tocar `email-corpus.md`, `outreach-rules.md` ni los agentes de email — este plan solo afecta el flujo de LinkedIn
- Después de implementar, correr `/linkedin-outreach` con el lead de Bitso (el que el usuario rechazó) para verificar que el nuevo DM no tenga T2 elaborado
