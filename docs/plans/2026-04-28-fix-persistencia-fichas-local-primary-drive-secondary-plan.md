---
title: "fix: persistencia de fichas — local primario, Drive secundario"
type: fix
status: completed
date: 2026-04-28
---

# fix: persistencia de fichas — local primario, Drive secundario

## Problema

`/research` depende exclusivamente de Google Drive como capa de persistencia. En la práctica, esto produce tres bugs combinados:

### Bug 1 — Google Docs quedan vacíos
El MCP de Drive crea el archivo con formato `application/vnd.google-apps.document`. La creación y la escritura de contenido son llamadas separadas. Si la escritura falla silenciosamente (timeout, permisos, cuota), el Doc existe pero está vacío. El skill no hace read-back de verificación, así que no detecta el fallo.

### Bug 2 — Archivos creados fuera de la carpeta correcta
El skill dice "guárdalo en la carpeta Leads activos (si no existe, créala)" pero no usa un folder ID hardcodeado. El MCP navega por nombre, y si no encuentra la carpeta o no soporta esa operación, el Doc cae en el root del Drive sin aviso.

### Bug 3 — Encoding corrompido (`â€¢` en lugar de `•`)
`â€¢` / `Ã³` / `â€™` son UTF-8 interpretado como Latin-1 — error clásico de doble encoding. Ocurre en el pipeline MCP → Google Docs → lectura posterior. Los Google Docs manejan internamente RTF/HTML; el texto plano con tildes y bullets se convierte a través de una capa intermedia que pierde el encoding.

### Bug 4 — Fallback implícito a archivos locales sin aviso
Cuando Drive falla, Claude usa `write_file` (creación de archivos locales) en lugar del MCP, sin informar al usuario. El archivo se crea en un path arbitrario fuera del repo, sin convención de nombres, sin registro de dónde quedó. En la siguiente sesión, el skill no sabe dónde buscar.

### Consecuencia sistémica
Si la ficha no persiste entre sesiones, los skills downstream (`/create-company`, `/create-contact`, `/outreach`) no tienen el contexto del lead. El usuario tiene que re-investigar o re-pegar la ficha manualmente cada vez que la sesión se reinicia.

---

## Solución propuesta

### Arquitectura: Local primario + Drive secundario

```
/research
  ├── [1] Genera ficha (investigación web + LinkedIn)
  ├── [2] Escribe ficha local en leads/[empresa-slug]/ficha.md  ← PRIMARIO (siempre funciona)
  ├── [3] Intenta subir a Drive como archivo .md (no Google Doc)  ← SECUNDARIO (no bloqueante)
  └── [4] Reporta resultado de ambas capas en el OUTPUT
```

**Por qué local primario:**
- `Read` de un archivo local nunca falla si el archivo existe
- Sin dependencia de MCP, red, permisos, ni cuota
- Los skills downstream pueden hacer `Read leads/empresa-slug/ficha.md` como fallback determinístico
- Git puede ignorar el directorio (datos sensibles del cliente)

**Por qué Drive secundario (no eliminarlo):**
- El equipo necesita compartir fichas entre máquinas
- Drive ya está integrado y funciona para lectura
- El fix es cambiar el formato (`.md` en lugar de Google Doc) para evitar los bugs de encoding

---

## Cambios por archivo

### 1. `research/SKILL.md`

**Sección "DESPUÉS DE ENTREGAR LAS FICHAS" — reemplazar completamente:**

```
### 1. Escribir ficha local (siempre, primero)

Escribe cada ficha en `leads/[empresa-slug]/ficha.md`:
- empresa-slug: nombre de la empresa en minúsculas, espacios → guiones,
  sin acentos ni caracteres especiales (ej: "Fibra Uno" → "fibra-uno")
- Contenido: el texto completo de la ficha, en UTF-8, con el formato
  de separadores ━━━ igual a como aparece en el OUTPUT
- Si el archivo ya existe (lead investigado antes), sobreescríbelo

Esta escritura es siempre exitosa. Captura el path local y agrégalo
a la sección GOOGLE DOC de la ficha como "Path local: leads/[slug]/ficha.md".

### 2. Subir a Drive como archivo .md (intento, no bloqueante)

Sube el archivo `leads/[empresa-slug]/ficha.md` a Drive:
- Formato: archivo de texto plano (.md), NO Google Doc
  (mimeType: text/plain — esto evita el encoding corrompido)
- Carpeta destino: "Leads activos" (busca por nombre; si no existe, créala)
- Nombre del archivo en Drive: `[Empresa] — Lead Research YYYY-MM-DD.md`
- Si la subida falla (error MCP, permisos, timeout): registra el fallo
  en el OUTPUT pero NO detengas el flujo. El archivo local ya es suficiente
  para continuar la sesión.
- Si la subida tiene éxito: captura el URL del archivo en Drive y agrégalo
  a la ficha en la sección GOOGLE DOC

### 3. Reportar resultado
```

