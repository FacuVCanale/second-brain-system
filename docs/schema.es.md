---
type: meta
summary: "Schema del vault: tipos de nota, frontmatter y vocabulario cerrado. Contrato que el agente lee antes de crear/editar."
tags: [meta, schema]
---
# Schema del vault (`taxonomy`)

> Contrato para el agente (Claude Code) y para cualquier skill que escriba en el vault. **Leer esto antes de crear o editar notas.** Pensalo como el `schema.sql` del sistema.
>
> Todos los ejemplos usan un **elenco inventado** (nada real): personas *Ana Pérez* y *Beto Díaz*; empresas *Acme Corp* y *Globex*; proyecto-programa *Aurora* (módulos *Backend* / *Datos* / *App*) y proyecto simple *Faro*; modelos *orion* y *rotor*; dataset *atlas*.

## Reglas de oro
1. **Una entidad = una nota.** Antes de crear, buscá por título + `aliases`. Si existe, **actualizá (merge), no dupliques.**
2. **`summary` obligatorio** en toda nota (1-2 frases). Es lo que deja previsualizar sin abrir el archivo → controla el costo de contexto del agente.
3. Ante **contradicción** entre fuentes, marcala explícita en la nota; no sobrescribas en silencio.
4. **Relaciones por wikilink** en el frontmatter (`company: "[[Acme Corp]]"`). Las vistas salen de backlinks + Dataview, no de editar a mano.
5. **Inbox es crudo**: se procesa y se vacía. No borres la fuente hasta promover su contenido.
6. **Tres capas de conocimiento técnico.** Antes de escribir, hacé DOS preguntas **separadas**:
   **(a)** ¿La nota describe algo del mundo que existe independientemente del proyecto (*"qué es X"*), o es trabajo/estado/decisión propia (*"qué hicimos / cómo aplicamos X"*)?
   **(b)** Si es *"qué es X"*: ¿lo citaría **algún proyecto distinto del actual**, o **sólo éste**?

   | (a) naturaleza | (b) reuso | Destino |
   |---|---|---|
   | trabajo propio | — | **Cuaderno** → `Projects/<Proyecto>/<Módulo>/` con prefijo de módulo (`datos-*`); linkea a la referencia, no la copia |
   | referencia *"qué es X"* | **sólo este proyecto** | **Referencia mono-proyecto** → `Projects/<Proyecto>/Referencia/`, **sin prefijo**, tags por tema |
   | referencia *"qué es X"* | **2+ proyectos** | **Library global** → `Library/{Models,Papers,Datasets,Concepts,Standards}/`, **sin prefijo**, agnóstica de proyecto |

   **Regla de promoción** (igual que `Lessons → Conventions`): una referencia nace en `Projects/<Proyecto>/Referencia/`; el día que un 2º proyecto la cita, se **mueve** a `Library/`. No adivines el reuso futuro — empezá local y promové cuando se materialice. *Ej.: el modelo `rotor` vive en `Projects/Aurora/Referencia/Models/` (hoy sólo lo usa Aurora); `orion` y el dataset `atlas` están en `Library/` porque los citan Aurora **y** Faro.* El error a evitar: meter en `Library/` algo que falla (b) sólo porque "existe en el mundo" — eso es lo que la rompe.

## Tipos (`type`)
`person` · `company` · `project` · `meeting` · `note` · `model` · `concept` · `adr` · `lesson` · `resource` · `moc` · `meta`

## Vocabulario cerrado
- **status**: `active` · `dormido` · `cerrado` · `archived`
- **context**: `trabajo` · `estudio` · `vida`  (puede ser lista si cruza contextos)
- **company.relationship**: `cliente` · `partner` · `competidor` · `institucion` · `proveedor` · `infra` · `prospecto`
- **person.roles**: `cliente` · `colega` · `stakeholder` · `integrador` · `empleado` · `profesor` · `companero` · `inversor` · `contacto`
- **meeting.kind**: `reunion-laburo` · `grupo-tp` · `charla-evento` · `clase` · `personal-idea`

> El vocabulario es **cerrado** a propósito: no se inventan valores fuera de estas listas. Así las vistas Dataview quedan consistentes y filtables. Adaptá las listas a tu vida, pero mantenelas cerradas.

## Frontmatter por tipo

