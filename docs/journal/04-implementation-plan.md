# Session 04: Stage 2 gate closed, Stage 3 implementation plan written, reviewed and revised

Date: 2026-10-04. Tool: Claude Code Desktop (Windows). Model: Claude Fable 5.1, then Claude Opus 5.5
from the cleanup commit onwards (the owner switched models).
Commits: fdbee4a (spec accepted, CLAUDE.md architecture), 35b4360 (intent revision 14), 9919729
(spec for D71), 7dfc3c8 (plan), d84bbc5 (council review 05), ffa7a24 (intent revision 15), c9c4251
(spec for D72 to D79), b5f0d22 (plan revised), 8ba417d (cleanup), c1e1408 (three quotations
corrected), and the commit that adds this entry.
The plan is revised after its council review and still not accepted; see section 7.

## 1. Opening prompt

```text
We are delivering this project with Anthropic's AI-Native SDLC playbook
(https://claude.com/blog/the-ai-native-sdlc-playbook). The Stage 2 spec has
been reviewed by the council, then by a single-reviewer principal pass with a
currency audit, and revised after each; it is committed and pushed, but I
have not yet accepted it. This session finishes the Stage 2 gate and, if I
say so, starts Stage 3: the implementation plan. Do not create application
code, Python packages or infrastructure.

Read first, in this order:
- CLAUDE.md (process rules; follow them all, including D70)
- docs/journal/03-spec-principal-review.md, section 7 (the handoff)
- docs/intent.md (revision 13; source of truth, decisions D1 to D70)
- docs/spec-phase-0-1.md (revised; status "awaiting the owner's acceptance")
- docs/council/04-spec-phase-0-1-principal-review.md (sections 1, 2, 11, 12)
- docs/council/03-spec-phase-0-1-review.md (owner questions only)

Pre-flight, before anything else. Report the result and stop if any item fails:
- /mcp shows azure, microsoft-learn and context7 connected.
- /council and /cleanup are available, and the seven council-* agents are listed.
- git is clean and in sync with origin at d7483af or later; gh's active account
  is ranaidrees.

Rules for this session:
- Start in plan mode. Ask, do not assume, but keep questions few: one short
  round, each question with options and the recommended option first, then
  offer to proceed on your recommendations.
- Research before the interview (D55). Every design claim cites an official
  document or sample with a URL you opened in this session. Paste quotations
  and check them against the page text, not a tool's summary.
- Research subagents save nothing under the repository; they use the scratchpad.
- Before committing the spec or the plan, run the citation check: every key in
  square brackets resolves to a reference row, code fences balance, no stale
  phrase remains.
- No Azure writes. Read-only lookups are fine. The plan may describe Azure
  writes, each with its estimated monthly cost and blast radius.
- The dev budget is 40 pounds a month (D21). Quote prices as a rate with a
  table of usage.
- Plain British English, no em dashes.

Tasks:
1. Ask me whether I accept the spec. Before I answer, give me a one-screen
   summary of D56 to D70, which were recorded on my instruction as a block,
   one line each for D47 to D55, and the sections of the spec you think I
   should read first.
2. If I want changes: make them, record any that alter the intent as revision
   14 (D71 onwards), commit the intent first and then the spec. Do not rerun
   the council (D70).
3. When I accept: set the spec's status to accepted, fill the Architecture
   section of CLAUDE.md from sections 3.1, 3.4 and 4 in about eight lines,
   keep the file under one page, and commit.
4. Then ask me whether to continue to Stage 3 in this session or stop. If I
   say continue: propose the outline of docs/implementation-plan-phase-0-1.md
   and wait for my approval before writing it. The plan:
   - covers Phase 0 and Phase 1 only, in the order of section 10 of the spec,
     and starts with spikes S2 then S1;
   - names files, order, risks and proof, as the playbook's plan.md does, so
     that any engineer could implement it;
   - gives every step the verification the session can run itself (a test, a
     build, a smoke call), and says which steps need my login (D20) and which
     need my yes;
   - settles the items section 12 of the spec leaves for the plan: the Bicep
     module layout, the script steps of the tenant module and the custom role
     definitions; how the gate step reads the eval result after spike S6
     (D56); the trace evaluation (D58); the mutation and complexity thresholds
     and the tool choices for mutation testing, the import contracts and the
     BDD runner (D67 to D69), with versions verified in this session; the
     names and flags of the two forms of down; the catalogue and FAQ seed
     content and the 110 eval rows, which I sign (D67);
   - drafts the three golden conversations of spec section 5.11 for my
     signature, the ADR template for the six spikes, the REVIEW.md passes
     (D69), and the Foundry Skill install step (D65);
   - states which Phase 0 deliverables fall first if the two-week box is at
     risk, as D48 orders them.
   Commit the plan with status "awaiting the owner's acceptance". Then run
   /council docs/implementation-plan-phase-0-1.md, since the plan is a new
   artifact (D70); report the verdict, each dissent and the owner questions,
   and do not change the plan until I decide.
5. Run /cleanup and report. Apply only what I approve.
6. Write docs/journal/04-<stage>.md from the template, commit and push.
7. Give me a summary, any questions, and a handover prompt for the next
   session, then stop.
```

