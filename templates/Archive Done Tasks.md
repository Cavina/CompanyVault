<!-- File: templates/archive-done.md -->
<%*
/* ── Prompts ─────────────────────────────────────────────────────────────── */
let daysStr = await tp.system.prompt("Archive DONE tasks older than how many days?", "30");
let days = parseInt(daysStr, 10);
if (!Number.isFinite(days) || days <= 0) days = 30;

/* ── Helpers ─────────────────────────────────────────────────────────────── */
function slugify(s){
  return s.normalize('NFKD').replace(/[\u0300-\u036f]/g,'')
    .toLowerCase().replace(/[^a-z0-9]+/g,'-').replace(/^-+|-+$/g,'').replace(/-+/g,'-');
}
function parseYMD(s){
  if (!s) return null;
  const m = String(s).trim().match(/^(\d{4})-(\d{2})-(\d{2})$/);
  if (!m) return null;
  return new Date(`${m[1]}-${m[2]}-${m[3]}T00:00:00`);
}
function ymd(d){ return d.toISOString().slice(0,10); }

async function ensureFolder(path){
  const parts = path.split('/'); let cur = '';
  for (const p of parts){
    cur = cur ? `${cur}/${p}` : p;
    if (!(await app.vault.adapter.exists(cur))) await app.vault.createFolder(cur);
  }
}

async function uniqueTarget(dir, baseName, ext){
  let name = `${baseName}.${ext}`, n = 2;
  while (await app.vault.adapter.exists(`${dir}/${name}`)){
    name = `${baseName}-${n++}.${ext}`;
  }
  return `${dir}/${name}`;
}

/* ── Compute cutoff & log paths ──────────────────────────────────────────── */
const now     = new Date();
const cutoff  = new Date(now.getTime() - days*24*60*60*1000);
const cutoffY = ymd(cutoff);

const tsId    = tp.date.now("YYYY-MM-DDTHH-mm-ss");
const logDir  = "logs/ops";
const logName = `${tsId}_archive-done-older-than-${days}-days.md`;
await ensureFolder(logDir);

/* Move THIS new note to logs/ops and make it the run log */
await tp.file.move(`${logDir}/${logName}`);

/* ── Scan tasks and move eligible files ──────────────────────────────────── */
const all = app.vault.getFiles().filter(f => f.path.startsWith("tasks/") && f.extension === "md");

const moved = [];              // { from, to, assignee }
const skipped = [];            // { path, reason }
const perAssignee = {};        // counts

for (const f of all){
  const cache = app.metadataCache.getFileCache(f) || {};
  const fm = cache.frontmatter || {};
  const status = String(fm.status || "").toLowerCase();

  if (status !== "done"){ skipped.push({path:f.path, reason:"status not done"}); continue; }

  // choose reference date: updated → due → file.mtime
  let refDate = parseYMD(fm.updated) || parseYMD(fm.due) || new Date(f.stat.mtime);
  if (!(refDate instanceof Date) || isNaN(refDate)) { skipped.push({path:f.path, reason:"no valid date"}); continue; }

  if (refDate > cutoff){ skipped.push({path:f.path, reason:`newer than cutoff (${cutoffY})`}); continue; }

  // assignee from fm or path
  let assignee = String(fm.assignee || "").toLowerCase().trim();
  if (!assignee){
    const m = f.path.match(/^tasks\/([^\/]+)/);
    assignee = m ? m[1] : "unknown";
  }

  const archiveDir = `archive/tasks/${assignee}`;
  await ensureFolder("archive");
  await ensureFolder("archive/tasks");
  await ensureFolder(archiveDir);

  const base = f.basename;  // filename without extension
  const toPath = await uniqueTarget(archiveDir, base, f.extension);

  await app.vault.rename(f, toPath);

  moved.push({ from: f.path, to: toPath, assignee });
  perAssignee[assignee] = (perAssignee[assignee] || 0) + 1;
}

/* ── Build summary strings for the log body ──────────────────────────────── */
const movedTotal = moved.length;
const skippedTotal = skipped.length;

let perAssigneeLines = Object.keys(perAssignee).sort().map(a => `- **${a}**: ${perAssignee[a]}`).join('\n');
if (!perAssigneeLines) perAssigneeLines = "- (none)";

let movedList = moved.map(m => `- [[${m.to}|${m.to.split('/').pop()}]] (from \`${m.from}\`)`).join('\n');
if (!movedList) movedList = "- (none)";

let skippedList = skipped.slice(0, 50).map(s => `- \`${s.path}\` — ${s.reason}`).join('\n');
if (!skippedList) skippedList = "- (none)";
_%>
---
type: log
action: archive-done
cutoff_days: <% days %>
cutoff_date: <% cutoffY %>
moved_total: <% movedTotal %>
skipped_total: <% skippedTotal %>
created: <% tp.date.now("YYYY-MM-DD") %>
---

# Archive Done — Older than <% days %> days

**Cutoff date:** <% cutoffY %>  
**Moved:** <% movedTotal %> files  
**Skipped:** <% skippedTotal %> files

## By assignee
<% perAssigneeLines %>

## Moved files
<% movedList %>

## Skipped (first 50)
<% skippedList %>
