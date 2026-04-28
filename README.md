# reasoning-gtm

Sales intelligence agent para Reasoning Labs. Investiga leads, genera outreach refinado con un loop de 3 agentes, gestiona el expediente del cliente en HubSpot y prepara briefs pre-cotización.

## Install

```bash
git clone https://github.com/GustavoCastaneda/reasoning-gtm.git ~/.claude/skills/reasoning-gtm && cd ~/.claude/skills/reasoning-gtm && ./setup
```

## Update

```bash
cd ~/.claude/skills/reasoning-gtm && git pull && ./setup
```

## What gets installed

**Slash commands** (skills):

| Skill | Cuándo usarlo |
|-------|---------------|
| `/gtm` | **Entry point recomendado.** Lenguaje natural — describe lo que quieres y orquesta los skills correctos. Acepta tabla pegada, CSV o XLSX |
| `/research` | Investiga un lead nuevo en 4 bloques (industria, empresa, competidores, contacto) y crea el Doc en Drive |
| `/create-company` | Crea la empresa en HubSpot y le adjunta la ficha como nota |
| `/create-contact` | Crea el contacto en HubSpot y lo vincula a su empresa |
| `/outreach` | Genera la secuencia de emails (Email 1 + Email 2 + break-up + LinkedIn) con loop Writer → Critic → Judge |
| `/post-llamada` | Procesa el transcript de una llamada y acumula el expediente del cliente |
| `/lead-brief` | Resumen ejecutivo de todo el contexto acumulado del lead antes de cotizar |

**Agents** (en `agents/`):

- `gtm-agent` — copiloto principal, conoce el producto, ICP, personas y pipeline
- `writer-agent` — escribe el primer draft del outreach
- `critic-agent` — revisa el draft y da recomendaciones
- `judge-agent` — califica del 1-10 simulando ser el prospecto (mínimo 8.5 para aprobar)

**MCP servers** — el setup hace merge de `.mcp.json` con tu `~/.claude.json` global. Servicios configurados: HubSpot, Granola, Google Calendar, Gmail y Google Drive.

## Requirements

- Claude Code
- `python3` (lo usa `setup` para hacer merge de `.mcp.json`)
- Credenciales de los MCP servers — `.mcp.json` contiene los URLs de cada uno; la autenticación se completa en Claude Code la primera vez que se invoca cada servicio

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
