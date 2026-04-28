# Reglas de Outreach — Reasoning Labs
> Fuente única de verdad para los agentes de outreach.
> Writer, Critic y Judge deben usar estas reglas como referencia.
> **Foco: solo Email 1.** Email 2/3 y LinkedIn están fuera del alcance de este flujo — se manejan después con contexto fresco.

---

## PRODUCTO — QUÉ HACE REASONING

**Datos puntuales que anclan el email** (son la fuente de verdad — no fabricar números, no inventar casos peer):

- **Operaciones concretas que automatizamos**: limpieza, deduplicación, homologación, gobernanza, estructuración. Estos son los verbos que debe ver el prospecto — no abstracciones como "preparación" u "orquestación".
- **Fracción del pipeline**: 80% del trabajo manual de los equipos de datos (limpiar/homologar/deduplicar/gobernar). Esa cifra está autorizada.
- **Compresión de tiempo**: flujos que tomarían **semanas o meses**, los hacemos en **un par de días**.
- **Industrias activas**: fintech, manufactura, retail. Ese es el ground truth — no inventar peers anónimos con N modelos.

**Frases ancla válidas** (úsalas como bloque concreto en el email — varía la redacción pero respeta los datos):
- "nuestra plataforma automatiza limpieza, deduplicación, homologación y gobernanza — el 80% del pipeline manual"
- "lo que hoy toma semanas o meses, lo hacemos en un par de días"
- "trabajamos con equipos en fintech, manufactura y retail"

**Mecanismo en 4 capas (contexto largo, no para el email — solo para que el writer entienda):**
1. **Conexión** — Conecta fuentes tal como están (PostgreSQL, SQL Server, Excel, CSV, APIs). Sin requerir limpieza previa.
2. **Estructuración** — Agentes que deduplican, estandarizan formatos, unifican fuentes y construyen una ontología viva. 80% automático, 20% confirmado con humano.
3. **Análisis** — Detección de anomalías y diagnóstico continuos.
4. **Acción** — Entrega proactiva por Slack/email/Teams + Q&A en lenguaje natural.

**Lo que Reasoning NO es** (nunca implicar esto):
- No es un dashboard ni reemplazo de Power BI/Looker
- No es una consultora de MDM ni implementación
- No es "AI genérico" ni un LLM wrapper
- No es un orquestador de Spark/Kafka — es la capa que elimina ese trabajo

**NO inventes nunca:**
- Casos de referencia con detalles fabricados ("trabajamos con un equipo de scoring con 15+ modelos en producción", "una financiera con N sucursales") — hoy hay **un piloto activo** y los datos puntuales reales son los de arriba. No fabriques peers.
- Cifras de resultado fuera de las autorizadas (semanas/meses → días). Nada de "5x más rápido", "85% más eficiente", "3 semanas a 4 días".
- Verbos abstractos cuando hay verbos concretos ("preparación de datos" debe ser "limpieza, deduplicación, homologación").

---

## MECANISMO — OBLIGATORIO

El Email 1 **debe contener una frase explícita** que enuncie qué hace Reasoning como mecanismo (no solo qué problema resuelve). Sin esa frase, el email falla aunque cumpla todo lo demás.

**Test de validez**: si un prospecto al terminar de leer no puede articular en una sola oración "estos automatizan X", el email no sirve.

**Dónde colocar el mecanismo**: la sede natural es el segmento T3 del framework 4-T (caso de referencia + cómo se logró), pero también puede vivir en T2 si es más natural ahí. Lo crítico es que esté.

**Ejemplos del mismo email con y sin mecanismo:**

❌ Sin mecanismo (falla — no se sabe qué venden):
> "trabajamos con un equipo de crédito con 17 modelos en prod — 3 semanas por ciclo bajaron a 4 días."

✅ Con mecanismo (sirve — se entiende qué hacen):
> "trabajamos con un equipo de crédito con 17 modelos en prod — **automatizando la preparación y orquestación de datos**, sus 3 semanas por ciclo bajaron a 4 días."

---

## FORMATO — INVIOLABLE (Email 1)

- **75–80 palabras máximo.** Debe caber en pantalla de teléfono sin scroll.
- Texto plano únicamente — sin HTML, imágenes, logos, adjuntos.
- Sin links en el cuerpo.
- Subject: 1–4 palabras en minúsculas — que parezca email interno, no marketing.
- Sin Title Case, sin emojis, sin signos de exclamación en el subject.

