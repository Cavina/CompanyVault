<!-- File: templates/new-employee.md -->
<%*
/* ── Prompts ─────────────────────────────────────────────────────────────── */
const whoInput = await tp.system.prompt("Assignee handle (e.g., alice, bob, manager)");
if (!whoInput) { tR = "❌ No assignee given"; return; }
const who = whoInput.trim().toLowerCase();
const alsoList = (await tp.system.prompt("Also create simple 'my-tasks' list? (y/N)", "N") || "N").toLowerCase().startsWith("y");

/* ── Helpers ─────────────────────────────────────────────────────────────── */
function slugify(s){
  return s.normalize('NFKD').replace(/[\u0300-\u036f]/g,'')
    .toLowerCase().replace(/[^a-z0-9]+/g,'-').replace(/^-+|-+$/g,'').replace(/-+/g,'-');
}
const tsId   = tp.date.now("YYYY-MM-DDTHH-mm-ss");
const tsDate = tp.date.now("YYYY-MM-DD");
const logTitle = `New Employee — ${who}`;
const logName  = `${tsId}_${slugify(logTitle)}.md`;
const logDir   = `logs/ops`;
const logPath  = `${logDir}/${logName}`;

/* ── Ensure log folder exists, then MOVE THIS NOTE there ─────────────────── */
if (!(await app.vault.adapter.exists("logs")))     await app.vault.createFolder("logs");
if (!(await app.vault.adapter.exists(logDir)))     await app.vault.createFolder(logDir);
await tp.file.move(logPath); // this note becomes the log at logs/ops/...

/* ── Side effects: create folders/files for the employee ─────────────────── */
const tasksDir  = `tasks/${who}`;
const dashesDir = `dashboards`;
if (!(await app.vault.adapter.exists(tasksDir)))  await app.vault.createFolder(tasksDir);
if (!(await app.vault.adapter.exists(dashesDir))) await app.vault.createFolder(dashesDir);

/* Home dashboard */
const F = "```";
const homePath = `${dashesDir}/home-${who}.md`;
const homeBody =
`---
assignee: ${who}
aliases: ["My Tasks — ${who}", "my-tasks-${who}"]
---

# My Home — ${who}

## ▶ Focus (Today & Overdue)
${F}dataview
TABLE WITHOUT ID title, due, status, project, file.link AS "Open"
FROM "tasks"
WHERE assignee = this.assignee
AND due AND date(due) <= date(today)
AND status != "done"
SORT date(due) ASC, status ASC
${F}

## ⏭ Upcoming (Next 7 days)
${F}dataview
TABLE WITHOUT ID title, due, status, project, file.link AS "Open"
FROM "tasks"
WHERE assignee = this.assignee
AND due AND date(due) > date(today) AND date(due) <= date(today) + dur(7 days)
AND status != "done"
SORT date(due) ASC
${F}

## 📥 Backlog (No due date)
${F}dataview
TABLE WITHOUT ID title, status, project, created, file.link AS "Open"
FROM "tasks"
WHERE assignee = this.assignee
AND !due AND status != "done"
SORT created ASC
${F}

## ✅ Done (Last 7 days)
${F}dataview
TABLE WITHOUT ID title, project, updated, file.link AS "Open"
FROM "tasks"
WHERE assignee = this.assignee
AND status = "done"
AND updated AND date(updated) >= date(today) - dur(7 days)
SORT date(updated) DESC
${F}
`;
if (await app.vault.adapter.exists(homePath)) {
  await app.vault.modify(await app.vault.getAbstractFileByPath(homePath), homeBody);
} else {
  await app.vault.create(homePath, homeBody);
}

/* Optional simple list */
let listPath = "";
if (alsoList) {
  listPath = `${dashesDir}/my-tasks-${who}.md`;
  const listBody =
`${F}dataview
TABLE WITHOUT ID title, status, due, project, file.link AS "Open"
FROM "tasks"
WHERE assignee = "${who}" AND status != "done"
SORT due ASC, status ASC, title ASC
${F}
`;
  if (await app.vault.adapter.exists(listPath)) {
    await app.vault.modify(await app.vault.getAbstractFileByPath(listPath), listBody);
  } else {
    await app.vault.create(listPath, listBody);
  }
}

/* Build a conditional line for the log */
const listLine = alsoList ? `- List: [[${listPath}]]` : "";
_%>
---
type: log
action: new-employee
assignee: <% who %>
created: <% tsDate %>
---

# <% logTitle %>

**When:** <% tsDate %>  

**Created:**
- Folder: `<% tasksDir %>/`
- Home: [[<% homePath %>]]
<% listLine %>

**Notes:**  
- Add initial tasks for **<% who %>** in `<% tasksDir %>/`.  
- Share [[<% homePath %>]] with the employee.
