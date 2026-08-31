---
name: second-brain-recall
description: >
  Search Fred's Obsidian vault (his "second brain", ~1850 notes) and answer from it.
  Use when the user asks what they know, wrote, or decided about something — "how did I fix
  that Linux audio issue?", "what are my notes on dbt?", "check my vault for X" — and also
  mid-task in an unrelated repo when precedent would help: "have I solved something like this
  before?", "did I write this up anywhere?". Read-only; never writes to the vault. For saving
  something new, use second-brain-save instead.
---

# Second brain — recall

Read-only retrieval from Fred's Obsidian vault. **This skill carries no instructions of its
own beyond getting you to the vault** — the recipe lives in the vault so all agent tools stay
in sync. Resolve, read, then follow.

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
`Vault not found.` Requiring a real `path` line *and* a real `.obsidian` directory is the
only guard that holds for failure modes you haven't seen.

**If that fails: tell Fred to open Obsidian, and stop.** Do not fall back to grep, `find`, or
reading files directly to answer anyway. The no-fallback rule is deliberate — a half-answer
assembled from the filesystem is worse than a clean retry prompt.

Always pin `vault=obsidian_personal` on every command.

## 2. Read the recipe

```bash
obsidian vault=obsidian_personal read path="ai-agent/workflows/retrieve.md"
```

Read `ai-agent/vault-map.md` too when the question implies a folder ("work notes", "my
projects"). If you have not used this CLI in the session, read
`ai-agent/references/obsidian-cli.md` first — **it fails silently: rc=0 on error, errors on
stdout, wrong flags ignored rather than rejected.**

Read these by path. `ai-agent/` is excluded from Obsidian's search index, so `obsidian search`
will not find them.

## 3. Follow it

`retrieve.md` is an ordered recipe: sanitize the query → triage with `search:context` →
cut `Daily/` noise → **bilingual retry** → sharpen with operators → expand through the link
graph → read the survivors → answer with wikilinks.

Three things from it that decide whether the answer is right:

- **Never say "there's nothing in your vault about X" without trying the other language.**
  The vault is bilingual PT/EN. `prayer` returns 0 results; `oração` returns 23. An empty
  result set is not evidence of absence.
- **An empty `backlinks` is not a dead end.** 749 of 1854 notes are orphans. The graph helps
  where it exists; search variants are the backbone.
- **Check every CLI result for `Error:` before believing it,** and sanity-check
  `[property:value]` searches against a count you can predict — `[type:: cheatsheet]` silently
  returns 467 instead of 8.

## 4. Answer

Cite with wikilinks so Fred can jump straight there: `[[Note]]` when the basename is unique,
`[[folder/Note|display]]` when it isn't (`Dart.md` exists in two folders). Say which folder
each note lives in, separate what the notes say from what you inferred, and flag thin or stale
notes rather than confidently synthesizing over them.

## Read-only by construction

Allowed: `vault`, `read`, `search`, `search:context`, `file`, `files`, `folder`, `folders`,
`backlinks`, `links`, `outline`, `tags`, `tag`, `properties`, `property:read`, `aliases`,
`bases`, `base:query`, `base:views`, `daily:path`, `daily:read`.

**Never, in this skill:**

- **`open`, `search:open`, `tab:open`, `daily`, `random`** — these hijack Fred's live app and
  change what he is looking at. Answer in the terminal; let him click the wikilink.
  Note the difference: bare **`daily` opens** today's note in his app and is forbidden, while
  `daily:read` and `daily:path` merely print and are fine. Don't collapse the two.
- `unresolved`, `orphans`, `deadends` — link-debt *maintenance*, not retrieval. A recall
  invocation has no business auditing the vault's health; it will burn context and answer a
  question nobody asked.
- Any write: `create`, `append`, `prepend`, `property:set`, `rename`, `move`, `delete`,
  `task`, `daily:append`.
- Reading `Templates/` — it contains unresolved Templater syntax.

If the retrieval turns up something worth saving, say so and hand off to
`second-brain-save`. Do not write it yourself.