**Ejemplos de subject válidos:**
- "pipeline de datos"
- "equipo de data en [empresa]"
- "plomería de datos"

---

## TONO

- Casual y directo — como founder escribiéndole a un peer técnico
- "Tú" más que "nosotros"
- El champion de datos responde a alguien que entiende su problema, no a un vendedor
- Sonar como suposición educada, no como reporte de stalking
- **Empático, nunca presuntuoso.** Asume que el prospecto sabe lo que hace. La info de la ficha sirve para **entender** al cliente, no como **arma** para decirle que algo en su setup está mal
- **Frame de oportunidad, no de diagnóstico.** Presentamos lo que NOSOTROS hacemos bien, no lo que ELLOS hacen mal. El email debe sentirse como "te aviso de algo que puede aplicarte" — no como "encontré tus problemas y te los expongo"
- Hipótesis sobre patrones de la industria > diagnóstico específico del prospecto. "La mayoría de equipos de ML que conocemos todavía…" > "tu pipeline no está diseñado para…"

---

## CTA — Email 1

Siempre de interés, **nunca pedir reunión**: "¿Te haría sentido verlo?" o "¿Vale la pena platicarlo?". El objetivo del primer touch es abrir conversación, no agendar.

---

## FRAMEWORK 4-T

El Email 1 sigue esta estructura:

**T1 — Saludo + Reconocimiento del contacto (línea 1)**

Apertura cálida con saludo + reconocimiento de la **PERSONA** (puesto, trayectoria, expertise), **no de su empresa como entidad noticiosa**. Usa los datos del CONTACTO de la ficha (nombre, puesto, tiempo en el rol, responsabilidades reales). **Nunca uses datos de la EMPRESA fácilmente googleables** — funding rounds, anuncios públicos, cifras de press releases, nombres de programas con sus montos.

Patrón válido (saludo + reconocimiento natural de la persona):
- "Hola Pablo, viendo tu trayectoria liderando ML en Konfio…"
- "Hola Pablo, tu rol como Director of Machine Learning en una fintech con la escala de Konfio…"
- "Hola Pablo, vi que llevas varios años liderando datos en Konfio…"
- "Hola [Nombre], como [Puesto] en [Empresa]…"

**Tono — cálido pero NO adulador:**
- ✅ "tu trayectoria me parece interesante" / "viendo tu rol en…" / "como [Puesto] en…"
- ❌ "tu perfil es impresionante" / "eres un líder excepcional" / "admiro tu trabajo" — adulación = se siente falso, pierde autoridad

**Anti-patrones PROHIBIDOS (factoides de Google):**
- ❌ "El compromiso de Konfio para Plan México — $2.5B USD en 24 meses — es uno de los movimientos más ambiciosos…"
- ❌ "Vi el anuncio de la Serie B…" / "Konfio anunció X…"
- ❌ Cualquier apertura con $XYZ, % de crecimiento, número de PyMEs, cifras de press release
- ❌ Cualquier sentence sin saludo previo (empezar con un hecho descontextualizado)

**Por qué importa**: arrancar con un factoide de Google genera dos problemas. (1) El prospecto sabe que ese dato está disponible públicamente — no demuestra que lo conoces a él. (2) El factoide queda desconectado del resto del email, se siente copy-paste y se rompe el flow. Reconocer su rol/trayectoria sí lo personaliza; reconocer su comunicado de prensa no.

---

**T2 — Empatía vertical + Hipótesis de dolor (línea 2)**

Demuestra entendimiento de la **complejidad operacional** de su industria/empresa — sin citar datos de Google. La empatía vertical viene de CONOCER el vertical y su dolor, no de quote-droppear noticias.

Encadena la realidad de su vertical con la hipótesis del 80% manual.

Patrón válido:
- "En una empresa como Konfio, con múltiples líneas de crédito y la expansión que han logrado, el trabajo de preparación y limpieza de datos debe ser un reto importante en el día a día."
- "Liderar ML en una fintech con la complejidad operacional de Konfio implica que una buena parte del tiempo se va en limpiar y orquestar fuentes a mano antes de poder iterar modelos."
- "En equipos de [vertical] con la operación de [Empresa], muchos equipos todavía dedican una buena parte del tiempo a [pain del 80% manual]."

**Estructura**: "En una empresa/equipo como [X], con [característica de complejidad operacional sin cifra googleable], [el dolor del 80% manual] debe ser un reto importante…"

