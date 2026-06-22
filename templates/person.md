<%* /* Persona (type: person). Entidad global: una sola nota aunque cruce contextos. */ -%>
---
type: person
aliases: []
company: "[[ ]]"
roles: []
projects: []
context: []
org: ""
email: ""
last-contact: <% tp.date.now("YYYY-MM-DD") %>
summary: ""
tags: [person]
---
# <% tp.file.title %>

## Notas
- 

## Interacciones
```dataview
TABLE date, kind, summary FROM "Meetings" WHERE contains(people, this.file.link) SORT date DESC
```