One later message changed the course of the session. While the first round of plan questions was
open, the owner dismissed it and pasted an external note recommending a central API gateway for
model and tool access in regulated industries, with six requirements, and wrote: "Please update
specs.md and implentation-plan.md. what do you recommend". To the session's first recommendation
the owner replied: "I think pass through is like providing central ai gateway. right. why you do
not recoomend it".

## 2. What happened, in order

1. Pre-flight passed. The azure-devops server timed out; nothing needed it.
2. Plan mode. The six artefacts were read in the order given. The owner received the summary of
   D56 to D70 and D47 to D55 and a reading order, accepted the spec and chose to continue.
3. The spec's status was set to accepted and CLAUDE.md gained its Architecture section.
4. Four research subagents ran in parallel (Python tooling; Azure control plane; Foundry operations
   and evaluation; GitHub release controls), about 1.5 million subagent tokens, each told to save
   only in the scratchpad. Meanwhile the session drafted the appendices and read-only checked the
   subscription, the provider registrations and the local tools.
5. A small script fetched each cited page and searched its text for the quotation. The quotations
   the spec and the plan rely on went through it.
6. The outline and four questions were put to the owner, who dismissed the dialog and sent the
   gateway note. The session stopped, researched it and recommended keeping the gateway on the
   model path with the comparison recorded. The owner asked why; the session gave its three
   reasons and what the pass-through would add; the owner chose the pass-through (D71).
7. Intent revision 14 and the spec revision for D71 were committed, in that order.
8. The plan was assembled from the drafts and committed with status "awaiting the owner's
   acceptance": 1,225 lines, 112 citation keys.
9. /council ran on the plan: six members, one rebuttal round, the chair. Accept with changes, no
   dissent, eleven owner questions.
10. The owner took the session's recommendation on all eleven. Intent revision 15 (D72 to D79),
    then the spec, then the plan were revised and committed.
11. /cleanup reported eight findings; the owner approved all eight.
12. A last pass over the plan's quotations found three that did not hold as written; corrected.
13. This entry, commit and push.

## 3. Owner decisions

| Question | Answer | Recorded in |
|---|---|---|
| Accept the revised spec | Yes | spec status line, commit fdbee4a |
| Continue to Stage 3 in this session | Yes | this entry |
| Gateway in front of salon-mcp as well as the models | Yes, as an MCP pass-through from Phase 1; bypass detected, not closed, until Phase 2 | intent D71 |
| Plan outline, and the session's recommendations on tools and routes | Proceed on all | plan sections 2, 3 and 9 |
| Council review 05, eleven questions | All of the session's recommendations | intent D72 to D79 |
| Cleanup findings | All eight | commit 8ba417d |
| Accept the implementation plan | Not yet asked; the revised plan awaits its principal review (D70) | section 7 |

## 4. Surprises and how they were handled

- **The council found three defects in the plan that would have stopped week 1.** The eval judge
  deployment reopened the model path during every gate run; the pull request check logged in to
  an environment restricted to `main`; the gateway had no telemetry sink, so its metrics and the
  D71 reconciliation would have produced nothing. All three were the session's own. Each is now a
  decision (D75, D72, D73).
- **The owner changed the design mid-interview.** The question dialog was dismissed, so nothing
  proceeded on assumed answers; the new question was researched and answered first.
- **Purging a gateway needs Contributor on it.** A purge-only role cannot purge, so the nightly
  identity now deletes only and `up` purges (D74). The same finding added a second teardown
  schedule: 09:00 to 22:00 would have cost about £42 a month for the gateway alone.
- **Trace evaluation contradicts content recording being off.** D58 was withdrawn for Phase 1
  rather than D46 weakened (D77).
- **The agentic-identity connection cannot be written in Bicep**, and no Azure Verified Module
  creates Foundry projects, guardrail policies or Workbooks. Several modules default to costly
  settings (Premium registry, Standard search, three replicas, 365-day retention).
- **Versions moved in a day.** The hosting protocol libraries are at stable 2.2.0; the spec's
  section 7 still pins 2.1.0b2. The plan pins 2.2.0 and says so; the spec row is still to correct.
