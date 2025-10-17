---
Casa: <% tp.system.prompt("¿Qué casa?") %>
Evento: <% tp.system.prompt("¿Qué evento o razón?") %>
Puntos: <% tp.system.prompt("¿Cuántos puntos? (+ o -)") %>
---

| Casa | Evento | Puntos | Notas |
|-------|---------|---------|-------|
| <% tp.file.cursor() %> |  |  |  |
