# reasoning-gtm

Sales intelligence agent para Reasoning Labs. Investiga leads, genera outreach refinado con un loop de 3 agentes, gestiona el expediente del cliente en HubSpot y prepara briefs pre-cotización.

## Prerequisites

- **Claude Code** instalado ([claude.com/claude-code](https://claude.com/claude-code))
- **`python3`** disponible en el PATH (lo usa `setup` para mergear `.mcp.json`, y `/gtm` para convertir CSV/XLSX)
- Cuentas de **HubSpot, Gmail, Google Calendar, Google Drive y Granola** — cada teammate usa las suyas. OAuth se hace en el browser la primera vez que se invoca cada servicio.

## Install

```bash
git clone https://github.com/GustavoCastaneda/reasoning-gtm.git ~/.claude/skills/reasoning-gtm
cd ~/.claude/skills/reasoning-gtm
./setup
```

Luego **reinicia Claude Code**. Al escribir `/` deben aparecer 7 comandos: `/gtm`, `/research`, `/create-company`, `/create-contact`, `/outreach`, `/post-llamada`, `/lead-brief`.

## First-time auth

La primera invocación de un skill que toca HubSpot/Gmail/Calendar/Drive/Granola abre un flujo de OAuth en el browser. Cada teammate autentica con su cuenta. Una sola vez por servicio.

## Usage

### Recommended — `/gtm` (lenguaje natural)

`/gtm` lee tu intención y encadena los skills correctos. Acepta tabla pegada, archivo CSV, archivo XLSX, o nombre suelto de empresa.

```
/gtm investiga este lead: <pegar tabla con empresa, contacto, puesto, work email, linkedin>
```

```
/gtm corre el flujo completo hasta el draft en mi gmail
[adjuntar leads.csv]
```

```
/gtm procesa la llamada de Financiera ABC | demo
```

```
/gtm brief para cotizar Distribuidora XYZ
```

### Granular skills

Si ya sabes exactamente qué quieres, los 6 skills granulares están disponibles directos:

| Skill | Cuándo usarlo |
|-------|---------------|
| `/gtm` | **Entry point recomendado.** Lenguaje natural — describe lo que quieres y orquesta los skills correctos. Acepta tabla pegada, CSV o XLSX |
| `/research` | Investiga un lead nuevo en 4 bloques (industria, empresa, competidores, contacto) y crea el Doc en Drive |
| `/create-company` | Crea la empresa en HubSpot y le adjunta la ficha como nota |
| `/create-contact` | Crea el contacto en HubSpot y lo vincula a su empresa |
| `/outreach` | Genera **el Email 1** con loop Writer → Critic → Judge (mínimo 8.5/10). Email 2/3 y LinkedIn se manejan después según evolucione la respuesta |
| `/post-llamada` | Procesa el transcript de una llamada y acumula el expediente del cliente |
| `/lead-brief` | Resumen ejecutivo de todo el contexto acumulado del lead antes de cotizar |

### Flujos típicos

**Cold outbound desde un Excel de leads:**
```
/gtm flujo completo hasta gmail
[adjuntar leads.xlsx]
```
→ Investiga → crea empresas en HubSpot → crea contactos → genera el Email 1 refinado → draft en Gmail.

**Después de una demo:**
```
/gtm procesa la llamada de [Empresa] | demo
```
→ Toma el transcript de Granola, lo agrega al Doc de Drive y actualiza el stage en HubSpot.

**Antes de mandar cotización:**
```
/gtm brief para [Empresa]
```
→ Devuelve un resumen ejecutivo con todo el contexto acumulado (HubSpot + Granola + Calendar).

## Update

```bash
cd ~/.claude/skills/reasoning-gtm && git pull && ./setup
```

Los skills son symlinks, así que `git pull` los actualiza solo. `./setup` solo es necesario si cambiaron los agentes o el `.mcp.json`.

## Agents (en `agents/`)

- `gtm-agent` — copiloto principal, conoce el producto, ICP, personas y pipeline
- `writer-agent` — escribe el draft del Email 1
- `data-validator-agent` — guardrail determinístico: verifica datos puntuales (operaciones concretas, 80%, semanas/meses → días, industrias) y bloquea fabricaciones
- `critic-agent` — revisa el draft y da recomendaciones
- `judge-agent` — califica del 1-10 simulando ser el prospecto (mínimo 8.5 para aprobar)

Los tres agentes de outreach (writer, critic, judge) los orquesta el skill `/outreach` automáticamente — no se invocan a mano.

## MCP servers

Definidos en `.mcp.json`. El setup los mergea con tu `~/.claude.json` global:

- **HubSpot** — CRM (empresas, contactos, notas, pipeline)
- **Granola** — transcripts de llamadas y demos
- **Google Calendar** — slots disponibles para proponer en outreach
- **Gmail** — drafts de outreach
- **Google Drive** — Docs de investigación en la carpeta "Leads activos"

## Troubleshooting

- **No aparecen los slash commands** → reinicia Claude Code después del setup
- **`/gtm` no encuentra el archivo CSV** → asegúrate que la ruta sea absoluta o que el archivo esté adjunto al mensaje
- **`/research` se queja de columnas faltantes** → verifica que el CSV/XLSX tenga las 5: `empresa, contacto, puesto, work email, linkedin`
- **HubSpot/Gmail piden auth cada vez** → el OAuth token expiró; vuelve a autorizar en el browser
- **`pandas` no instalado** (al usar XLSX) → `pip3 install pandas openpyxl` o exporta el archivo a CSV

## Repo layout

```
reasoning-gtm/
├── gtm/SKILL.md            ← orquestador (entry point)
├── research/SKILL.md
├── create-company/SKILL.md
├── create-contact/SKILL.md
├── outreach/{SKILL.md, references/}
├── post-llamada/SKILL.md
├── lead-brief/SKILL.md
├── agents/{gtm-agent.md, writer-agent.md, critic-agent.md, judge-agent.md}
├── .mcp.json
├── setup
├── CLAUDE.md
└── README.md
```
