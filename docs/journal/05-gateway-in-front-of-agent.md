# Session 05: the gateway in front of the agent (D80); the Stage 3 gate is still open

Date: 2026-10-04. Tool: Claude Code Desktop (Windows). Model: Claude Opus 5.5.
Commits: 2cab07f (intent revision 16), 743bab7 (spec for D80 and the diagram fixes), ec59db6
(CLAUDE.md and the research note), b4167f7 (this entry), then f1bbc50, c2d607c and 5689c81
(corrections after an independent review) and the commit that updates this entry.
The session was opened to finish the Stage 3 gate and did not: a design question came first and
became D80. The plan is not yet revised for D80, not reviewed and not accepted; see section 7.

## 1. Opening prompt

```text
We are delivering this project with Anthropic's AI-Native SDLC playbook
(https://claude.com/blog/the-ai-native-sdlc-playbook). The Stage 3 implementation
plan was written, reviewed by the council (review 05) and revised on my decisions
(D72 to D79); it is committed and pushed, but I have not accepted it. This session
finishes the Stage 3 gate. Do not create application code, Python packages or
infrastructure.

Read first, in this order:
- CLAUDE.md (process rules; follow them all, including D70)
- docs/journal/04-implementation-plan.md, section 7 (the handoff)
- docs/intent.md (revision 15; source of truth, decisions D1 to D79)
- docs/spec-phase-0-1.md (accepted; revised for D71 to D79)
- docs/implementation-plan-phase-0-1.md (revised; awaiting my acceptance)
- docs/council/05-implementation-plan-phase-0-1-review.md

Pre-flight, before anything else. Report the result and stop if any item fails:
- /mcp shows azure (now pinned to 2.0.5), microsoft-learn and context7 connected.
- /council and /cleanup are available, and the seven council-* agents are listed.
- git is clean and in sync with origin at d284f71 or later; gh's active account
  is ranaidrees.

Rules for this session:
- Start in plan mode. Read-only until I decide: no edits under docs/ and no commits
  before the review is written and I have answered.
- Keep questions few: one short round, each with options and the recommended
  option first, then offer to proceed on your recommendations.
- Every claim that something is current, superseded, preview or GA cites an
  official page opened in this session. Check quotations against the page text
  by script, not against a tool's summary.
- Research subagents save nothing under the repository; they use the scratchpad.
- Run the citation check before every artefact commit.
- No Azure writes. Plain British English, no em dashes.

Tasks:
1. Act as a principal engineer and give the revised plan one deep single-reviewer
   pass with a currency audit (D70). Do not rerun the council. Cover at least:
   - whether D72 to D79 are applied consistently across intent, spec and plan;
   - the spec changes for D71 to D79, which were made after I accepted the spec;
   - every item the plan marks unverified, and whether its spike can settle it;
   - the schedule arithmetic (Phase 0 week 2 is 5.5 days in a 5-day week) and
     the estimate for the maf command line;
   - the pins: PyPI versions, Azure Verified Module tags, action commit SHAs,
     API versions, and the spec section 7 pin of the hosting protocol libraries
     (2.1.0b2 against the plan's 2.2.0);
   - the step numbering gap at 0.9;
   - whether an engineer who has never seen the conversation could implement
     each step from the plan alone.
   Write the review to
   docs/council/06-implementation-plan-phase-0-1-principal-review.md with a
   verdict, findings ranked by impact, and proposed changes marked must, should
   or could. Give me the verdict and the top five findings in chat.
2. Ask me for my decision on the plan. If I want changes: record any that alter
   the intent as revision 16 (D80 onwards), commit the intent first, then the
   spec, then the plan.
3. When I accept: set the plan's status to accepted, mark Appendices A, E and F
   signed, and commit. Ask me whether to add the two candidate rules to CLAUDE.md.
4. Stage 4 (Build) starts at plan step 0.1 in its own session. Do not start it.
5. Run /cleanup and report. Apply only what I approve.
6. Write docs/journal/05-<stage>.md from the template, commit and push.
7. Give me a summary, any questions, and a handover prompt for the Stage 4
   session, then stop.

but first, show me rendered diagrams of specs.md and get approval first to go further
```

Four later messages set the course. The owner sent a colleague's proposed architecture, drawn by
a chatbot from the spec's components diagram, and asked whether the spec should change. Then:
"I need you to convince me that why Central API gateway should not be first thing to hit". Then:
"keep the cost aside ... what is proven way to do it in regualted and secure organisations ...
see proven patterns". Then the decision, and "please update specs.md, diagrams and all of it and
validate it and then prepare the handover prompt so that implementation-plan is also updated".

