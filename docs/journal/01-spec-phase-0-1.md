# Session 01: Stage 2, spec for Phases 0 and 1

Date: 2026-10-03. Tool: Claude Code Desktop (Windows). Model: Claude Opus 5.5.
Commits: 7ab4b7b (intent revision 11), c8b8a4d (spec), e13557b (cleanup), and the commit that adds
this entry. The council review of the spec was not run; see section 7.

## 1. Opening prompt

```text
We are delivering this project with Anthropic's AI-Native SDLC playbook
(https://claude.com/blog/the-ai-native-sdlc-playbook). Stage 0 (harness) is
complete and committed. This session is Stage 2, Design: produce
docs/spec-phase-0-1.md only. Do not create application code, Python packages,
infrastructure or the implementation plan.

Read first, in this order:
- CLAUDE.md (process rules; follow them all)
- docs/journal/00-harness.md, section 7 (the handoff from the last session)
- docs/intent.md (revision 10; source of truth, decisions D1 to D18)
- docs/council/02-intent-review.md (twelve owner questions; 1 to 3 are answered
  by D13, D14 and D18)
- docs/research.md

Pre-flight, before anything else. Report the result and stop if any item fails:
- /mcp shows azure, microsoft-learn and context7 connected.
- /council and /cleanup are available, and the seven council-* agents are listed.
- git is clean and in sync with origin; gh's active account is ranaidrees.
- Report whether the GitHub repository is public or private.

Rules for this session:
- Start in plan mode. Ask, do not assume.
- Reuse before build. Every design claim cites an official document or sample
  with a URL you opened in this session (Microsoft Learn MCP, Context7, web).
- This stage needs no Azure writes. Read-only lookups are fine.
- Plain British English, no em dashes.

Tasks:
1. Interview me on everything the spec depends on that is still open. At least:
   - Council review 02, questions 4 to 12.
   - Intent questions 5, 6 and 7, and the data store (D11).
   - The contradiction between D6 (prod credentials only in GitHub Actions) and
     D13 (sessions may write to prod with my approval).
   - How the approval rules in D13 and D14 are enforced: hooks, Azure RBAC, or
     written rule only.
   - Gateway region and tier, given that APIM v2 cannot currently be created in
     UK South.
   - Phase 1 length (two weeks with cuts, or four weeks) and where the stop
     line is.
   - Who may call the Phase 1 agent, and the identity at each hop.
   Give options, a recommendation and a source for each question. Record my
   answers as new decisions in docs/intent.md (revision 11) and commit that
   before writing the spec.
2. Verify the council's technical findings against current documentation before
   designing around them: hosted agents reaching models without APIM, APIM
   tiers, regions and prices, the limits on gateway metric dimensions, AI
   Search tier limits, and hosted agent availability in UK South and UK West.
3. Propose the outline of docs/spec-phase-0-1.md and wait for my approval. It
   must include a clarifications section, the identity flow, the time-boxed
   Phase 0 spikes with go or no-go criteria, a cost estimate, preview
   components with fallbacks, and exit criteria that show each control working,
   not only present. Cover the remaining Phase 0 deliverables and the Phase 1
   MVP, nothing later.
4. Write the spec and commit it.
5. Run /council docs/spec-phase-0-1.md. Report the verdict, each dissent and
   the owner questions. Do not change the spec in response until I decide.
6. Run /cleanup and report. Apply only what I approve.
7. Write docs/journal/01-spec-phase-0-1.md from the template, commit, push,
   show me a summary with any questions, then stop.
```

A second request arrived after the spec was written: "I need a handover prompt for new session and
cleanup and consolidation. what has been created and what is being created". Asked whether the
council should run first, the owner chose to hand over and run it in the next session.

## 2. What happened, in order

1. Pre-flight: MCP servers, skills, agents, git, gh account and repository visibility. All passed.
2. Entered plan mode and read the five documents in the order given.
3. Checked the council's technical findings against current documentation before the interview,
   so each question came with a verified recommendation. Task 2 was done ahead of task 1.
4. Interview: six rounds and three follow-ups, 27 questions, each with options, a recommendation
   and a source.
5. Stopped on two answers that could not both hold (use the owner's Desktop login for prod; block
   prod with Azure RBAC). The owner resolved it in their own words: least friction.
6. Stopped again to explain how a gateway billed by the hour produces £113 a month, after which
   the owner set a ceiling of £40 for the whole dev environment.
7. The owner changed the scope: dev is the only environment.
8. Corrected two statements made earlier in the interview once further pages were opened.
9. The plan, holding decisions D19 to D36 and the spec outline, was approved.
10. Recorded the decisions as intent revision 11 and committed.
11. Opened the remaining sources, wrote the spec, checked that every citation key resolved, and
    committed.
12. Ran /cleanup: eight findings, all approved and applied in one commit.
13. Wrote this entry and pushed.