- **The repository is public but unprotected today**: no ruleset, no environments, secret
  scanning and push protection off. The bootstrap fixes this in step 0.7.
- **The Retail Prices API returns four decimals**, so the spec's six-decimal rates cannot be
  reproduced; the plan uses four.
- **The citation check gave false alarms** on TOML table names inside code spans; the checker now
  ignores code. One research note relayed quotations through a summarising fetch tool; those the
  plan uses were rechecked against page text, and three in the plan were wrong and fixed.
- **Two stages and two intent revisions in one session.** CLAUDE.md says one stage per session;
  the owner chose to continue. The cost was a long session and a plan revised the day it was written.

## 5. What was produced

- [docs/spec-phase-0-1.md](../spec-phase-0-1.md): accepted; revised for D71 and for D72 to D79.
- [docs/intent.md](../intent.md): revisions 14 and 15, decisions D71 to D79.
- [docs/implementation-plan-phase-0-1.md](../implementation-plan-phase-0-1.md): the Stage 3
  artefact, 30 steps, six appendices, 117 references; awaiting acceptance.
- [docs/council/05-implementation-plan-phase-0-1-review.md](../council/05-implementation-plan-phase-0-1-review.md).
- [CLAUDE.md](../../CLAUDE.md): the Architecture section. Commands is still a TODO.
- [.mcp.json](../../.mcp.json): the Azure MCP server pinned to 2.0.5 (D64).
- [docs/research.md](../research.md): one line aligned with D71.
- This entry. No application code, package, infrastructure or Azure write.

Cleanup record: nothing was removed or merged. Six statements were corrected (four in the spec,
one in the plan, one in the research note) and the `.mcp.json` pin was changed; CLAUDE.md was kept
at 72 lines by the owner's choice.

## 6. Reusable lessons

- Check quotations by script, not by eye: fetch the page, strip the tags, search for the words.
  It caught three wrong quotations after the reading passes had missed them. Worth a place in the
  harness in Phase 0.
- A council on a new artefact finds contradictions between steps (0.7 against 0.8) that the
  research behind each step does not. D70's split, council for new and one reviewer for revised,
  held up.
- When a decision withdraws or amends an earlier one, search the artefacts for the earlier
  decision's number before committing. Four of the eight cleanup findings were such echoes.
- Draft the content the owner must sign while the research agents run; nothing in it depends on them.
- When the owner sends a design question during an interview, answer it before returning to the
  interview. When the owner asks why an option was not recommended, give the reasons and the case
  for the option, then ask again.
- Two lessons have now appeared twice (journal 03 and here) and are candidates for CLAUDE.md, for
  the owner to decide: run the citation check before every artefact commit; research briefs say
  "save nothing under the repository".
- Harness mechanism not needed this session (D70): plan mode's exploration and planning
  subagents, because the files were read directly, and the azure-devops MCP server, which failed
  to connect and was not missed.

## 7. Next session

Owner, before the next session:
- Read the revised plan: section 4 for the order of week 1, sections 9.3 and 9.4 for the roles and
  the gate, and Appendices A, E and F, which are the content you sign (D67).
- Read D71 to D79 in section 14 of docs/intent.md; D72 to D79 were taken as a block.
- Nothing to install yet. If a push is refused, run `gh auth switch --user ranaidrees`.

Next session (finish the Stage 3 gate; Stage 4 only after acceptance and in its own session):
1. Pre-flight as in this session. The Azure MCP server now starts at 2.0.5; confirm it connects.
2. One deep single-reviewer pass with a currency audit on the revised plan, and on the spec's
   changes for D71 to D79, written to docs/council/06-implementation-plan-phase-0-1-principal-review.md
   (D70). No council rerun.
3. Ask the owner for the decision on the plan. Changes that alter the intent are revision 16
   (D80 onwards), intent first.
4. On acceptance: set the plan's status to accepted and mark Appendices A, E and F signed.
5. Stage 4 starts at plan step 0.1.

Still open: the spec's section 7 pin of the hosting protocol libraries (2.1.0b2 against the plan's
2.2.0); the daily token quota after S6 (D39); the search tier after S4 (D27); the tool rate limit
after S1 (D71); the judge route after S6 (D75); the EU-region project (D63); the price of Defender
after its trial (D66); the items the plan marks unverified (Foundry User deploying a version, the
secret scanning setting by API on a Free personal repository, hosted agents with local
authentication disabled, the admin-connected judge in UK South); the six spikes; the accepted
risks in section 11 of the spec; the two CLAUDE.md candidates above; the chair's word limit,
which review 05 exceeded again at 2,315 words; the plan's step numbers skip 0.9 (the cost reading
became a sentence at the end of week 1), left as it is because review 05 cites the step numbers.
