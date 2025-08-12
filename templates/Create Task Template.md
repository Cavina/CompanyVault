<%*
const title = await tp.system.prompt("Task title");
if (!title) { tR += "❌ No title given"; return; }

const assignee = await tp.system.prompt("Assignee folder (e.g., alice)");
if (!assignee) { tR += "❌ No assignee given"; return; }

const project = await tp.system.prompt("Project key/folder (optional)");
const due = await tp.system.prompt("Due date (YYYY-MM-DD, optional)");

// Timestamp ID: safe ISO-ish (no colons)
const nowId = tp.date.now("YYYY-MM-DDTHH-mm-ss");

// slugify → lowercase, no spaces, only [a-z0-9-]
function slugify(s){
  return s
    .normalize('NFKD').replace(/[\u0300-\u036f]/g, '') // strip accents
    .toLowerCase()
    .replace(/[^a-z0-9]+/g, '-')   // non-alnum -> hyphen
    .replace(/^-+|-+$/g, '')       // trim hyphens
    .replace(/-+/g, '-');          // collapse hyphens
}

const slug = slugify(title);
// Filename: 2025-08-11T16-45-23_infiltrate-the-ninja-compound.md
const base = `${nowId}_${slug}`;
const dir = `tasks/${assignee}`;

// Ensure uniqueness: append -2, -3, ...
let name = base, n = 2;
while (await app.vault.adapter.exists(`${dir}/${name}.md`)) {
  name = `${base}-${n++}`;
}

await tp.file.move(`${dir}/${name}.md`);
_%>
---
id: <% nowId %>
title: <% title %>
assignee: <% assignee %>
status: todo
due: <% (due || "") %>
project: <% (project || "") %>
tags: []
created: <% tp.date.now("YYYY-MM-DD") %>
updated: <% tp.date.now("YYYY-MM-DD") %>
---
# Context
- What does “done” mean?
- Links / references:

# Subtasks
- [ ] First step
- [ ] Second step
