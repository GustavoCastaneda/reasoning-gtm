---
description: >
  Investiga a fondo cada lead antes de generar outreach. Se activa cuando el
  usuario pega una tabla o lista de leads con columnas: empresa, contacto,
  puesto, work email, linkedin. Para cada lead investiga la industria, la
  empresa, sus competidores y el contacto. Al terminar crea un Google Doc por
  lead en la carpeta "Leads activos" en Drive. Máximo 10 leads por sesión.
---

# Research de Leads — $ARGUMENTS

Se te va a pegar una tabla con leads. Cada fila tiene: empresa, contacto, puesto, work email, linkedin.

Procesa cada lead en secuencia. Para cada uno ejecuta los 4 bloques de investigación usando web search y el perfil de LinkedIn. No pidas confirmación entre leads — corre todo de corrido y al final entrega el reporte completo.

---

## POR CADA LEAD, INVESTIGA ESTOS 4 BLOQUES

### 1. La industria
- ¿En qué industria opera la empresa?
- Tendencias relevantes del sector en los últimos 6 meses
- Regulaciones recientes que afecten su operación
- Retos comunes que enfrentan empresas de ese sector con sus datos

### 2. La empresa
- ¿Qué hacen exactamente? Producto o servicio principal
- Mercados donde operan (geografía, segmentos)
- Tamaño estimado (empleados, sucursales, presencia)
- Noticias recientes: expansión, nuevos productos, funding, cambios de liderazgo
- Señales de complejidad operacional: múltiples sucursales, líneas de negocio, volumen de datos

### 3. Los competidores
- ¿Quiénes son sus 2–3 competidores directos?
- ¿Alguno de esos competidores ya usa una solución de datos similar a Reasoning? Si encuentras evidencia, márcalo como **🔥 FOMO** — esto es oro para el outreach

### 4. El contacto
- Cargo específico y responsabilidades reales (más allá del título)
- Tiempo en el puesto actual
- Publicaciones recientes en LinkedIn — temas que le importan, proyectos que menciona
- Señales de que vive el dolor del pipeline manual: posts sobre datos, herramientas que usa, problemas que menciona

---

## OUTPUT POR CADA LEAD

Entrega esta ficha por cada lead investigado:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
LEAD: [Empresa] — [Contacto] | [Puesto]
Email: [work email] | LinkedIn: [url]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

INDUSTRIA
• Sector: [nombre]
• Tendencia relevante: [1–2 líneas]
• Reto común con datos: [1 línea]

EMPRESA
• Qué hacen: [1–2 líneas]
• Mercados: [geografía / segmentos]
• Tamaño estimado: [empleados / sucursales]
• Noticia reciente: [si existe]
• Señal de complejidad: [qué encontraste]

COMPETIDORES
• [Competidor 1] — [¿usa solución de datos? Sí/No/No encontrado]
• [Competidor 2] — [¿usa solución de datos? Sí/No/No encontrado]
• 🔥 FOMO: [si algún competidor ya usa algo similar, descríbelo aquí]

CONTACTO
• Responsabilidades reales: [1–2 líneas]
• Tiempo en el puesto: [X meses/años]
• Publicación reciente relevante: "[cita o resumen]"
• Señal de dolor: [si encontraste algo concreto]

TRIGGER PARA OUTREACH
• [1 línea con el dato más específico y accionable para personalizar el email]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Al final de todos los leads, entrega una tabla resumen:

| Empresa | Contacto | Trigger principal | FOMO |
|---------|----------|-------------------|------|
| [nombre] | [nombre] | [1 línea] | 🔥 / — |

---

## DESPUÉS DE ENTREGAR LAS FICHAS

Ejecuta estas dos acciones por cada lead sin pedir confirmación:

### 1. Crear Google Doc en Drive
- Crea un Doc con el contenido completo de la ficha
- Título del Doc: `[Empresa] — Lead Research [fecha]`
- Guárdalo en la carpeta **"Leads activos"** en Drive (si no existe, créala)

### 2. Pasar las fichas al siguiente skill
Las fichas quedan disponibles en el chat para que el siguiente skill (`create-company`) las consuma directamente. No hace falta que el usuario las copie — están en el contexto de la sesión.