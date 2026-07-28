---
type: dashboard
---

# 🏆 **House Cup Dashboard**

```dataview
TABLE Casa, Puntos
FROM "Session Logs"
```

```dataviewjs
// Get all pages in the folder
let pages = dv.pages('"Session Logs"');

// Map house to total points
let groups = new Map();

for (let page of pages) {
    // check if the page has a table called "Casa"
    if (!page.file || !page.file.frontmatter) continue;
    
    // Get table rows
    let rows = dv.io.parse(page.file.path).rows || [];
    for (let row of rows) {
        let casa = row.Casa;
        if (!casa) continue;
        let puntos = parseInt(row.Puntos) || 0;
        groups.set(casa, (groups.get(casa) || 0) + puntos);
    }
}

// Convert Map to array
let data = Array.from(groups, ([house, points]) => ({house, points}))
                .sort((a,b)=> b.points - a.points);

if (data.length === 0) {
    dv.paragraph("⚠️ No hay datos aún. Asegúrate de que las tablas tengan columnas `Casa` y `Puntos`");
} else {
    dv.header(2, "🏆 Ranking de Casas");
    for (let [i, h] of data.entries()) {
        dv.paragraph(`${i+1}. **${h.house}** — ${h.points} puntos`);
    }
}



```
