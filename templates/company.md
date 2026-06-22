<%* /* Empresa / institución (type: company). Entidad global. */ -%>
---
type: company
aliases: []
website: ""
industry: ""
relationship: []
people: []
projects: []
context: []
status: active
summary: ""
tags: [company]
---
# <% tp.file.title %>

## Contexto
- 

## Personas
```dataview
LIST FROM "People" WHERE contains(company, this.file.link)
```

## Proyectos
```dataview
LIST FROM "Projects" WHERE contains(companies, this.file.link)
```