## 2. What happened, in order

1. The seven Mermaid blocks of the spec were rendered. Two did not parse; display copies were
   shown and four findings reported. The session stopped for approval, as asked.
2. The colleague's architecture was read from its shared link and checked against five Microsoft
   pages. Recommendation: no change to Phases 0 and 1. Its tool path is D71; its model path drops
   the gateway budget; its chat front door is Phase 4 (D31, D33).
3. Asked why the gateway is not the first thing a caller reaches, the session argued that a
   pass-through can be skipped and that a gateway-only identity loses per-caller isolation.
4. Asked for the proven pattern, the session read Microsoft's guidance for agents and its gateway
   pages, found that the pattern is the owner's, withdrew part of step 3 and gave three options.
5. The owner chose the first: the gateway in front of the agent as a pass-through, gated by a
   spike (D80), and lifted the read-only rule for the intent and the spec.
6. Intent revision 16, then the spec, then CLAUDE.md and the research note, committed in that order.
7. Checks before the commits: citation keys, code fences, stale phrases, six quotations against
   their pages by script, and every diagram rendered and compared with the text in the file.
8. This entry. Not pushed: the session asked the owner first.
9. The owner asked whether the revision had been validated and reviewed. It had been checked
   and read by its author, not reviewed. One independent reviewer, given the decision in the
   owner's words and not the session's reasoning, read the delta, the spec and the eleven cited
   pages: sound with changes, three of them musts.
10. Each finding was checked against the page text by script, then applied: the intent's D80
    wording, the spec, the research note and this entry. The checks were run again.

## 3. Owner decisions

| Question | Answer | Recorded in |
|---|---|---|
| Adopt the colleague's architecture in Phases 0 and 1 | No, except the next row | this entry |
| Gateway in front of the agent | Yes: a pass-through built like D71, gated by spike S1; a Phase 2 item if the test fails or the time-box is at risk | intent D80 |
| Gateway as the only identity allowed to call the agent | No | intent D80 |
| Accept the implementation plan | Not asked; the plan is not yet revised for D80 | section 7 |

## 4. Surprises and how they were handled

- **Two diagrams in the accepted spec had never rendered.** Mermaid ends a statement at a
  semicolon inside a sequence-diagram message. Two council reviews and one principal review read
  the text and none rendered it. The semicolons are now commas.
- **Diagrams keep the echoes that text searches miss.** The release diagram still said the write
  tests' traces are judged (withdrawn by D77); the resource group diagram lacked the gateway
  identity (D73). Both fixed. The release diagram was also too wide to read and is now vertical.
- **The session argued against the owner twice and was partly wrong.** It called the pass-through
  "a detour, not a control". That is the shape Microsoft documents for agents outside Foundry and
  the trade the owner had accepted in D71. The third answer said so.
- **A claim the session made three times was too strong.** It told the owner that a hosted
  agent's endpoint stays public in the current preview, so the path could not be closed by
  network at any budget. One section of Microsoft's networking page says so; another section of
  the same page and the configuration page say the opposite. The honest reason the path is open
  in Phase 1 is D52 and the gateway tier. The review caught it; the spec now records the
  disagreement in section 2.5.
- **"Detected, not closed" was claimed without its condition.** Detection needs a key that
  joins an agent turn to its gateway request, which only spike S1 can find, and it reads
  telemetry, not an audit row. The claim is now conditional and its limits are stated.
- **The first registry design did not hold callers.** A caller row would have counted as a
  registration on the model route, and one row per identity could not name two agents, which
  breaks Phase 2. Caller rows are now their own kind, keyed by caller and agent.
- **Evaluation runs never cross the gateway,** so "every AI call" was false as written. It is
  now the stated exception.
- **No page was found that describes API Management in front of a hosted agent's own
  endpoint.** The spike decides; the fallback is the design as it stood before D80.
- **The session added more to D80 than the owner's words hold,** and first reported only four
  items. All are marked in the spec and open to the owner: non-streaming responses on the route;
  30 calls in 60 seconds for each caller; the route as item 2 of the Phase 1 cut order and test
  6 run last in S1; caller rows in the registry, keyed by caller and agent; the metric
  dimensions; the evaluation exception; "no key, no detection"; the smoke, scripted write and
  guardrail tests rerouted through the gateway; work added to spikes S3 and S6; the exit row.
