---
name: cleanup
description: Find and remove clutter in this repository - broken links and references, stale or contradictory statements, duplicated content, unused agents, commands, skills and MCP servers, orphaned files, and documents or artefacts nobody asked for. It reports first and changes nothing until the owner approves. Use it before the final commit of every stage, whenever the owner asks to clean up, tidy, consolidate, prune or audit the repo or its docs, and whenever you are about to add a new document and are not sure it is needed.
argument-hint: "[optional path to limit the pass, for example docs/]"
---

# Cleanup and consolidation

Run a cleanup pass on: $ARGUMENTS (the whole repository if empty).

## Why this exists

This repository is read by people (the owner, interviewers) and by every later Claude Code
session. Each unnecessary, duplicated or wrong file costs attention and trust, and each later
session pays to read it. The aim is fewer, truer documents.

The repository is also the audit trail. Tidying must never remove evidence. That is why this skill
reports before it changes anything, and why some files are protected.

## What each file is

Classify every file before judging it.

| Class | Files | What this skill may do |
|---|---|---|
| Protected record | `docs/council/*.md`, `docs/journal/NN-*.md`, accepted revisions of chain artifacts, git history | Report only. Never delete or rewrite. If one would mislead a later stage, propose a one-line "Superseded by" note at the top. |
| Chain artifact | `docs/intent.md`, `docs/spec-phase-N.md`, `docs/implementation-plan-phase-N.md` | Propose corrections. A change to an accepted artifact is a new revision recorded as an owner decision, not a tidy-up. |
| Harness | `CLAUDE.md`, `.claude/`, `.mcp.json`, `.gitignore`, `docs/journal/TEMPLATE.md` | Fix, merge or remove with approval. |
| Supporting document | anything else under `docs/` | Correct, merge or remove with approval. |
| Code and infrastructure | source, tests, IaC, CI | Look only for dead files, leftover scaffolding and generated files that should be ignored. Do not refactor; that is a code review's job. |
| Stray | anything that fits none of the above | Usually delete or ignore, with approval. |

## Steps

1. **Orient.** Read `CLAUDE.md` and section 7 of the latest entry in `docs/journal/` so you know the
   current stage and what is planned but not yet built.
2. **Inventory.** List tracked files (`git ls-files`), then untracked and ignored ones
   (`git status --porcelain --ignored`). Give each file a class.
3. **Check.** Work through the checks below. Change nothing.
4. **Report.** Give the report in the chat, in the format below. Do not write it to a file: a
   cleanup that leaves a new document behind defeats its purpose.
5. **Wait.** The owner approves all, some or none of the numbered findings.
6. **Apply** only what was approved. Delete with `git rm` so the file stays recoverable from
   history. Never rewrite history. Make one commit titled `Cleanup: <summary>` that lists each
   change. Then rerun checks 1 and 5 to confirm nothing new broke.
7. **Record.** Add one line to this session's journal entry saying what was removed or merged.

## Checks

1. **Broken references.** Every relative link and every backticked path in Markdown, agent,
   command and skill files should resolve. Separate broken from planned: a path that the artifact
   chain says a later stage will create (for example a spec before the Design stage) is planned,
   not broken. Say which is which.
2. **Stale or contradictory statements.** Text that disagrees with the latest decisions in
   `docs/intent.md` section 14 or with `CLAUDE.md`: old file names, old phase numbers, superseded
   rules, questions already answered. In protected records these are history; report them and
   leave them.
3. **Duplication.** The same content kept in two places will drift. Each kind of content has one
   home: decisions in `docs/intent.md`; process rules in `CLAUDE.md`; how to run something in its
   command or skill; what happened in the journal. Propose replacing copies with a link.
4. **Sprawl.** `CLAUDE.md` over one page, a journal entry over two pages, two documents serving the
   same purpose, a document longer than its readers need.
5. **Unused harness.** Agents that no command or skill starts; commands and skills that nothing
   mentions and nobody has used; MCP servers in `.mcp.json` that no agent or document relies on;
   settings or hooks that point at missing files.
6. **Orphans and strays.** Files outside the chain that nothing references; empty or near-empty
   files; TODO placeholders whose stage has passed; scratch notes, exports, logs, editor and
   operating-system leftovers; tracked files that should be ignored.
7. **Secrets and personal data.** Keys, tokens, connection strings or real personal data. Put
   these at the top of the report. Name the file and line, never print the value.

Check external URLs only when the owner asks for a deep pass. It is slow, and a dead link in a
dated record is history rather than clutter.

## Report format

```markdown
# Cleanup report: <scope>, <date>, commit <short hash>

<N> files checked, <N> findings, <N> proposed deletions.

## Findings
| # | File | Finding | Class | Proposed action |
|---|---|---|---|---|

## Protected records (reported, not changed)
- <file>: <what is stale or wrong, and whether a "Superseded by" note is proposed>

## Checked and clean
One line listing what was checked and found in order.

## Prevention
At most three points, only where a pattern is clear: how this clutter arose and a rule that would
stop it. These are proposals for the owner, not changes.
```

Use these words for the proposed action so findings are easy to approve by number: Keep, Fix
reference, Correct statement, Merge into `<file>`, Mark superseded, Delete, Ignore, Ask owner.

## Before adding any new document

The cheapest clutter to remove is the clutter never created. Add a document only if it is one of:
a chain artifact, a council review, a journal entry, harness configuration, or something the owner
asked for. Otherwise put the content in an existing file or in the chat.
