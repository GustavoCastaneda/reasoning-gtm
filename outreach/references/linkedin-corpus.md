# LinkedIn DM Corpus — Reasoning Labs

Biblioteca de DMs aprobados y rechazados. Cada entrada captura qué funcionó, qué se rechazó y por qué — es inspiración de **patrones**, no plantilla de **phrasings**.

---

## Índice

| ID | Vertical | Rol | Patrón | Estado |
|----|----------|-----|--------|--------|
| #1 | CPG / Retail | Revenue analytics / Pricing | Hook de rol + escape hatch "sin tema" | Aprobado — abrió demo |
| #2 | CPG / Manufactura | Head of Data & Analytics | Feedback angle — "tu experiencia es valiosa" | Aprobado — abrió demo |

---

## Cómo usar este corpus

**Antes de escribir o evaluar**, lee el índice, identifica las 2–3 entradas más relevantes por vertical y rol del prospecto, y lee solo esas saltando directo a `## #N`. No leas el archivo entero.

**No copies frases.** El corpus es inspiración de patrones — qué tipo de hook funciona, qué cierre se rechazó y por qué. El loop existe para descubrir la frase específica para cada prospecto.

---

## #1 — Hook de rol + escape hatch "sin tema" (CPG / Pricing)

**Prospecto**: Revenue Analytics & Pricing lead en **Church & Dwight** (CPG con portafolio de marcas de consumo masivo, modelos de elasticidad y forecasting de demanda).

**DM que abrió demo** (producto previo — estructura aplicable):

> [Contacto], vi que llevas el revenue analytics y pricing en Church & Dwight. Me llamó la atención tu experiencia con modelos de elasticidad y forecasting — es un tema que nos toca directo.
> Nosotros estamos construyendo una plataforma que simplifica y automatiza toda la generación de analítica y BI. Se conecta a tu base de datos y en segundos tienes respuestas que hoy toman horas de trabajo manual.
> ¿Te haría sentido verlo? Son 15 min y si no aplica para lo que haces, sin tema.

(~70 palabras. Nota: el producto descrito era distinto al actual — el patrón estructural es lo que aplica.)

**✅ Lo que funcionó**:
• **Hook de rol específico**: `"vi que llevas el revenue analytics y pricing en Church & Dwight"` — nombra persona + rol + empresa en la primera línea. Sin "gracias por conectar", sin cumplido.
• **Posicionamiento como par**: `"es un tema que nos toca directo"` — el founder no se posiciona como vendedor sino como alguien que trabaja en el mismo problema. Baja la guardia inmediatamente.
• **Escape hatch en el CTA**: `"Son 15 min y si no aplica para lo que haces, sin tema."` — le da permiso explícito de decir no. Paradójicamente, esto aumenta la tasa de respuesta porque elimina el miedo a quedar atrapado en una llamada de ventas. La combinación "tiempo acotado + escape hatch" hace que pedir la reunión en el primer DM sea perfectamente válido.

**📝 Lección**: La regla "nunca pidas reunión en el primer DM" es demasiado rígida. Sí se puede pedir demo/reunión en el primer DM **con dos ingredientes obligatorios**: (1) límite de tiempo explícito (15 min, demo rápida) + (2) escape hatch ("si no aplica, sin tema" / "si no es el momento, sin problema"). Sin ambos ingredientes, es presión. Con ambos, es invitación de bajo riesgo. El hook de rol + puente "nos toca directo" + escape hatch es un patrón probado.

---

## #2 — Feedback angle — "tu experiencia es valiosa" (CPG / Analytics)

**Prospecto**: Head of Data & Analytics México en **Nestlé** (multinacional CPG con operaciones en múltiples mercados, team de analítica con alto volumen de datos).

**DM que abrió demo** (producto previo — estructura aplicable):

> Hola [Contacto]! Como estas?
> No te quito mucho tiempo, soy CEO y Co-Founder en reasoning.so, ayudamos a las empresas a automatizar la generación de BI y analítica, creo que les podemos ayudar a agilizar el dia a dia de del area de analítica de Nestlé.
> Incluso de no ser así, tu feedback nos sería muy valioso dada tu experiencia.
> Tendrás un espacio libre para mostrarte una demo rápida?
> Abrazo!

(~60 palabras. Nota: producto previo, estructura aplicable.)

**✅ Lo que funcionó**:
• **Feedback angle**: `"Incluso de no ser así, tu feedback nos sería muy valioso dada tu experiencia."` — este es el mecanismo real. En lugar de vender directamente, posiciona al prospecto como experto cuya opinión vale. Convierte "dame 15 min para venderte" en "quiero tu criterio como referente". Para un Head of Data en una empresa como Nestlé, esto activa el deseo de ser consultado como par, no de ser convencido como cliente.
• **Energía de founder**: tono informal, "abrazo!", "como estas?" — auténtico sin ser forzado. El prospecto sabe que está hablando con el founder real, no con un SDR leyendo un script.
• **"No te quito mucho tiempo"**: señal de respeto que baja la fricción de entrada.

**❌ Puntos débiles (funcionó a pesar de esto, no gracias a esto)**:
• Hook genérico ("Hola [nombre]! Como estas?") — no específico al rol ni al vertical. Aplica a cualquier prospecto.
• Self-intro al inicio ("soy CEO y Co-Founder en reasoning.so") — el prospecto ya vio tu perfil. Presentarte de nuevo es redundante y consume el espacio del hook.
• El producto descrito era BI/analítica — para Reasoning Labs actual (pipeline de datos, 80% manual), el T3 cambiaría completamente.

**📝 Lección**: El **feedback angle** es un patrón alternativo poderoso cuando hay duda sobre fit o cuando el prospecto es de alto estatus. "Tu feedback nos sería muy valioso dada tu experiencia" funciona porque convierte la dinámica de vendedor-cliente en par-consultado. Úsalo como variación cuando el hook de rol estándar no termina de anclar, o cuando el prospecto es muy senior y el fit es menos obvio. La clave es que la frase suene genuina — si el prospecto siente que es truco retórico, pesa en contra. Solo funciona cuando el founder realmente querría su opinión. Para Reasoning Labs, aplica bien con CDOs/Heads of Data senior donde el feedback sobre el producto realmente sería valioso.

---

## Cómo se agregan entradas nuevas

El skill `/linkedin-outreach` agrega una entrada automáticamente después de cada DM aprobado. Formato:

```markdown
## #[N] — [Patrón corto de 3–6 palabras] ([vertical])

**Prospecto**: [Rol] en **[Empresa]** ([descripción de 1 línea del vertical]).

**DM aprobado por el [Judge / usuario] ([score]/10)**:

> [cuerpo verbatim — nombre del contacto → [Contacto]]

([N] palabras.)

**✅ Lo que funcionó**:
• [elemento específico — hook, empatía vertical, CTA, etc.]

**❌ Lo que se rechazó / Lo que el usuario corrigió**:
• [qué se detectó + cita del feedback]

**📝 Lección**: [principio internalizable en 2–4 líneas]
```
