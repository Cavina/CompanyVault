``` dataview
table assignee, title, due, status, file.link as "Open"
from "tasks"
where due and date(due) <= date(now) + dur(14 days) and status != "done"
sort date(due) asc
```