**Por qué importa**: la empatía vertical demuestra que entendemos la industria — no solo que la investigamos. Citar el comunicado de prensa demuestra solo lo segundo. Los buenos prospects distinguen.

**ANTI-PATRONES PROHIBIDOS:**
- ❌ "Tu pipeline no fue diseñado para…" (diagnóstico — ya cubierto en TONO)
- ❌ Cifra específica de Google ("con $2.5B USD en compromiso…") como sustituto de empatía vertical
- ❌ Frase desconectada que no encadena con el dolor del 80% manual

**T3 — Producto concreto + escala + industrias (líneas 4–5)**
Bloque concreto del producto: **operaciones específicas + fracción + tiempo + industrias**. Anclado en los datos puntuales de la sección "PRODUCTO — QUÉ HACE REASONING" arriba. **No inventar casos peer.** Ejemplo válido:

> "Nuestra plataforma automatiza limpieza, deduplicación, homologación y gobernanza — el 80% del pipeline manual — y baja flujos de semanas o meses a un par de días. Hoy lo usan equipos en fintech, manufactura y retail."

Reglas duras:
- **Verbos concretos** (limpieza, deduplicación, homologación, gobernanza), no abstracciones ("preparación", "orquestación").
- **Fracción real** (80% manual), no inventada.
- **Tiempo real** (semanas/meses → días), no inventado ("3 semanas a 4 días" = fabricación).
- **Industrias reales** (fintech, manufactura, retail), no peers anónimos con N modelos.
- Si el caso no se puede anclar a datos reales, omite el caso y deja el bloque concreto del producto. **Es preferible un T3 sin peer que un T3 con peer fabricado.**

**T4 — Talk (línea 5)**
CTA suave basado en interés, **enmarcado como oportunidad de explorar** — no como pregunta cuestionadora. Ejemplos válidos: "¿te hace sentido explorarlo?" / "¿vale la pena platicarlo?" / "si te interesa ver cómo aplica, te muestro". **Nunca**: "¿cómo vas a resolver eso?", "¿cómo tienes pensado cubrirlo?" — eso obliga al prospecto a justificarse, lo pone defensivo.

---

## PROHIBIDO — NUNCA USAR

**Saludos:**
- "Estimado/a" — usar el nombre directo siempre

**Dentro del cuerpo:**
- Bullet points
- Párrafos de más de 2 líneas
- Más de un tema por email

**Palabras y frases prohibidas:**
- "Solución integral"
- "Transformación digital"
- "Sinergia"
- "Espero que este correo te encuentre bien"
- "Me permito contactarte"
- "Inteligencia artificial" o "AI" — al champion no le importa la tecnología, le importa el resultado
- "Revolucionario"
- "Best-in-class"
- "Líder en"
- Porcentajes o cifras que no vengan de la ficha o de las estadísticas de respaldo

**Estructura:**
- No pedir reunión en el Email 1 (CTA = interés, no calendar)
- Enunciar resultados sin mecanismo ("lo bajamos a 4 días", "lo aceleramos 5x") sin decir CÓMO
- Implicar el producto en lugar de declararlo — "ayudamos con esto" no es enunciar el mecanismo

**Tono — diagnóstico al cliente (PROHIBIDO):**
- Decirle al prospecto que su pipeline/stack/setup actual está mal o no fue diseñado para X
- Asumir gaps específicos en su infraestructura basándose en research ("seguro que con Spark/Kafka tienen problemas con Y")
- Preguntarle cómo va a resolver algo, como si lo estuviéramos cuestionando ("¿cómo tienes pensado cubrirlo?")
- Cualquier frase que implique "yo sé más de tu negocio que tú" — superioridad o egocentrismo
- Usar la info de research como exhibición ("vi que…", "noté que…") en lugar de para informar el mensaje internamente

---

## ESTADÍSTICAS DE RESPALDO AUTORIZADAS
Solo usar estas — no inventar datos:

- 80% del tiempo de data science se va en preparar datos
- $285K USD/año por equipo de 5 analistas en trabajo manual reactivo
- Solo 29% de empleados con acceso real a BI (cifra estancada 7 años)
- 68% de empresas citan silos de datos como problema #1
- 40–60% de dashboards nunca se usan
- La confianza en datos cayó del 54% al 40% entre 2023 y 2025

---

## IDIOMA

- Español para prospectos de LATAM
- Inglés para prospectos de US
- Nunca mezclar en el mismo mensaje