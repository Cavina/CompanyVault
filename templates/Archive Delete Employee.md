<!-- File: templates/remove-employee.md -->
<%*
/* ── Prompt ─────────────────────────────────────────────────────────────── */
const whoInput = await tp.system.prompt("Assignee to archive (e.g., alice, bob, manager)");
if (!whoInput) { tR = "❌ No assignee given"; return; }
const who = whoInput.trim().toLowerCase();

/* ── Helpers ─────────────────────────────────────────────────────────────── */
function slugify(s){
  return s.normalize('NFKD').replace(/[\u0300-\u036f]/g,'')
    .toLowerCase().replace(/[^a-z0-9]+/g,'-').replace(/^-+|-+$/g,'').replace(/-+/g,'-');
}
async function ensureFolder(path){
  const parts = path.split('/'); let cur = '';
  for (const p of parts){
    cur = cur ? `${cur}/${p}` : p;
    if (!(await app.vault.adapter.exists(cur))) await app.vault.createFolder(cur);
  }
}
async function moveFileSafe(file, toPath){
  const targetDir = toPath.split('/').slice(0,-1).join('/');
  await ensureFolder(targetDir);
  // De-dup if needed
  let finalPath = toPath, n = 2;
  const base = toPath.replace(/\.md$/,'');
  const ext  = file.extension;
  while (await app.vault.adapter.exists(finalPath)){
    finalPath = `${base}-${n++}.${ext}`;
  }
  await app.vault.rename(file, finalPath);
  return finalPath;
}

/* ── Prep log destination; move THIS note to logs/ops ───────────────────── */
const tsId   = tp.date.now("YYYY-MM-DDTHH-mm-ss");
const tsDate = tp.date.now("YYYY-MM-DD");
const logDir = "logs/ops";
await ensureFolder(logDir);
const logName = `${tsId}_remove-employee-${slugify(who)}.md`;
const logPath = `${logDir}/${logName}`;
await tp.file.move(logPath);  // this note becomes the log

/* ── Archive targets ─────────────────────────────────────────────────────── */
const tasksPrefix = `tasks/${who}/`;
const dashPrefix  = `dashboards/`;
const dashArchive = `archive/dashboards/${who}`;
const tasksArchive= `archive/tasks/${who}`;
await ensureFolder("archive");
await ensureFolder("archive/tasks");
await ensureFolder("archive/dashboards");
await ensureFolder(tasksArchive);
await ensureFolder(dashArchive);

/* ── Collect files to move ───────────────────────────────────────────────── */
const files = app.vault.getFiles();

// 1) Tasks for this assignee (any extension)
const taskFiles = files.filter(f => f.path.startsWith(tasksPrefix));

// 2) Dashboards to archive:
//    - dashboards/home-<who>.md
//    - dashboards/my-tasks-<who>.md
//    - any dashboards/*.md with frontmatter assignee == <who>
const dashFiles = files.filter(f => f.path.startsWith(dashPrefix) && f.extension === "md");
const dashToMove = [];
for (const f of dashFiles){
  const name = f.name.toLowerCase();
  const fm = (app.metadataCache.getFileCache(f) || {}).frontmatter || {};
  const fmAssignee = String(fm.assignee || "").toLowerCase().trim();
  if (name === `home-${who}.md` || name === `my-tasks-${who}.md` || fmAssignee === who){
    dashToMove.push(f);
  }
}

/* ── Move files ──────────────────────────────────────────────────────────── */
const movedTasks = [];
for (const f of taskFiles){
  // Preserve any subpath under tasks/<who>/
  const rel = f.path.slice(tasksPrefix.length);
  const dest = `${tasksArchive}/${rel}`;
  const finalPath = await moveFileSafe(f, dest);
  movedTasks.push({ from: `/${tasksPrefix}${rel}`, to: `/${finalPath}` });
}

const movedDash = [];
for (const f of dashToMove){
  const dest = `${dashArchive}/${f.name}`;
  const finalPath = await moveFileSafe(f, dest);
  movedDash.push({ from: `/${f.path}`, to: `/${finalPath}` });
}

/* ── Clean up empty task folders for this assignee ───────────────────────── */
async function removeEmptyDirsRec(path){
  const listing = await app.vault.adapter.list(path).catch(() => null);
  if (!listing) return; // folder doesn't exist
  // Recurse into subfolders first
  for (const sub of listing.folders) {
    await removeEmptyDirsRec(sub);
  }
  // Re-check after recursion
  const after = await app.vault.adapter.list(path);
  if (after.files.length === 0 && after.folders.length === 0) {
    const tfolder = app.vault.getAbstractFileByPath(path);
    if (tfolder) {
      // true = bypass system trash; set to false if you prefer trash
      await app.vault.delete(tfolder, true);
    }
  }
}
await removeEmptyDirsRec(`tasks/${who}`);

/* ── Build log output ───────────────────────────────────────────────────── */
const movedTasksMD = movedTasks.length
  ? movedTasks.map(m => `- [[${m.to}|${m.to.split('/').pop()}]]  _(from \`${m.from}\`)`).join('\n')
  : "- (none)";

const movedDashMD = movedDash.length
  ? movedDash.map(m => `- [[${m.to}|${m.to.split('/').pop()}]]  _(from \`${m.from}\`)`).join('\n')
  : "- (none)";

const summary = `Archived **${who}**\n- Tasks moved: **${movedTasks.length}**\n- Dashboards moved: **${movedDash.length}**`;

tR = `---
type: log
action: remove-employee
assignee: ${who}
created: ${tsDate}
tasks_moved: ${movedTasks.length}
dashboards_moved: ${movedDash.length}
---

# Remove Employee — ${who}

${summary}

## Tasks archived → \`/${tasksArchive}/\`
${movedTasksMD}

## Dashboards archived → \`/${dashArchive}/\`
${movedDashMD}

### Notes
- Team dashboards will no longer surface ${who}'s tasks because they now live under \`/archive/tasks/${who}/\`.
- If you need to restore, move files back to \`/tasks/${who}/\` and (optionally) \`/dashboards/\`.
`;
_%>
