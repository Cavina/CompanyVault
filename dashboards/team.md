``` dataview
table assignee, status, due, project, file.link as "Open"
from "tasks"
where status != "done"
sort assignee asc, due asc
```
