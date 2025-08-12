```dataview
table without id title, status, due, project, file.link as "Open"
from "tasks/alice"
where assignee = "alice" and status != "done"
sort due asc, status asc, title asc
```

