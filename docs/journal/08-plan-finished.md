# Session 08: the plan finished before Build; intent revision 20 (D97 to D101)

Date: 2026-10-10 and 2026-10-11. Tool: Claude Code Desktop (Windows). Model: Claude Opus 5.5.
Commits: 3d51839, 67748d7, e10eea5 (intent revision 20); 1d35fb8, 0488f05 (the spec); 4c6cebe (the plan
and CLAUDE.md); and this entry, with the plan's last corrections and review 08.

## 1. Opening prompt

```text
We are delivering this project with Anthropic's AI-Native SDLC playbook
(https://claude.com/blog/the-ai-native-sdlc-playbook). Session 07 ran the
AgentOps coverage check and revised the intent (revision 19, D89 to D96), the
spec and the implementation plan. The plan's independent read found defects
that are not yet applied. This session finishes the plan, then starts Stage 4
(Build) at plan step 0.1.

My decisions, given at the end of session 07 and written in journal 07,
section 7. Record them as D97 onwards, intent first:
1. Do not worry about the day estimates. Phase 1 stays as it is, nothing is
   cut or moved, and the estimates are not a constraint: Claude Code does
   nearly all of the work, and I can run sessions in parallel. Do it all in
   one go where that makes sense.
2. A session may run az acr build into the project's registry. This is an
   exception to D40 and changes CLAUDE.md's rule that nothing touches the
   persistent group. You do the heavy lifting.
3. CLAUDE.md's architecture summary is already updated. Nothing to do.

Read first, in this order:
- CLAUDE.md (process rules; follow them all)
- docs/journal/07-agentops-coverage-check.md, section 7
- docs/council/07-agentops-coverage-principal-review.md, sections 10 and 11
- docs/intent.md, section 14 from D89 to D96
- docs/implementation-plan-phase-0-1.md, sections 1 to 4, 9 and 11, and steps
  0.4, 0.10, 1.1, 1.3, 1.7, 1.9 and 1.12
- docs/spec-phase-0-1.md, sections 2.5, 5.7, 5.8, 5.10 and 5.12

Pre-flight, before anything else. Report the result and stop if any item fails:
- /mcp shows azure (pinned to 2.0.5), microsoft-learn and context7 connected.
- git is clean and in sync with origin at cc9637d or later; gh's active account
  is ranaidrees.
- az bicep, uv 0.12.23 or later, and azd with the azure.ai.agents extension are
  installed. Docker is not required (D89).
- The system drive has about 5 GB free. Install nothing without measuring the
  disk and asking me first.

Rules for this session:
- Start in plan mode. No edits before I have approved the plan.
- Research before the interview: every statement that something is current,
  preview or GA cites an official page opened in this session, with quotations
  checked against the page by script, by paragraph and over the whole document.
- Keep questions few, each with options and your recommendation first.
- A change to the intent or the spec is a new decision, D97 onwards: intent
  first, then spec, then plan, each after the citation check, with an
  independent read before the next is derived.
- Research subagents save nothing under the repository.
- No Azure write before step 0.2. State the added monthly cost and the blast
  radius before any Azure write. Never approve a GitHub deployment.
- Plain British English, no em dashes.

Tasks:
1. Finish the plan. Record my decisions above. Apply every finding of review
   07, section 11: none has been applied. Findings 1, 3, 5, 6, 7, 8 and 10 are
   text. Finding 2 needs decision 2 above and the registry's role mode checked
   against Microsoft's page, which session 07 did not open. Finding 4 touches
   D94's sentence on Phase 0. Finding 9 needs step 0.10 read. Finding 11 needs
   the blocklist item's page. Then the citation check, the schedule sums by
   script, a render of any diagram that changes, and one short confirming
   independent read of the corrected plan. Commit and push.
2. Step 0.1, the toolchain and workspace skeleton, on a branch and by pull
   request. Re-resolve the pins and record every change: uv 0.13.0, fastmcp
   4.1.0, langgraph 1.2.14, azure-ai-projects 2.8.0, pydantic 2.14.0, the azd
   extension's 1.0.0-beta.19 and the Foundry API version 2026-09-01. Fill the
   Commands section of CLAUDE.md.
3. Step 0.2: write the Bicep and maf bootstrap azure, show me the what-if and
   the cost, and stop for me to run it.
4. Tell me which of the remaining Phase 0 steps could run in parallel
   sessions once steps 0.1 and 0.2 are in, and which must stay in order.
5. Write docs/journal/08-<stage>.md from the template, with the actual time
   each step took against its estimate. Run /cleanup, commit and push.
6. Give me a summary, any questions, and a handover prompt for the next
   session, then stop. If the session grows long, stop at a clean, pushed
   state and hand over.
```

