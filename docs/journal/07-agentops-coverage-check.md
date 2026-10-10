# Session 07: the AgentOps coverage check; intent revision 19; the plan revised and read, with corrections open

Date: 2026-10-04 (pre-flight) and 2026-10-10. Tool: Claude Code Desktop (Windows). Model: Claude Opus 5.5.
Commits: 98bf704, 547e9c4, b82d7a1 (intent revision 19); 7a85a41, 60c845a (the spec); 4766d74 (the plan);
and this entry with review 07.

## 1. Opening prompt

```text
We are delivering this project with Anthropic's AI-Native SDLC playbook
(https://claude.com/blog/the-ai-native-sdlc-playbook). Stages 1 to 3 are closed:
the intent (revision 18, D1 to D88), the spec and the implementation plan are
accepted and pushed. Before Stage 4 (Build) starts at plan step 0.1, this
session checks that the three documents are aligned and consolidated on
AgentOps practice.

Read first, in this order:
- CLAUDE.md (process rules; follow them all)
- docs/journal/06-implementation-plan-gate.md, section 7 (the handoff, with
  what the last session's read-only check found)
- docs/intent.md, sections 3, 6, 7, 10 and 14
- docs/spec-phase-0-1.md, sections 3.4, 5.1, 5.6, 5.7, 5.8, 5.11, 9 and 11
- docs/implementation-plan-phase-0-1.md, sections 1 to 4, 9 and 11, steps
  1.3, 1.7 and 1.9, and Appendices A and F
- docs/council/06-implementation-plan-phase-0-1-principal-review.md, sections
  1 and 2

Pre-flight, before anything else. Report the result and stop if any item fails:
- /mcp shows azure (pinned to 2.0.5), microsoft-learn and context7 connected.
- git is clean and in sync with origin at 38d48e3 or later; gh's active account
  is ranaidrees.
- The local tools of plan section 3 are installed: az bicep, uv 0.12.23 or
  later, azd with the azure.ai.agents extension, Docker Desktop.

Rules for this session:
- Start in plan mode. No edits before I have answered.
- Research before the interview: every statement that something is current,
  preview or GA cites an official page opened in this session, with quotations
  checked against the page by script.
- Keep questions few: one short round, each with options and your
  recommendation first, then offer to proceed on your recommendations.
- A change to an accepted document is a new decision, D89 onwards: intent
  first, then spec, then plan, each after the citation check, and an
  independent read before the next one is derived from it.
- Research subagents save nothing under the repository.
- No Azure writes in task 1. Plain British English, no em dashes.

Tasks:
1. The AgentOps coverage check, as a gate before any code. For each practice
   below, say where the intent, the spec and the plan state it, in which
   phase it is built, whether the three agree, and what is missing:
   - offline evaluation: the eval gate, its evaluators, thresholds and
     baseline, and the scripted write tests;
   - online evaluation: continuous evaluation of live traffic, and scheduled
     evaluation of the served version against the golden dataset; say which
     of these needs message content, given that D46 keeps recording off;
   - the golden dataset: how it is signed, versioned and hashed, and how it
     grows from what online evaluation, guardrail hits and user feedback
     find, including Foundry's route from traces to a dataset;
   - Pydantic and typed schemas: the graph's state, the extraction node's
     output, the tools' inputs and outputs, the registry rows, the release
     record and the gate result; whether Pydantic is named and pinned;
   - input guards: prompt-injection screening, validation in code, tool
     schemas;
   - output guards: the citation check, the output content filter and a test
     that it refuses, PII in responses, the shape of the final response;
   - the rest of good AgentOps practice: tracing and attribution, cost and
     token budgets, versioning, approval, rollback, drift, safety evaluators
     and red teaming, human in the loop, alerts and the maintain loop, user
     feedback.
   Check the list itself against Microsoft's current guidance for Foundry
   observability and evaluation, and add what it names that I have not.
   Give me: one coverage table (practice, phase, where stated in each
   document, aligned or not); the gaps ranked by impact; and for each gap a
   recommendation with its cost in days and the phase it belongs to. Say
   plainly which gaps fall beyond the stop line at Phase 3 (D30).
2. Ask me for my decisions in one short round. If I want changes: record them
   as intent revision 19 (D89 onwards), then the spec, then the plan,
   including one table in the spec that says which practice is built in which
   phase. Phase 0 is 12 working days and Phase 1 is 18, both full: every
   addition says what pays for it. If any accepted document changes, write the
   journal entry, push and stop; Build starts in the next session.
3. Only if task 2 changed nothing: step 0.1, the toolchain and workspace
   skeleton, on a branch and by pull request. Re-resolve the pins and record
   any change (fastmcp 4.0.11 and the Foundry API version 2026-09-01 have
   appeared since the plan). Fill the Commands section of CLAUDE.md.
4. Then step 0.2: write the Bicep and maf bootstrap azure, show me the
   what-if and the cost, and stop for me to run it. State the added monthly
   cost and the blast radius before any Azure write. Never approve a GitHub
   deployment.
5. Write docs/journal/07-<stage>.md from the template, with the actual time
   each step took against its estimate. Run /cleanup, commit and push.
6. Give me a summary, any questions, and a handover prompt for the next
   session, then stop.
```