**person** — `type, aliases[], company "[[ ]]", roles[], projects[], context[], org, email, last-contact, summary, tags[person]`
**company** — `type, aliases[], website, industry, relationship[], people[], projects[], context[], status, summary, tags[company]`
**project** — `type, status, context, module(slug del planner), repos[], people[], companies[], decisions[], summary, tags[project,...]`
  · secciones fijas: `## Estado actual` · `## Cierre` · `## Aprendizajes` · `## Conocimiento (Brain)` · `## Tasks activas`
**adr** — `type, id, status, date, project "[[ ]]", supersedes, superseded-by, summary, tags[adr]`
  · secciones: `## Contexto` · `## Decisión` ("We will…") · `## Alternativas` · `## Consecuencias`
**meeting** — `type, kind, date, people[], project "[[ ]]", company "[[ ]]", context, summary, tags[meeting]`
**note** — `type, context, source, summary, tags[note]`
**lesson** — `type, date, rule, area, summary, tags[lesson]` (formato qué pasó→por qué→regla; vive en `Brain/Lessons/`)
**resource** — `type, kind(paper|dataset|snippet|link), url, topic, domain, summary, tags[resource]` (vive en `Library/Papers/` o `Library/Datasets/`)
**model** — `type, family(classic-ml|deep-nn|process-based|hybrid), learning(supervised|self-supervised|unsupervised|na), domain, aliases[], ref, summary, tags[model]` (modelo/arquitectura/algoritmo reutilizable; vive en `Library/Models/`)
**concept** — `type, domain, aliases[], ref, summary, tags[concept]` (concepto de dominio reutilizable; vive en `Library/Concepts/`)

## Naming de archivos
- **Entidades** (person/company/project): **título legible** → `Ana Pérez.md`, `Acme Corp.md`, `Aurora.md`.
- **Time-based** (meeting, captura): **prefijo ISO** → `2026-02-10 - aurora-kickoff.md`.
- **ADR / conceptos**: **kebab** → `Brain/Decisions/0001-formato-de-export.md`, `Notes/algun-concepto.md`.
- **Biblioteca y Referencia** (model/paper/dataset/concept/standard): **kebab por nombre canónico**, sin prefijo de proyecto → `Library/Models/orion.md`, `Library/Papers/<paper>.md`, o `Projects/<Proyecto>/Referencia/Models/rotor.md` si es mono-proyecto (regla 6). El prefijo de módulo se reserva para el **cuaderno** (`Projects/Aurora/Datos/datos-notas.md`).

## Estructura de un proyecto-programa (varios módulos)
Cuando un proyecto crece a varios módulos (ej. *Aurora*), sub-estructurá `Projects/<Proyecto>/`:
- el **hub** (`type: project` raíz) queda en la raíz de la carpeta (`Projects/Aurora/aurora.md`);
- una **carpeta por módulo** en PascalCase (`Backend/`, `Datos/`, `App/`), con su nota-módulo (`type: project`, su `module` slug) + el cuaderno con prefijo (`datos-*`);
- una carpeta **`Referencia/`** para la referencia mono-proyecto (regla 6).

> ⚠️ El extractor del planner camina `Projects/**` **recursivo** e identifica la nota por su **`basename` == `module` slug** (no por carpeta). Por eso **mover notas a subcarpetas es seguro** (no toca wikilinks, que en Obsidian resuelven por nombre, ni el mapeo del planner). **Renombrar el archivo NO**: rompe los `[[wikilinks]]` y el vínculo `module ↔ repo ↔ task`.

## ADR ≠ Lesson
- **ADR** (`Brain/Decisions/`): prospectivo e **inmutable** — "por qué elegimos X". Si cambia, se crea otro ADR y se marca `superseded-by`. No se edita.
- **Lesson** (`Brain/Lessons/`): retrospectivo — "qué aprendimos cuando algo pasó/salió mal". Se linkea al ADR, no lo reemplaza.

## Integración con el planner
- Las notas de `Projects/` son la **fuente de verdad del estado**. El planner (skills `logueador`/`capturador`/`archivador`) **escribe** `## Estado actual` / `## Cierre` / `## Aprendizajes` / `## Tasks activas`; el planner (`planner-diario`/`revisor-*`) **lee** `## Cierre` / `## Estado actual` vía el extractor.
- `module:` es el **slug** que vincula task ↔ nota-proyecto ↔ repo (en `state/task-module-map.json` y `state/repo-map.json` del planner). **No cambiarlo** sin actualizar esos mapeos.
- Links `Projects/ → Brain/` son **one-way**. Ninguna skill del planner edita `Brain/Conventions/`.
