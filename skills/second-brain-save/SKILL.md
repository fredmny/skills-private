---
name: second-brain-save
description: >
  Save something into Fred's Obsidian vault (his "second brain", ~1850 notes) — a new note or
  an addition to an existing one. Use when he says "save this", "remember this", "add this to
  my notes/vault", "write this up", "capture this" — and also mid-task when something worth
  keeping was just learned or decided and he asks for it to be kept. Always asks before
  writing anywhere outside the agent scratch folder. For looking something up instead, use
  second-brain-recall.
---

# Second brain — save

The write path into Fred's Obsidian vault. **This skill carries no instructions of its own
beyond getting you to the vault** — the recipes live in the vault so all agent tools stay in
sync. Resolve, read, then follow.

## The rule that governs everything here

**Outside `ai-agent/docs/`, ask Fred before every single write. Every time, not just when
unsure.** This is the core decision of the whole design, not a safety default to be optimized
away. The vault is not a git repo and there is no verified backup — an unapproved write is
not cheaply undoable.

## 1. Resolve the vault and confirm Obsidian is running

The Obsidian CLI only works while the Obsidian **app** is running, but it works from anywhere
— **cwd is irrelevant, so never `cd` to make it work.**

```bash
out=$(obsidian vault=obsidian_personal vault 2>&1)
VAULT=$(printf '%s\n' "$out" | awk -F'\t' '/^path\t/{print $2}')
if [ -z "$VAULT" ] || [ ! -d "$VAULT/.obsidian" ]; then
    echo "Obsidian isn't reachable — open it and I'll retry. (CLI said: ${out%%$'\n'*})"
    exit 1
fi
echo "$VAULT"
```

**Validate positively; never match on error strings.** The CLI returns rc=0 on failure and
prints errors to stdout, and the strings vary — a bad vault name yields a bare
`Vault not found.` Requiring a real `path` line *and* a real `.obsidian` directory is the only
guard that holds for failure modes nobody has seen yet.

**If that fails: tell Fred to open Obsidian, and stop.** Never fall back to writing files
directly to answer anyway — a write that bypasses the CLI bypasses the index and livesync.

Always pin `vault=obsidian_personal` on every command.

## 2. Read the recipe

New note, or not sure yet:

```bash
obsidian vault=obsidian_personal read path="ai-agent/workflows/capture.md"
```

Changing a note that already exists:

```bash
obsidian vault=obsidian_personal read path="ai-agent/workflows/update.md"
```

Also read, before placing or formatting anything:

```bash
obsidian vault=obsidian_personal read path="ai-agent/conventions.md"   # §4 frontmatter contract, §5 markers
obsidian vault=obsidian_personal read path="ai-agent/vault-map.md"     # which folder, and why
```

If you have not used this CLI in the session, read
`ai-agent/references/obsidian-cli.md` first — **it fails silently: rc=0 on error, errors on
stdout, wrong flags ignored rather than rejected.**

Read these by path. `ai-agent/` is excluded from Obsidian's search index, so `obsidian search`
will not find them.

## 3. Follow it

`capture.md` is an ordered recipe: draft freely into `ai-agent/docs/` → dedup check (enumerate
the drafts folder **and** run a bilingual search of the real vault) → pick type and folder →
**ask Fred** → create, mark, link in, log, delete the draft → re-verify link debt.

Four things from it that decide whether the write is right:

- **Dedup before you propose.** A 1851-note vault does not need a second note on the same
  topic. And the bilingual gate is mandatory — `prayer` returns 0 results while `oração`
  returns 23, so an empty English search is not evidence the vault lacks the topic.
- **Go where the topic's neighbours already are**, not where the category name says. A Linux
  how-to belongs in `resources/` with the rest of the Linux cluster, not in `Instructions/`.
- **Only five types may be agent-created:** `reference`, `cheatsheet`, `tool`, `article`,
  `thoughts`. Never invent a `type:` value the vault does not already use.
- **Measure `unresolved total` before the write and re-check it after** — must come back
  equal. Fix only the delta you broke; do not touch Fred's pre-existing backlog (some of it is
  daily-note date navigation that must never be "fixed"). Do not rely on a hardcoded count —
  it drifts as Fred edits his own notes.

## 4. Report back

Say what was written where, which note now links to it, which markers were set, and the
`unresolved` count. If Fred declined, say where the draft is sitting in `ai-agent/docs/` so it
doesn't silently vanish from the next session's context.

## Blocking rules — no judgment calls

- **Never write secrets or client-confidential data (PII or otherwise) into the vault.**
  `BranchingMinds/`, `Trustly/`, `Loft/` are where this bites.
- **Never read or copy from `Templates/`** — unresolved Templater syntax, and
  `tp.system.prompt()` would pop a modal in Fred's live app.
- **Never `mv` or `git mv` a note** — it silently breaks every wikilink. Use `obsidian rename`
  / `obsidian move`, and ask first: a rename changes vault structure.
- **Never `delete ... permanent`.** Trash only.
- **Never touch git.** The vault is not a git repo; there is nothing there to touch.
- **Never set `ai_*` markers on drafts in `ai-agent/docs/`** — only on real vault notes.
- **Unattended/scheduled invocation is unsupported.** The whole design rests on stopping to
  ask a human. If nobody can answer, stop.
