# Email Corpus — Outreach Lessons

> Biblioteca viva de emails reales con sus correcciones. Sirve como
> **inspiración** para writer/critic — NUNCA para copiar.
>
> Cada entrada captura un patrón concreto que se aprobó o rechazó, con
> la lección internalizable. Crece con el uso: cuando un email se
> rechaza con feedback útil, se agrega como nueva lección.
>
> **Anonimización**: nombres del contacto reemplazados por `[Contacto]`.
> Empresas reales preservadas (mantienen el grounding del vertical sin
> exponer al prospecto individual).

## Cómo usar este corpus

- **writer-agent**: antes de escribir, lee 2–3 entradas que matcheen el
  vertical o el rol del prospecto. Si no hay match exacto, lee 2–3
  generales para calibrar el sistema. NO copies frases — el corpus es
  inspiración de patrones, no template de phrasings.
- **critic-agent**: cuando flagees un eje, **cita la entrada del corpus
  si aplica** ("este closer cae en el patrón de #2, rechazado por
  workflow inventado"). El feedback se vuelve concreto y memorable
  para la siguiente iteración del writer.

---

## #1 — T1 con factoide de Google (lending / fintech)

**Prospecto**: Director of Machine Learning en **Konfio** (fintech
mexicana de crédito a PyMEs).

**Email original (rechazado)**:

> Subject: licencia y los modelos
>
> [Contacto],
>
> El compromiso de Konfio para Plan México — $2.5B USD en 24 meses —
> es uno de los movimientos más ambiciosos en crédito PyME en LATAM.
>
> [...resto del email...]

**❌ Lo que se rechazó**:
- T1 abrió con cifra de funding y nombre de programa público ($2.5B USD,
  Plan México)
- El factoide quedó **desconectado** del resto del email — se sintió
  copy-paste de Google
- Citó información que cualquier reclutador encuentra públicamente; no
  demostraba que conocíamos al prospecto

**✅ Versión aprobada después de iterar**:

> Hola [Contacto], viendo tu rol como Director of Machine Learning en
> una fintech con la complejidad operacional de Konfio…

**📝 Lección**: T1 reconoce a la **persona** (puesto, trayectoria,
expertise) — nunca cita noticias, funding, ni programas con montos.
La empatía vertical va en T2, no en T1. Si la frase de apertura
podría aparecer en un comunicado de prensa, es factoide.

---

## #2 — T3 closer inventando workflow (fintech solar)

**Prospecto**: VP de Expansión y Financiamiento Solar en **KAM**
(empresa de financiamiento solar en CAM/MX).

**Email original (rechazado)**:

> Hola [Contacto], vi que lideras expansión y financiamiento solar en
> KAM. En fintech solar, compilar datos de satélite, geográficos y
> regulatorios para evaluar viabilidad y riesgo es donde el equipo pasa
> la mayoría del tiempo — para cada proyecto, toma semanas.
>
> Nosotros automatizamos ese 80% — limpieza, deduplicación, homologación
> y gobernanza de esos datos. Lo que hoy toma semanas queda listo en
> días. Tu equipo se libera de compilar satélite, geolocación y
> regulatorio — y se enfoca en **scoring de riesgo y aprobación de deals
> que aceleran tu pipeline de financiamiento.**

**✅ Lo que funcionó**:
- T1 saludo + role recognition limpio ("vi que lideras expansión y
  financiamiento solar en KAM")
- T2 empatía vertical concreta (satélite, geográficos, regulatorios —
  3 fuentes obvias del vertical)
- T3 producto + compresión declarados correctamente (operaciones
  concretas + 80% + semanas → días)

**❌ Lo que se rechazó**:
- T3 closer fabricó workflow específico ("aprobación de deals que
  aceleran tu pipeline de financiamiento") que asume conocimiento del
  proceso interno del equipo
- Cita textual del rechazo: "termina diciendo que su equipo se puede
  centrar en un flujo muy particular que ni siquiera sabemos si hacen,
  no hay que llegar a algo tan ultra específico"

**📝 Lección**: closer al **nivel de CATEGORÍA** del rol (lo que el rol
implica defendiblemente — "evaluación de viabilidad y aprobación de
proyectos") — NO al **nivel de WORKFLOW inventado** (pasos específicos
del proceso interno que solo sabríamos si trabajáramos ahí). Test
rápido: si el closer requiere que sepas el flujo de trabajo del equipo,
es invención. Mantente en categoría.

---

## #3 — Email demasiado largo (educación / IT)

**Prospecto**: Director IT & Transformación Digital en **Universidad
Anáhuac** (institución educativa con expansión a Brasil y Colombia).

**Email original (rechazado por longitud, ~145 palabras vs límite 75–80)**:

> Hola [Contacto]. Como Director IT & Transformación Digital en Anáhuac
> desde hace ~3 años, llevas orquestando la gobernanza de datos a través
> de múltiples campus.
>
> Uno de los desafíos que veo en instituciones que expanden a nuevos
> países es que la consolidación de datos entre sistemas se vuelve
> exponencialmente más compleja. En tu caso, Brasil y Colombia
> multiplican esa complejidad: el equipo pasa la mayoría del tiempo
> reconciliando y homologando registros manualmente — limpieza,
> deduplicación, gobernanza de datos sensibles (calificaciones,
> información de estudiantes) y cumplimiento de trazabilidad
> regulatoria entre campus.
>
> Nuestros agentes de data quality usan machine learning para
> automatizar ese 80% manual. Lo que hoy toma semanas queda listo en
> días. El equipo deja de pasar el día armonizando datos entre campus
> y se enfoca en decisiones estratégicas: qué expansión académica es
> rentable, dónde optimizar eficiencia.
>
> ¿Te interesaría una llamada breve para que entiendas cómo el 80%
> manual desaparece en tu contexto de expansión Brasil y Colombia?

**✅ Lo que funcionó**:
- T1 sólido — saludo + role recognition con tenure ("desde hace ~3 años")
- T2 con empatía vertical concreta — campus, registros sensibles,
  trazabilidad regulatoria
- T3 declaró producto + compresión correctamente
- Closer al nivel de categoría defendible ("decisiones estratégicas:
  qué expansión académica es rentable")

**❌ Lo que se rechazó**:
- **~145 palabras** (rule = 75–80). Caería el budget de 1 pantalla de
  teléfono — el prospecto deja de leer
- T2 era 3 oraciones donde 1 alcanzaba: la primera sobre la complejidad
  de expansión, la segunda específica a Brasil/Colombia, la tercera
  enumerando pains. Una sola oración condensada hace el mismo trabajo
- En general: validator no detectaba longitud — bug del guardrail que
  se corrigió con chequeo determinístico #12

**📝 Lección**: longitud es regla inviolable. Cuando el contenido es
bueno pero excede 80 palabras, recortar T2 (combinar oraciones) o T3
(remover compresión temporal si ya está implícita en otro lado) — nunca
recortar T1 (saludo + reconocimiento) ni T4 (CTA value-framed). El
validator chequea esto determinísticamente; sobre 80 = fail.

---

## #4 — T1 con factoide de noticias camuflado (manufactura solar)

**Prospecto**: Líder de expansión en **Norau** (proyecto de manufactura
solar con expansión a Brasil).

**Email original (rechazado)**:

> Subject: evaluación solar en brasil
>
> Hola [Contacto],
>
> **Viendo que estás liderando la expansión de Norau a Brasil.** En
> manufactura con múltiples ubicaciones (como proyectos de energía
> con regulaciones distintas — CFE vs ANEEL), compilación de datos y
> normalización sigue siendo 80% manual — satélite, cadastral,
> infraestructura. Nosotros automatizamos limpieza, deduplicación,
> homologación y gobernanza — lo que toma semanas queda listo en días.
> Tu equipo deja de compilar y se enfoca en viabilidad financiera y
> cierre de decisiones.
>
> ¿Te interesaría una llamada para que veas dónde liberas tiempo en
> tu evaluación de viabilidad en Brasil?

**❌ Lo que se rechazó**:
- T1 ("Viendo que estás liderando la expansión de Norau a Brasil") es
  un **factoide de noticias camuflado de role recognition** — la
  expansión a Brasil es una noticia pública/press release; pretender
  que es role recognition no engaña al prospecto
- A T2 le faltaba empatía: saltó muy rápido a "compilación 80% manual"
  sin tender el puente narrativo de imaginar el día a día
- Cita textual: "ataca muy la info de la empresa pero no le está
  diciendo que les resolvemos ni cómo, le falta entendimiento"

**📝 Lección**: distinguir entre **role recognition puro** (tenure,
puesto, expertise — "como [Puesto] en [Empresa] desde hace X años") y
**role recognition anclado en noticias** ("vi que estás liderando
[evento reciente público]"). El segundo es factoide camuflado: si la
frase implica que leíste su press release, es Google factoid aunque no
mencione cifras. Mantente en tenure/puesto, sin verbos de "vi que…
[hicieron noticia]".

---

## #5 — T2 saltó mecanismo + closer débil inventado (manufactura masiva)

**Prospecto**: Líder de Datos en **Grupo Bimbo** (manufactura de
consumo masivo, multi-país).

**Email original**: no preservado verbatim — capturado desde feedback
del rechazo.

**✅ Lo que funcionó**:
- T1 sólido (primer párrafo descrito como "muy bueno" por el revisor)

**❌ Lo que se rechazó**:
- T2 cayó en no enunciar el mecanismo del producto — saltó del problema
  al closer sin el puente "nuestra plataforma corre agentes que…"
- Closer fabricó propósito que no podemos defender ("...se enfoca en
  optimizar margen") — no sabemos si "optimizar margen" es algo que el
  equipo de datos del prospecto haga en su día a día. Mismo patrón que
  #2 (KAM): closer al nivel de workflow inventado en lugar de categoría
- Cita textual: "el segundo cayó otra vez a no poner cómo lo
  solucionamos y el cierre es algo débil, incluso no sabemos si
  'optimizar margen' es algo que hace ese equipo en su día a día"

**📝 Lección**: dos patrones combinados:
1. **Estructura T2→T3 debe sostenerse** incluso cuando T1 sale fluido
   — el writer no puede saltarse declarar el mecanismo. El validator
   chequea presencia de operaciones concretas + 80% + compresión
   determinísticamente para evitar este drift.
2. **El closer category-level (no workflow inventado)** aplica también
   a verbos económicos como "optimizar margen", "reducir CAC", "mejorar
   EBITDA" — son outcomes que SÍ ocurren en la empresa, pero no
   necesariamente son lo que el equipo de DATOS del prospecto hace. Si
   no podemos defender que el rol/equipo del prospecto trabaja en eso,
   es invención.

---

## Cómo se agregan entradas nuevas

Cuando se rechace un email con feedback útil, agregar una entrada
siguiendo el formato:

1. **Header**: `## #N — [Patrón corto] ([vertical])`
2. **Prospecto**: rol + empresa real + 1 línea de contexto del vertical
3. **Email original** (citado verbatim cuando se preserve, descrito
   cuando no)
4. **✅ Lo que funcionó** (cuando aplique — no todos los emails
   tienen aciertos)
5. **❌ Lo que se rechazó** con cita textual del feedback si está
   disponible
6. **📝 Lección**: 2–4 líneas con el principio internalizable

Anonimización: solo el contacto (nombre del prospecto) — empresas
preservadas para mantener grounding del vertical.
