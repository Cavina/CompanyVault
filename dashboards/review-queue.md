``` dataview
table assignee, title, project, file.link as "Open"
from "tasks"
where status = "review"
sort assignee asc, project asc
```
