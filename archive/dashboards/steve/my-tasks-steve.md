```dataview
TABLE WITHOUT ID title, status, due, project, file.link AS "Open"
FROM "tasks"
WHERE assignee = "steve" AND status != "done"
SORT due ASC, status ASC, title ASC
```