Part-way through, after an expired sign-in stopped one independent reader, the owner wrote:
"continue, when makes sense then provide me handover prompt for new session". The session stopped
at its first clean checkpoint, the finished plan. Tasks 2 and 3 go to the next session.

## 2. What happened, in order

1. Pre-flight passed. azd is installed but off the session's path, as before, and is called by its
   full path.
2. Plan mode. The session read the named sections itself and opened the pages session 07 had not:
   the container registry's role modes and the blocklist item. It resolved every pin again.
3. Four questions to the owner, who took every recommendation (section 3).
4. A throwaway workspace in the scratchpad locked the re-resolved pins together on Python 3.11.
5. Intent revision 20, then its independent read: two must-level defects, both in D98.
6. The spec, then its independent read, which first failed on an expired sign-in and was run again:
   four must-level defects, two of which reached back into the intent.
7. The plan: all fifteen findings of review 07, the five decisions, and nine gaps the session found
   itself. Then the confirming read: Build can start, with one must-level gap in step 0.2.
8. `/cleanup` reported and changed nothing. The session stopped here.

Time, from commit and tool clocks, so approximate. Steps 0.1 and 0.2 were not reached, so there is
no actual to set against their 0.25 and 0.5 day.

| Task | Time |
|---|---|
| Pre-flight, reading, research, the interview and the session's plan | About 15 minutes |
| Intent revision 20, its read and its corrections | About 30 minutes |
| The spec, its read (run twice) and its corrections | About 20 minutes |
| The plan, its confirming read and its corrections; review 08; this entry | About 30 minutes |

## 3. Owner decisions

| Question | Answer | Recorded in |
|---|---|---|
| What happens to the Phase 1 box (answered after session 07) | The day figures are sizes, not boxes. Nothing is cut or moved | intent D97 |
| What replaces the time-boxes | The go criteria decide. Any miss goes to the owner, who chooses the fallback or another session. Only the owner invokes a cut order | intent D97 |
| Who runs `az acr build` (answered after session 07) | A session may, without asking | intent D98 |
| Which role mode for the registry, and how far the exception reaches | The newer mode. Builds go to `scratch/` only, and the owner's write role is limited to it by a condition | intent D98 |
| CLAUDE.md's architecture summary (answered after session 07) | Stands as changed | intent D99 |
| Where it is learned whether structured outputs work through the gateway | In spike S1, test 1, as a record | intent D100 |
| Two corrections from the plan's read that reach the spec | Accepted with the session's plan: `up` applies the environment's Bicep on every run; three new fixed texts | intent D101 |
| What may be installed | The workspace's packages with uv's cache on M:; the azd extension's beta.19 | Review 08, section 2 |

## 4. Surprises and how they were handled

- **Finding 2 was wider than the reviewer said.** The plan named roles from both of the registry's
  role modes. Whichever mode the registry took, either the agent's image pull or the pipeline's
  push would have failed.
- **Every independent read found must-level defects again:** two, four and one. The pipeline
  identity could not have signed in to the registry. A guard the session added on its own
  initiative would have failed the very pipeline run it was meant to protect, and was withdrawn.
  The owner's login had no data role on either storage account.
- **The storage module closes an account's firewall by default.** Nothing in the plan said so.
  salon-mcp and the pipeline would have been refused at their first write.
- **The session's own citation script was wrong twice.** It mispaired quotation marks after a short
  quotation, and it took the text between two code spans for a code span, which hid quotations
  from the check. Both were found only because a count looked too low.
- **A `git stash` round trip changed two working copies to CRLF,** and the next scripted edit
  failed to match. The files were put back to LF; nothing was committed wrongly.
- **Shell here-documents in this harness are not literal:** backslashes were rewritten. Scripts
  are now written as files.
- **A sign-in expired mid-session** and stopped one reader. The read was run again in full.

## 5. What was produced

- [docs/intent.md](../intent.md): revision 20, decisions D97 to D101.
- [docs/spec-phase-0-1.md](../spec-phase-0-1.md): revised; section 2.9 is new; 153 keys, 145
  quotations, none missing; none of the seven diagrams changed.