**Sección OUTPUT — agregar columna de estado de persistencia:**

```
Al final de todos los leads, la tabla resumen incluye:

| Empresa | Contacto | Trigger | FOMO | Local | Drive |
|---------|----------|---------|------|-------|-------|
| ...     | ...      | ...     | 🔥   | ✅    | ✅ / ⚠️ fallback |
```

### 2. `outreach/SKILL.md` — Paso 1: leer ficha con fallback local

En el paso donde el skill lee la ficha de investigación, agregar:

```
Si la ficha no está en el contexto de la sesión actual, búscala en
`leads/[empresa-slug]/ficha.md`. El slug se deriva del nombre de la
empresa: minúsculas, espacios → guiones, sin acentos. Si no encuentras
el archivo, detente y pide al usuario que pegue la ficha o que corra
primero `/research`.
```

### 3. `create-company/SKILL.md` — mismo fallback

Mismo patrón: si el URL del Google Doc no está en contexto, buscar en
`leads/[empresa-slug]/ficha.md` la línea `Path local:` o `URL:`.

### 4. `create-contact/SKILL.md` — mismo fallback

Mismo patrón.

### 5. `.gitignore` — nuevo archivo

```
# Lead research files (sensitive client data)
leads/
```

---

## Por qué `.md` en Drive y no Google Doc

Google Docs (`application/vnd.google-apps.document`) tiene una capa de conversión:

```
Texto plano (UTF-8) → API de Drive → formato interno de Google Docs (RTF/JSON)
                                    ↑ aquí ocurre la pérdida de encoding
```

Subiendo como `text/plain` (`.md`), el contenido se guarda verbatim:

```
Texto plano (UTF-8) → API de Drive → archivo binario de bytes exactos
                                    ↑ sin conversión, sin encoding loss
```

El equipo puede abrir el `.md` en Drive (se renderiza como texto), en VS Code,
o en cualquier editor. Es más portátil que un Google Doc para este caso de uso.

---

## Acceptance Criteria

- [x] `/research` escribe `leads/[empresa-slug]/ficha.md` antes de intentar Drive
- [x] El archivo local existe y tiene contenido completo (sin encoding corrompido) después de correr `/research`
- [x] La subida a Drive usa formato `text/plain`, no `application/vnd.google-apps.document`
- [x] Si Drive falla, el flujo continúa y reporta el fallo en el OUTPUT sin detener al usuario
- [x] `/outreach` busca `leads/[slug]/ficha.md` si la ficha no está en contexto
- [x] `/create-company` y `/create-contact` hacen lo mismo
- [x] `leads/` está en `.gitignore`
- [x] El OUTPUT de `/research` muestra columna de estado de persistencia (Local ✅ / Drive ✅ o ⚠️)

---

## Riesgos

| Riesgo | Probabilidad | Mitigación |
|--------|-------------|------------|
| El MCP de Drive no soporta `text/plain` upload con contenido | Media | Si falla, el local ya es suficiente; Drive queda como "a futuro" |
| Dos instancias del equipo sobreescriben `leads/empresa/ficha.md` | Baja | gitignore evita conflictos en git; cada quien tiene su copia local |
| `empresa-slug` inconsistente entre skills | Media | Documentar la función de normalización explícita en cada skill: minúsculas, espacios→guiones, sin acentos |
| Ficha local no se encuentra porque el slug fue calculado diferente | Media | Mismo fix: normalización explícita y documentada |

---

## Contexto adicional

Esta arquitectura es temporal hasta que el equipo defina una solución de persistencia compartida más robusta (ej: HubSpot notes como fuente de verdad, o un backend propio). Por ahora, local + Drive es suficiente para el caso de uso de founder+equipo pequeño en la misma máquina o con Drive sincronizado.

## Sources

- `research/SKILL.md` — flujo actual de escritura en Drive
- `outreach/SKILL.md` — cómo consume la ficha hoy (contexto de sesión)
- `create-company/SKILL.md` — referencia al Google Doc URL
- `create-contact/SKILL.md` — misma referencia