## 2. What happened, in order

1. Pre-flight failed on the local tools: no Bicep CLI, uv 0.10.4, azd off the session's path with
   no extension, no Docker. The session reported and stopped, as told.
2. The owner said to install them. Bicep 0.48.1, uv 0.13.0 and the azd extension went in. Docker
   Desktop went onto the system drive, took 3.9 GB of its last 6, and was removed the same hour
   when the owner said the drive was nearly full. The owner then chose no Docker at all (D89).
3. Plan mode. The session read the named sections of the three documents itself. Three research
   subagents, which wrote nothing, checked Microsoft's pages on offline evaluation, online
   evaluation, and observability and guardrails. 68 quotations were matched to their pages by script.
4. Task 1 went to the owner: a coverage table of 26 practices, six ranked gaps, what Microsoft
   names that the list did not, and what lies beyond the stop line.
5. One round of four questions. The owner took every recommendation: typed contracts, the output
   filter shown refusing, a nightly evaluation of the served version, and a longer Phase 1.
6. Intent revision 19, then its independent read: three must-level defects. Two went back to the
   owner (a fifth test identity, D96; the cut order).
7. The spec, then its independent read: four must-level defects, six sentences beyond their
   sources, and an estimate of 2.75 days against the session's one. The owner extended Phase 1 to
   20.75 working days. Two older quotations had drifted from their pages and were corrected.
8. The plan, then its independent read: two must-level defects, nine should-level, and a third
   estimate of about four days. **None of these is applied.** The owner asked for a handover at a
   sensible point, and the session stopped here.

Time, from tool and commit clocks, so approximate. No step had an estimate, and steps 0.1 and
0.2 were not reached.

| Task | Time |
|---|---|
| Pre-flight, the installs and the Docker reversal | About 40 minutes over two days |
| Task 1: reading, research, the check | About three hours |
| Task 2: three documents, three independent reads, review 07 | About an hour and a half |

## 3. Owner decisions

| Question | Answer | Recorded in |
|---|---|---|
| Docker on a workstation with 5 GB free | None; spike images by `az acr build` | intent D89 |
| Typed schemas | Pydantic 2 at every boundary; committed schemas compared on every pull request | intent D90 |
| The output content filter | Claimed and shown refusing a planted term | intent D91 |
| Online evaluation | A nightly evaluation of the served version in Phase 1; continuous evaluation stays in Phase 6 | intent D92 |
| The dataset growth loop | Restated for traces without message content; the drill in Phase 2 | intent D93 |
| What pays | Phase 1 extended, first to 19 and then to 20.75 working days on the independent estimate | intent D94 |
| One table of practice by phase | Section 5.12 of the spec | intent D95 |
| Who makes the filter test's call | A fifth test identity | intent D96 |
| Where the two new builds sit in the cut order | Third and fourth, after the agent route | intent D94 |

## 4. Surprises and how they were handled

- **The session installed 3.9 GB on a drive with 6 GB free without looking.** It measured, removed
  it, and found that the drive the project lives on is a virtual disk file on another partition.
- **The session told the owner that nothing in Phase 0 uses Docker. That was wrong.** The spikes
  build images before the pipeline exists. It was caught while writing D89 and the owner was told.
- **Every independent read found must-level defects, and each estimate was higher than the last:**
  1.0, 2.75, about 4. The nightly evaluation as first written would have failed on about 64
  nights in 100, and then on one in three.
- **The intent cited a review file that did not exist.** It was written before the spec was derived.
- **Two accepted quotations no longer matched their pages** after a week: Microsoft had reworded
  the guardrail page and the checkpoint saver's README. Only a check of the whole document, by
  paragraph, found them; a check of the new lines would not have.
- **The first citation script checked line by line** and so skipped every quotation that wrapped.
  It was rewritten to check paragraphs.
