<%*
const who = await tp.system.prompt("Assignee (folder name, e.g., alice)");
if (!who) { tR += "❌ No assignee given"; return; }
const safe = who.toLowerCase().trim();
await tp.file.move(`dashboards/${safe}-tasks.md`);
_%>
---
assignee: <%* tR += safe %>
---

# My Tasks — <%* tR += safe %>

```dataview
TABLE WITHOUT ID title, status, due, project, file.link AS "Open"
FROM "tasks"
WHERE assignee = this.assignee AND status != "done"
SORT due ASC, status ASC, title ASC
```
