# reasoning-gtm — project context

Sales intelligence toolkit para Reasoning Labs. Cubre el flujo completo de ventas del founder: investigar leads, crear empresa y contacto en HubSpot, generar outreach con un loop de refinamiento (Writer → Critic → Judge), procesar transcripts de llamadas y armar briefs pre-cotización.

## Layout

Skills viven como **carpetas en la raíz** del repo, cada una con un `SKILL.md` adentro. Los agentes viven en `agents/`. No hay `.claude-plugin/`, no hay `plugin.json`, no hay namespace `/reasoning-gtm:` — los slash commands son directos (`/research`, `/outreach`, etc.).

Instalación: `./setup` desde el repo. El script symlinkea las carpetas a `~/.claude/skills/`, copia los agentes a `~/.claude/agents/` y mergea `.mcp.json` con `~/.claude.json`.

## Skills

| Skill | Use when… |
|-------|-----------|
| `/gtm` | **Entry point recomendado.** Cuando quieras hacer algo de GTM en lenguaje natural — describe la intención + payload (tabla, CSV, xlsx, o nombre suelto) y orquesta los skills correctos. Ideal para no-power-users del equipo |
| `/research` | Llega un lead nuevo y hay que investigarlo antes de contactar |
| `/create-company` | Después de `research`, para crear la empresa en HubSpot |
| `/create-contact` | Después de `create-company`, para crear y vincular el contacto |
| `/outreach` | Hay que generar el Email 1 (loop Writer/Critic/Judge, mínimo 8.5/10). Solo el primer touch — no genera Email 2/3 ni LinkedIn |
| `/post-llamada` | Acaba de terminar una llamada (mapeo, demo, revisión de piloto) y hay que acumular el expediente |
| `/lead-brief` | Antes de una cotización o reunión importante — consolida HubSpot + Granola + Calendar |

## Agents

`gtm-agent` es el copiloto principal y se carga automáticamente al entrar al repo. Los otros tres (`writer-agent`, `critic-agent`, `judge-agent`) los orquesta el skill `/outreach` — no se invocan a mano.

## MCP servers

Definidos en `.mcp.json`:

- **HubSpot** — CRM (empresas, contactos, notas, pipeline)
- **Granola** — transcripts de llamadas y demos
- **Google Calendar** — slots disponibles para proponer en outreach
- **Gmail** — drafts de outreach
- **Google Drive** — Docs de investigación en la carpeta "Leads activos"

## Idioma

Responde en **español**. El usuario opera ventas en LATAM y trabaja en español. Para outreach: español a prospectos LATAM, inglés a prospectos US — nunca mezclar idiomas en el mismo mensaje.

## Editar este repo

- Skills: edita el `SKILL.md` dentro de cada carpeta de la raíz
- Agentes: edita los `.md` en `agents/`
- MCP servers: edita `.mcp.json` y vuelve a correr `./setup` para propagar el merge
- No agregues `.claude-plugin/`, `plugin.json` ni un manifiesto de marketplace — el formato es gstack-style skills, no plugin
