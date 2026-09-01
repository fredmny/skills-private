---
name: second-brain-setup
description: >
  Bootstrap or verify this machine's local configuration for the agentic second brain — the
  Obsidian CLI, vault resolution, the ai-agent excluded-files entry, the three ai_* property
  types, the CLAUDE.md symlink, and $OBSIDIAN_VAULT. Use when setting up a new machine for
  second-brain-recall/second-brain-save, when either of those fails with a vault-resolution
  or property error, or when Fred says "set up my vault here" / "bootstrap obsidian" / "why
  doesn't this work on this machine". Fixes only local machine config, never vault content
  beyond a throwaway scratch note it deletes itself.
---

# Second brain — setup

One-time (or re-run-safe) bootstrap for a new machine. Unlike `second-brain-recall` and
`second-brain-save`, not all of this skill's logic can live in a vault-resident doc — some of
what it checks is the very thing that makes reading that doc possible. That's why the four
preconditions below live here, in the skill, and everything else lives in
`ai-agent/workflows/setup.md`.

## 1. Preconditions — stop here if any of these fails

```bash
command -v obsidian >/dev/null || { echo "obsidian CLI not on PATH — install it, then re-run this skill."; exit 1; }
```

```bash
out=$(obsidian vault=obsidian_personal vault 2>&1)
VAULT=$(printf '%s\n' "$out" | awk -F'\t' '/^path\t/{print $2}')
if [ -z "$VAULT" ] || [ ! -d "$VAULT/.obsidian" ]; then
    echo "Obsidian isn't reachable — open it and I'll retry. (CLI said: ${out%%$'\n'*})"
    exit 1
fi
```

**Validate positively, never on error strings** — same rule as every other skill here; `rc=0`
on failure, and the failure strings vary.

```bash
obsidian vault=obsidian_personal files folder="ai-agent" total
```

**Assert a plausible floor, not just non-zero** — per this project's "wrong flag is silence,
not an error" rule, confirm the output is a bare integer before trusting it (`total` alone,
not e.g. a typo'd `totals`, which silently returns the full file listing instead). `ai-agent/`
holds at least 12 files as of 2026-08-31 (README, vault-map, conventions, 4 workflow docs, 2
references, state, docs) — treat anything under ~10 as suspect, and anything non-numeric as a
failed check, not a pass.

If it's `0` (or clearly wrong): **stop and tell Fred.** `ai-agent/` is vault *content* — it
travels via livesync like any other note — so if it isn't here yet, sync hasn't landed it, and
that isn't something this skill can fix by creating the directory itself.

All three passed? You now have `$VAULT` (the real filesystem path) and confirmed `ai-agent/`
exists. Proceed.

## 2. Read the recipe

```bash
obsidian vault=obsidian_personal read path="ai-agent/workflows/setup.md"
```

That doc has the fix procedures for the rest: the excluded-files entry, the three `ai_*`
property types, the `CLAUDE.md` symlink, and `$OBSIDIAN_VAULT`. Follow it in order — each
check there is idempotent, safe whether or not the fix is already applied.

## 3. Report

For each check — binary, app reachable, `ai-agent/` present, excluded-files, the three
property types, `CLAUDE.md`, `$OBSIDIAN_VAULT` — say **already correct**, **fixed**, or
**needs Fred**. Note plainly if this run only exercised the all-green path (everything was
already configured) — the fix branches only get exercised for real on a machine that actually
needs them.

## Rules carried over from the rest of this project

- **Never touch git.** The vault isn't a repo. Dotfiles and `skills-private` are — this skill
  may edit files inside them (§4 of `setup.md`, and its own copy-out step), but never
  `commit` or `push` either without Fred asking.
- **`ai-agent/docs/` is free-write scratch** — the property-type registration's throwaway note
  needs no confirmation, per the two-tier model in `ai-agent/README.md`. This skill should
  never need to touch anything outside `ai-agent/` at all.
- **Measure `unresolved total` before any vault write and re-check it after** — editing
  `setup.md` itself counts. Compare against your own before-measurement, not a hardcoded
  number; the recorded baseline drifts as Fred edits his own notes. Fix only what this run broke.
- Direct edits to `.obsidian/app.json` and `.obsidian/types.json` are safe with the app
  running (verified 2026-08-28 and 2026-08-31 — no reload needed, and the app does not
  clobber a direct edit on its own) — but prefer the CLI (`property:set`) for the property
  types specifically, to stay on the one path Phase 3 already uses for every other property
  write, rather than opening a second one.
