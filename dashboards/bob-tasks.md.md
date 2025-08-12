---
assignee: bob
---

# My Tasks — bob

```dataview
TABLE WITHOUT ID title, status, due, project, file.link AS "Open"
FROM "tasks"
WHERE assignee = this.assignee AND status != "done"
SORT due ASC, status ASC, title ASC
```