- **The gap the last session reported as "stated nowhere" was stated,** in spec 5.11, and could
  not work: it built an eval row from a trace that holds no message content.

## 5. What was produced

- [docs/intent.md](../intent.md): revision 19, decisions D89 to D96, fourteen amendment notes.
- [docs/spec-phase-0-1.md](../spec-phase-0-1.md): revised; section 5.12 is new; 148 keys; seven
  diagrams rendered, one changed.
- [docs/implementation-plan-phase-0-1.md](../implementation-plan-phase-0-1.md): revised; Phase 1
  sums to 20.75 by script; 141 keys. **Its review's findings are open.**
- [docs/council/07-agentops-coverage-principal-review.md](../council/07-agentops-coverage-principal-review.md):
  the check, the currency audit and the three independent reads.
- Local tools on the workstation: Bicep 0.48.1, uv 0.13.0, azd 1.35.0 with `azure.ai.agents`
  1.0.0-beta.18. No Docker.
- [CLAUDE.md](../../CLAUDE.md): the architecture summary names typed contracts, the output filter's
  test and the nightly evaluation, on the owner's yes. 81 lines.
- No application code, no infrastructure, no Azure write. `/cleanup` found nothing to remove.

## 6. Reusable lessons

- Before installing anything, measure the disk and say where it will go.
- Never tell the owner that a step does not need something without searching the plan for it.
- An independent read is worth more than the author's check every time. This is its fourth
  appearance. Budget for three of them when three documents change, and for the corrections.
- Take the independent estimate, not the author's. Ask for it before the owner is asked what pays.
- Check citations by paragraph and over the whole document, not the changed lines: pages move.
- A statistical rule needs its false alarm rate worked out before it is written down.
- When a decision names a document, write the document first.
- Harness mechanism not needed this session: the council agents, because revised artefacts get a
  single-reviewer pass (D70); and the Azure MCP server's write tools.

## 7. Next session

Three questions were put to the owner at the end of the session. All three were answered the same
evening, and the answers follow each question. They are not yet recorded as decisions: that is
the next session's first work, D97 onwards, intent first.
1. **The Phase 1 box.** Three estimates exist for the additions: 1.0, 2.75 (adopted) and about 4.
   Hold 20.75 and let the cut order pay, where the nightly evaluation goes third; or extend to 22;
   or move the nightly evaluation to Phase 2 now.
   **Answer:** do not worry about the day estimates. Do it all in one go where that makes sense;
   sessions may run in parallel, and Claude Code does nearly all of the work. Nothing is cut or
   moved.
2. **Who runs `az acr build`.** The registry is in the persistent group, which D40 says no session
   touches. Either the owner runs each spike image build, or D40 gains an exception.
   **Answer:** the exception, as recommended: a session runs the image builds into the project's
   registry.
3. **CLAUDE.md's eight-line architecture summary** does not mention typed contracts, the output
   filter's test or the nightly evaluation. Say whether to add them.
   **Answer:** yes. Done in this session.

Next session (finish the plan, then Stage 4, Build, from plan step 0.1):
1. Pre-flight without Docker. Read CLAUDE.md, this section, review 07 sections 10 and 11, and
   plan sections 1 to 4, 9 and 11.
2. Apply review 07, section 11, to the plan: every finding is open. Findings 1, 3, 5, 6, 7, 8 and
   10 are text. Finding 2 needs decision 2 above and the registry's role mode checked against
   Microsoft's page, which this session did not open; CLAUDE.md's rule that nothing touches the
   persistent group changes with it. Finding 4 touches D94's sentence on Phase 0. Finding 9 needs step 0.10 read. Finding 11 needs the blocklist item's page. A change to the
   intent or the spec is D97 onwards, intent first. Citation check by paragraph over the whole
   plan; schedule sums by script; one short confirming read.
3. Step 0.1 on a branch and by pull request. Re-resolve the pins: uv 0.13.0, fastmcp 4.1.0,
   langgraph 1.2.14, azure-ai-projects 2.8.0, pydantic 2.14.0, the azd extension's beta.19, the
   Foundry API version. Fill the Commands section of CLAUDE.md.
4. Step 0.2, the bootstrap of the persistent group. State cost and blast radius; the owner runs it.
5. Stop at the end of a step, with a journal entry.

Still open: the items of plan section 12 and spec 2.5; the wording of the fixed replies; the
ten-row rule; and whether the deployment's guardrail acts on calls through the gateway, which the
output filter's test settles. The workstation's system drive has about 5 GB free: install nothing
large on it.
