# reasoning-gtm

Plugin de Claude Code para el proceso completo de ventas de Reasoning Labs. Investiga leads, genera outreach refinado con un loop de 3 agentes, gestiona el expediente del cliente, y prepara briefs pre-cotización.

---

## Instalación

```bash
# Local (desarrollo)
claude --plugin-dir ./reasoning-gtm

# Desde GitHub
/plugin install github.com/reasoning-labs/reasoning-gtm
```

---

## Flujo de ventas

```
1. research          → investiga el lead y crea el Doc en Drive
2. create-company    → crea la empresa en HubSpot
3. create-contact    → crea el contacto vinculado en HubSpot
4. outreach          → genera y refina la secuencia de emails (loop Writer → Critic → Judge)
5. post-demo         → procesa cada llamada y acumula el expediente
6. lead-brief        → resumen ejecutivo antes de cotizar
```

---

## Skills

### `research`
Investiga cada lead en 4 bloques: industria, empresa, competidores y contacto. Detecta señales de FOMO si un competidor ya usa una solución de datos similar. Crea un Google Doc por lead en la carpeta "Leads activos" en Drive.

```
Input: tabla pegada del Excel con columnas:
empresa | contacto | puesto | work email | linkedin

Máximo 10 leads por sesión.
```

### `create-company`
Crea la empresa en HubSpot con los datos de la ficha. Agrega la ficha completa como nota en el registro. Actualiza el Google Doc con el link de HubSpot.

```
Input: fichas del contexto de la sesión (después de research)
```

### `create-contact`
Crea el contacto en HubSpot y lo vincula a su empresa. Si el contacto llega sin empresa, pregunta antes de continuar.

```
Input: fichas del contexto de la sesión (después de create-company)
Regla: siempre requiere empresa + contacto
```

### `outreach`
Genera la secuencia completa de emails usando un loop de refinamiento entre 3 agentes. El email necesita mínimo 8.5/10 del Judge para llegar al chat. Máximo 3 iteraciones. Después del visto bueno crea los drafts en Gmail y actualiza HubSpot.

```
Input: nombre de empresa (toma la ficha del contexto)
Output: Email 1 + Email 2 + Break-up + LinkedIn
```

**Loop de refinamiento:**
```
Writer → draft + autocrítica
Critic → reporte + recomendaciones
Writer → reescribe
Judge  → califica (mínimo 8.5/10)
  └── < 8.5 y loops < 3 → regresa al Critic
  └── ≥ 8.5 o 3 iteraciones → muestra en chat para aprobación
```

### `post-demo`
Procesa el transcript de cualquier llamada en Granola y agrega una entrada al expediente acumulativo del cliente en HubSpot y Drive. Nunca reemplaza información existente — siempre acumula.

```
Input: [Empresa] | [propósito de la llamada]
Ejemplos:
  Financiera ABC | llamada de mapeo
  Distribuidora XYZ | demo
  Retail Norte | revisión de piloto
```

### `lead-brief`
Consolida todo el contexto acumulado del lead — HubSpot, Granola y Calendar — en un resumen ejecutivo para entrar preparado a la cotización.

```
Input: nombre de empresa
```

---

## Agentes

| Agente | Rol |
|--------|-----|
| `gtm-agent` | Agente principal — se activa automáticamente al cargar el plugin |
| `writer-agent` | Escribe el draft de outreach usando la ficha del lead |
| `critic-agent` | Revisa el draft y da recomendaciones concretas |
| `judge-agent` | Califica del 1-10 simulando ser el prospecto en su bandeja de entrada |

Los agentes `writer`, `critic` y `judge` son invocados automáticamente por el skill `outreach`. No se llaman manualmente.

---

## Integraciones (MCP)

| Servicio | Uso |
|----------|-----|
| HubSpot | CRM — empresas, contactos, notas, pipeline |
| Granola | Transcripts de llamadas y demos |
| Google Calendar | Reuniones próximas y slots disponibles |
| Gmail | Drafts de outreach |
| Google Drive | Docs de investigación en carpeta "Leads activos" |

---

## Estructura del repositorio

```
reasoning-gtm/
├── .claude-plugin/
│   └── plugin.json
├── agents/
│   ├── gtm-agent.md
│   ├── writer-agent.md
│   ├── critic-agent.md
│   └── judge-agent.md
├── skills/
│   ├── research/
│   │   └── SKILL.md
│   ├── create-company/
│   │   └── SKILL.md
│   ├── create-contact/
│   │   └── SKILL.md
│   ├── outreach/
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── outreach-rules.md
│   ├── post-demo/
│   │   └── SKILL.md
│   └── lead-brief/
│       └── SKILL.md
├── .mcp.json
└── README.md
```