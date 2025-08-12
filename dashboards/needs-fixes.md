```dataview
table assignee, title, status, due, project, file.link as "Open"
from "tasks"
where !assignee or !title or !status or !project
```