## 3. Owner decisions

| Question | Answer | Recorded in |
|---|---|---|
| D6 against D13; enforcement of D13 and D14 | The owner's login throughout; written rules only; no hook, alert or lock | intent D20 |
| Environments and the release gate | Dev only; candidate, eval gate and approval inside dev; single approver | intent D19 |
| Budget | £40 a month for all of dev, as the cumulative ceiling on D9 | intent D21 |
| Gateway | API Management Basic v2 in UK West, on demand; spike both model paths | intent D22, D23 |
| Tenancy and provisioning | One Foundry project per tenant; admin script with an audit record | intent D24, D25 |
| Data and knowledge | Table Storage; index per tenant on the Search Free tier; synthetic data only | intent D26, D27, D29 |
| Teardown and evidence | Delete everything; keep evidence in a persistent group | intent D28 |
| Schedule, scope, Langfuse | Phase 0 two weeks, Phase 1 three weeks with cuts, stop line at Phase 3; guardrail added; Langfuse off by default | intent D30, D31, D32 |
| Callers, models, evals, harness | Named test identities; Global Standard in UK South; one eval harness; hooks, skills and verifier stay | intent D33 to D36 |
| Council in this session | No, hand over first | this entry |
| Cleanup findings | All eight | commit e13557b |

## 4. Surprises and how they were handled

- **Answers went against the recommendations and against each other.** Asked again with the
  conflict stated. D20 is recorded as an accepted risk, and the council's first debate in review
  02 stays open.
- **"Dev only" removed the dev to prod story.** Microsoft documents a release flow that keeps the
  gate in one environment: pin the served version, deploy a candidate, test a session pinned to
  it, promote after approval. The spec uses it.
- **A monthly price misled.** "£113 a month" read as a flat fee. Restated as a price per hour with
  a table of hours against cost.
- **The documentation disagrees with itself** on whether keyless access works on the Search Free
  tier. Made a spike, with a priced fallback.
- **Three facts the council did not have** changed the options: Foundry's own AI Gateway, a durable
  checkpointer built into Foundry hosting, and current models being Global Standard only in UK
  South.
- **Two controls interact.** The eval Action needs a judge model deployment in the project, which
  could reopen the gateway bypass. Found while writing; spike S6 now runs after S1.
- **Some prices could not be retrieved**: model tokens, hosted agent compute and logs. The spec
  says so and measures them in Phase 0 week 1.
- **Retyped quotations were wrong twice.** /cleanup caught both before the council saw them.
- **CLAUDE.md still says a dev write that deletes anything waits for the owner.** With daily
  teardown, a session that runs `down` must ask every time. Left for the owner.

## 5. What was produced

- [docs/intent.md](../intent.md): revision 11, decisions D19 to D36.
- [docs/spec-phase-0-1.md](../spec-phase-0-1.md): draft for council review, 61 sources, seven
  questions for the owner in section 12.
- [CLAUDE.md](../../CLAUDE.md) and [docs/research.md](../research.md): aligned with revision 11 by
  the cleanup.
- This entry. No code, infrastructure, implementation plan or Azure resource.

## 6. Reusable lessons

- Verify the reviewers' facts before the interview. It changed three sets of options here.
  Session 00 recorded the same lesson ("research before the interview"), so it is a candidate for
  CLAUDE.md. Not added; the owner decides.
- When two answers conflict, show the conflict and its consequence and ask again. Do not choose.
- Quote prices as a rate with a table of usage, never as a single monthly figure.
- Paste quotations from the source and check them in /cleanup.
- Run /cleanup before the council, so reviewers do not spend effort on wording.
- For the prompt next time: state the monthly budget, say whether the council runs in the same
  session, and ask for the handover prompt as a deliverable.

## 7. Next session

Owner, before the next session:
- Read section 12 of the spec (seven questions) and section 2.4 (proposals not yet put to you).
- Decide whether a session may run `down` without asking, given CLAUDE.md's "deletes nothing" rule.
- Nothing to install. If a push is refused, run `gh auth switch --user ranaidrees`.

Next session (Stage 2, council gate on the spec):
1. Pre-flight as in this session, and read this section.
2. Run `/council docs/spec-phase-0-1.md`. Report the verdict, each dissent and the owner
   questions. Do not change the spec until the owner decides.
3. Interview the owner on the council's questions and on the spec's section 12. Record answers
   that change the intent as revision 12 (D37 onwards) and commit before revising the spec.
4. Revise the spec, commit, run /cleanup, write the journal, push, stop.
5. Only when the owner accepts the spec: fill the Architecture section of CLAUDE.md. Stage 3, the
   implementation plan, is a separate session.

Still open: the unpriced items in section 2.3 of the spec; the six spikes, which belong to Phase 0
and not to the next session; the accepted risks in section 11.