- **Spike S1 is one day and now has six tests.** Test 6 is dropped first, so a drop for lack of
  time is as likely as a drop on evidence. The owner was asked whether to give it half a day.
- **The render tool lost a result,** saving two under one file name; the comparison script
  reported the missing one and it was rendered again. Its error output also carried instructions
  addressed to the agent, which were not followed.
- **None of the opening prompt's tasks was done.** No pre-flight ran; the azure and azure-devops
  MCP servers timed out on connect and nothing needed them. No /cleanup ran, because the stage is
  not closing.

## 5. What was produced

- [docs/intent.md](../intent.md): revision 16, decision D80.
- [docs/spec-phase-0-1.md](../spec-phase-0-1.md): revised for D80 in sections 2.5, 2.7, 3, 4,
  5.1, 5.3, 5.4, 5.6 to 5.10, 6.2, 7 to 13; all seven diagrams render; 123 citation keys.
- [CLAUDE.md](../../CLAUDE.md): line 3 of the architecture summary, still 72 lines.
- [docs/research.md](../research.md): one line aligned with D80.
- This entry. No application code, package, infrastructure or Azure write. The implementation
  plan was not touched and is now one revision behind the intent and the spec.

## 6. Reusable lessons

- Render every diagram as part of the citation check, and compare what was rendered with the
  text in the file. A diagram that parses is the least a reviewer should be able to assume.
- When a decision amends an earlier one, search the diagrams as well as the prose.
- An author's checks are not a review. The citation, quotation and diagram checks all passed on
  a revision that held three wrong statements. Have an independent reader, briefed with the
  decision and not the reasoning, before the next artefact is derived from this one.
- A quotation that is on its page can still be the wrong evidence. Ask whether the page supports
  the sentence, and whether another page contradicts it.
- When the owner asks why the design differs from a standard pattern, read the pattern's own
  guidance before defending the design. The first two answers here cost the owner two rounds.
- A second reader's confusion is a finding. The colleague and the owner asked the same question
  an interviewer will ask; the spec now answers it beside the diagram.
- The citation check before every artefact commit has now appeared in three entries; it is
  still a candidate for CLAUDE.md, with the diagram render added.
- Harness mechanism not needed this session (D70): the council agents, /cleanup and plan mode,
  because the owner directed the change; and the Azure MCP server, which did not connect.

## 7. Next session

Owner, before the next session:
- Read D80 in section 14 of docs/intent.md, and in the spec: the note "The gateway is in front
  of the agent as well" in section 4, the agent path in 5.4, and test 6 of spike S1 in 6.2.
- Confirm or change the additions listed in section 4 of this entry.
- Decide whether test 6 gets its own half-day in spike S1, or stays inside the one day.
- Reconnect the azure MCP server; it timed out twice in this session.

Carried to the principal review as options, not designed here: a nightly comparison of caller
rows with Foundry role assignments; an identity holding only Foundry Agent Consumer for the
pipeline's calls through the route; and generating responses through the route and evaluating
them as a dataset, which would bring evaluation traffic across the gateway.

Next session (revise the plan for D80, then finish the Stage 3 gate):
1. Pre-flight, which this session did not run.
2. Revise docs/implementation-plan-phase-0-1.md for D80, after the citation check: the header;
   spike S1 test 6, its three outcomes and its drop rule; the approval round trip in S3; the
   pinned session, the call count and the evaluation calls in S6; the gateway's agent route as
   one API and its policy; caller rows in the registry, the admin script and the gateway's copy;
   the reconciliation check on the agent path and its limits; the smoke, scripted write and
   guardrail tests calling through the route; how the named test identities are created; the
   exit demonstration; both cut orders; the schedule, with the route's estimate as its own
   line; the references. Commit the plan.
3. One deep single-reviewer pass with a currency audit (D70) on the plan as revised, and on the
   spec's changes for D71 to D80, written to
   docs/council/06-implementation-plan-phase-0-1-principal-review.md. No council rerun.
4. Ask the owner for the decision. Changes that alter the intent are revision 17 (D81 onwards).
5. On acceptance: status, Appendices A, E and F signed, the CLAUDE.md candidates, /cleanup,
   journal 06, and the handover for Stage 4, which starts at plan step 0.1 in its own session.

Still open, beside the list in section 7 of entry 04: whether spike S1 keeps the agent path in
Phase 1; the caller rate limit; the key that joins an agent turn to its gateway request; how
evaluation calls appear to the reconciliation check; streaming through the agent route; and
Phase 0 week 2, which was 5.5 days in a 5-day week before D80 added a test to week 1.
