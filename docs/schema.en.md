---
type: meta
summary: "Vault schema: note types, frontmatter and closed vocabulary. The contract the agent reads before creating/editing a note."
tags: [meta, schema]
---
# Vault schema (`taxonomy`)

> Contract for the agent (Claude Code) and for any skill that writes to the vault. **Read this before creating or editing notes.** Think of it as the system's `schema.sql`.
>
> Every example uses an **invented cast** (nothing real): people *Ana Pérez* and *Beto Díaz*; companies *Acme Corp* and *Globex*; program-project *Aurora* (modules *Backend* / *Datos* / *App*) and simple project *Faro*; models *orion* and *rotor*; dataset *atlas*.
>
> Note on names: this vault is run in Spanish, so real identifiers stay literal — fixed section names like `## Estado actual`, planner skills like `capturador`, and the closed vocabulary (`trabajo`, `cliente`, …). They're glossed in English the first time.

## Golden rules
1. **One entity = one note.** Before creating, search by title + `aliases`. If it exists, **update (merge), don't duplicate.**
2. **`summary` is mandatory** on every note (1-2 sentences). It's what lets you preview without opening the file → it directly controls the agent's context cost.
3. On a **contradiction** between sources, flag it explicitly in the note; don't silently overwrite.
4. **Relationships via wikilink** in the frontmatter (`company: "[[Acme Corp]]"`). Views come from backlinks + Dataview, never hand-edited.
5. **The Inbox is raw**: it gets processed and emptied. Don't delete the source until its content is promoted.
6. **Three layers of technical knowledge.** Before writing, ask TWO **separate** questions:
   **(a)** Does the note describe something in the world that exists independently of the project (*"what X is"*), or is it your own work/state/decision (*"what we did / how we applied X"*)?
   **(b)** If it's *"what X is"*: would **some project other than the current one** cite it, or **only this one**?

   | (a) nature | (b) reuse | Destination |
   |---|---|---|
   | own work | — | **Notebook** → `Projects/<Project>/<Module>/` with module prefix (`datos-*`); links to the reference, doesn't copy it |
   | reference *"what X is"* | **only this project** | **Single-project reference** → `Projects/<Project>/Referencia/`, **no prefix**, tags by topic |
   | reference *"what X is"* | **2+ projects** | **Global Library** → `Library/{Models,Papers,Datasets,Concepts,Standards}/`, **no prefix**, project-agnostic |

   **Promotion rule** (same as `Lessons → Conventions`): a reference is born in `Projects/<Project>/Referencia/`; the day a 2nd project cites it, it **moves** to `Library/`. Don't guess future reuse — start local and promote when it materializes. *E.g.: the model `rotor` lives in `Projects/Aurora/Referencia/Models/` (today only Aurora uses it); `orion` and the dataset `atlas` are in `Library/` because both Aurora **and** Faro cite them.* The mistake to avoid: putting something in `Library/` that fails (b) just because "it exists in the world" — that's what breaks it.

## Types (`type`)
`person` · `company` · `project` · `meeting` · `note` · `model` · `concept` · `adr` · `lesson` · `resource` · `moc` · `meta`

## Closed vocabulary
- **status**: `active` · `dormido` (dormant) · `cerrado` (closed) · `archived`
- **context**: `trabajo` (work) · `estudio` (study) · `vida` (life) — can be a list if it crosses contexts
- **company.relationship**: `cliente` · `partner` · `competidor` · `institucion` · `proveedor` · `infra` · `prospecto`
- **person.roles**: `cliente` · `colega` · `stakeholder` · `integrador` · `empleado` · `profesor` · `companero` · `inversor` · `contacto`
- **meeting.kind**: `reunion-laburo` · `grupo-tp` · `charla-evento` · `clase` · `personal-idea`

> The vocabulary is **closed** on purpose: no values are invented outside these lists. That's what keeps Dataview views consistent and filterable. Adapt the lists to your life, but keep them closed.

## Frontmatter by type