- [docs/implementation-plan-phase-0-1.md](../implementation-plan-phase-0-1.md): every finding
  applied; section 11.3, order and parallel sessions, is new; 159 keys, 82 quotations, none
  missing; the sums are 12 and 20.75 by script.
- [docs/council/08-plan-corrections-principal-review.md](../council/08-plan-corrections-principal-review.md):
  the registry check, the currency audit, the three reads, and what was done with each finding.
- [CLAUDE.md](../../CLAUDE.md): the Azure writes rule carries the D98 exception. 84 lines.
- On the workstation: the azd extension is at 1.0.0-beta.19. uv's cache is at `M:\uv-cache`. The
  system drive has 4.97 GB free.
- No application code, no infrastructure, no Azure write, no pull request. `/cleanup` checked 33
  files and removed nothing. It proposed one note, which the owner approved: review 07, section 11,
  now says at its top that its findings were applied in session 08.

## 6. Reusable lessons

- An independent read is worth more than the author's check every time. This is its fifth
  appearance, and it is already a rule in CLAUDE.md. Budget for one after every document.
- Do not add a control on your own initiative while correcting a document. The one added here was
  wrong, and cost a correction in three documents.
- Check a checking script against a count you can make by eye before trusting it.
- Never `git stash` in a working copy that scripts are editing. Compare with `git show` instead.
- Write scripts as files. Do not pass them through the shell.
- Read a module's source for its defaults, not only its README. The storage module's closed
  firewall is in the source alone.
- Harness mechanism not needed this session: the council agents, because revised artefacts get a
  single-reviewer pass (D70); and the Azure MCP server's write tools.

## 7. Next session

Four things were put to the owner at the end of the session, and all four were answered the same
night. The answers follow each. The first needs nothing. The second and third are not yet
recorded as a decision: that is the next session's first work, D102, intent first.

1. **The pipeline identity holds a second role on the registry,** the read-only configuration
   reader, because Microsoft's roles reference lists `az acr login` under it and gives the writer
   role no right on the registry resource. It goes beyond what was asked.
   **Answer:** no objection. It stands as D98 has it.
2. **Whether CLAUDE.md should carry the rules for parallel sessions** of plan section 11.3: one
   step, branch, pull request and working tree for each session; one session at a time holds the
   dev environment; a spike's miss goes to the owner.
   **Answer:** yes, as recommended, in three lines.
3. **Who merges the pull requests of Build.** The plan says a session does not merge its own.
   **Answer:** Claude Code can merge Build's pull requests. This changes rule 1 of plan section
   11.3 and the Commits row of plan section 3.
4. **The one note `/cleanup` proposed,** on review 07, section 11.
   **Answer:** add it. Done in this session, in commit e302f93.

Next session (Stage 4, Build, steps 0.1 and 0.2):

1. Pre-flight without Docker. Read CLAUDE.md, this section, and the plan's sections 1 to 3, steps
   0.1 and 0.2, and sections 9.1, 9.3, 9.9 and 11.3. Then record answers 2 and 3 above as D102:
   the intent, then the plan's section 3 and rule 1 of section 11.3, then CLAUDE.md.
2. Step 0.1 on a branch and by pull request: the workspace, the three packages, `cli.py`, `pr.yml`
   in its first form, and CLAUDE.md's Commands section. The pins are in plan section 3 and were
   locked together on 2026-10-10. Every uv command runs with `UV_CACHE_DIR=M:\uv-cache`.
3. Step 0.2: the persistent group's Bicep and `maf bootstrap azure`. Show the what-if, the cost and
   the blast radius, and stop. The owner runs it. The subscription also holds another project's
   groups, `rg-iu-admissions-weu` and `rg-iu-admissions-us`, which nothing here touches.
4. Once step 0.1 is merged, steps 0.14 and 0.16 can each run in a session of their own, and step
   0.7 once the bootstrap has run. Section 11.3 of the plan gives the rest.

Still open: the items of plan section 12 and spec 2.5; the wording of the fixed texts; the ten-row
rule; whether the condition on the owner's write role holds, which step 0.4 shows; and whether the
names `apim-maf-dev`, `fnd-maf-agent` and `fnd-maf-models` are free across Azure, which steps 0.3
and 0.5 find out. The workstation's system drive has about 5 GB free: install nothing large on it.
