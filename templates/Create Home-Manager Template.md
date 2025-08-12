<!-- File: templates/new-manager-home.md -->
<%*
const name = await tp.system.prompt("Manager Home title", "Manager Home — Team") || "Manager Home — Team";
const path = await tp.system.prompt("Save to (path/filename.md)", "dashboards/home-manager-team.md") || "dashboards/home-manager-team.md";
await tp.file.move(path);
_%>
# <% name %>

## 🔥 Today & Overdue (by assignee)
```dataview
TABLE WITHOUT ID assignee, title, due, status, project, file.link AS "Open"
FROM "tasks"
WHERE due AND date(due) <= date(today) AND status != "done"
GROUP BY assignee
SORT assignee ASC, date(due) ASC
```

## 🧪 Review Queue
```dataview
TABLE WITHOUT ID assignee, title, project, updated, file.link AS "Open"
FROM "tasks"
WHERE status = "review"
SORT assignee ASC, project ASC, date(updated) DESC
```

## 🧹 Needs Fixes (missing metadata)
```dataview
TABLE WITHOUT ID assignee, title, status, due, project, file.link AS "Open"
FROM "tasks"
WHERE !assignee OR !title OR !status OR !project
```

## 🕑 Recently Updated (last 24h)
```dataview
TABLE WITHOUT ID assignee, title, status, updated, file.link AS "Open"
FROM "tasks"
WHERE updated AND date(updated) >= date(now) - dur(24 hours)
SORT date(updated) DESC
```