**person** — `type, aliases[], company "[[ ]]", roles[], projects[], context[], org, email, last-contact, summary, tags[person]`
**company** — `type, aliases[], website, industry, relationship[], people[], projects[], context[], status, summary, tags[company]`
**project** — `type, status, context, module(planner slug), repos[], people[], companies[], decisions[], summary, tags[project,...]`
  · fixed sections: `## Estado actual` (current state) · `## Cierre` (closing) · `## Aprendizajes` (learnings) · `## Conocimiento (Brain)` (knowledge) · `## Tasks activas` (active tasks)
**adr** — `type, id, status, date, project "[[ ]]", supersedes, superseded-by, summary, tags[adr]`
  · sections: `## Contexto` · `## Decisión` ("We will…") · `## Alternativas` · `## Consecuencias`
**meeting** — `type, kind, date, people[], project "[[ ]]", company "[[ ]]", context, summary, tags[meeting]`
**note** — `type, context, source, summary, tags[note]`
**lesson** — `type, date, rule, area, summary, tags[lesson]` (format what happened→why→rule; lives in `Brain/Lessons/`)
**resource** — `type, kind(paper|dataset|snippet|link), url, topic, domain, summary, tags[resource]` (lives in `Library/Papers/` or `Library/Datasets/`)
**model** — `type, family(classic-ml|deep-nn|process-based|hybrid), learning(supervised|self-supervised|unsupervised|na), domain, aliases[], ref, summary, tags[model]` (reusable model/architecture/algorithm; lives in `Library/Models/`)
**concept** — `type, domain, aliases[], ref, summary, tags[concept]` (reusable domain concept; lives in `Library/Concepts/`)

## File naming
- **Entities** (person/company/project): **readable title** → `Ana Pérez.md`, `Acme Corp.md`, `Aurora.md`.
- **Time-based** (meeting, capture): **ISO prefix** → `2026-02-10 - aurora-kickoff.md`.
- **ADR / concepts**: **kebab** → `Brain/Decisions/0001-formato-de-export.md`, `Notes/some-concept.md`.
- **Library and Referencia** (model/paper/dataset/concept/standard): **kebab by canonical name**, no project prefix → `Library/Models/orion.md`, `Library/Papers/<paper>.md`, or `Projects/<Project>/Referencia/Models/rotor.md` if single-project (rule 6). The module prefix is reserved for the **notebook** (`Projects/Aurora/Datos/datos-notas.md`).

## Program-project structure (several modules)
When a project grows into several modules (e.g. *Aurora*), sub-structure `Projects/<Project>/`:
- the **hub** (`type: project` root) stays at the folder root (`Projects/Aurora/aurora.md`);
- one **folder per module** in PascalCase (`Backend/`, `Datos/`, `App/`), each with its module note (`type: project`, its `module` slug) + the notebook with prefix (`datos-*`);
- a **`Referencia/`** folder for single-project reference (rule 6).

> ⚠️ The planner's extractor walks `Projects/**` **recursively** and identifies the note by its **`basename` == `module` slug** (not by folder). That's why **moving notes into subfolders is safe** (it doesn't touch wikilinks, which resolve by name in Obsidian, nor the planner mapping). **Renaming the file is NOT**: it breaks the `[[wikilinks]]` and the `module ↔ repo ↔ task` link.

## ADR ≠ Lesson
- **ADR** (`Brain/Decisions/`): prospective and **immutable** — "why we chose X". If it changes, a new ADR is created and marked `superseded-by`. It isn't edited.
- **Lesson** (`Brain/Lessons/`): retrospective — "what we learned when something happened/went wrong". It links to the ADR, doesn't replace it.

## Planner integration
- `Projects/` notes are the **source of truth for state**. The planner (skills `logueador`/`capturador`/`archivador`) **writes** `## Estado actual` / `## Cierre` / `## Aprendizajes` / `## Tasks activas`; the planner (`planner-diario`/`revisor-*`) **reads** `## Cierre` / `## Estado actual` via the extractor.
- `module:` is the **slug** that links task ↔ project note ↔ repo (in the planner's `state/task-module-map.json` and `state/repo-map.json`). **Don't change it** without updating those mappings.
- Links `Projects/ → Brain/` are **one-way**. No planner skill edits `Brain/Conventions/`.
