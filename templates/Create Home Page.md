<%*
const whoInput = await tp.system.prompt("Assignee (e.g., alice, bob, manager)");
if (!whoInput) { tR = "❌ No assignee given"; return; }
const who = whoInput.trim().toLowerCase();
await tp.file.move(`dashboards/home-${who}.md`);
_%>
---
assignee: <%* tR += who %>
---

# My Home — <%* tR += who %>

## ▶ Focus (Today & Overdue)
```dataview
TABLE WITHOUT ID title, due, status, project, file.link AS "Open"
FROM "tasks"
WHERE assignee = this.assignee
AND due AND date(due) <= date(today)
AND status != "done"
SORT date(due) ASC, status ASC
```

## ⏭ Upcoming (Next 7 days)
```dataview
TABLE WITHOUT ID title, due, status, project, file.link AS "Open"
FROM "tasks"
WHERE assignee = this.assignee
AND due AND date(due) > date(today) AND date(due) <= date(today) + dur(7 days)
AND status != "done"
SORT date(due) ASC
```

## 📥 Backlog (No due date)
```dataview
TABLE WITHOUT ID title, status, project, created, file.link AS "Open"
FROM "tasks"
WHERE assignee = this.assignee
AND !due AND status != "done"
SORT created ASC
```

## ✅ Done (Last 7 days)
```dataview
TABLE WITHOUT ID title, project, updated, file.link AS "Open"
FROM "tasks"
WHERE assignee = this.assignee
AND status = "done"
AND updated AND date(updated) >= date(today) - dur(7 days)
SORT date(updated) DESC
```
