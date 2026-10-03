# Session 02: Stage 2, council gate on the spec for Phases 0 and 1

Date: 2026-10-03. Tool: Claude Code Desktop (Windows). Model: Claude Opus 5.5.
Commits: bf23c9b (council review 03), 1c3f4e9 (intent revision 12), b52b10e (spec revision),
6299a38 (CLAUDE.md rules), 5e23e89 (cleanup), and the commit that adds this entry.
The spec is revised but not yet accepted by the owner; see section 7.

## 1. Opening prompt

```text
We are delivering this project with Anthropic's AI-Native SDLC playbook
(https://claude.com/blog/the-ai-native-sdlc-playbook). Stage 2 (Design) has
produced a draft spec that is committed and pushed but not yet reviewed or
accepted. This session is the Stage 2 gate: the council review of
docs/spec-phase-0-1.md, my decisions on it, and a revised spec. Do not create
application code, Python packages, infrastructure or the implementation plan.

Read first, in this order:
- CLAUDE.md (process rules; follow them all)
- docs/journal/01-spec-phase-0-1.md, section 7 (the handoff from the last session)
- docs/intent.md (revision 11; source of truth, decisions D1 to D36)
- docs/spec-phase-0-1.md (draft; sections 2.4 and 12 hold questions for me)
- docs/council/02-intent-review.md (the debates the spec answers)

Pre-flight, before anything else. Report the result and stop if any item fails:
- /mcp shows azure, microsoft-learn and context7 connected.
- /council and /cleanup are available, and the seven council-* agents are listed.
- git is clean and in sync with origin at b18cdae or later; gh's active account
  is ranaidrees.

Rules for this session:
- Start in plan mode. Ask, do not assume.
- Every design claim cites an official document or sample with a URL you opened
  in this session. Paste quotations from the source; do not retype them.
- This stage needs no Azure writes. Read-only lookups are fine.
- The dev budget is 40 pounds a month (D21). When you quote a price, give the
  rate and a table of usage, not a single monthly figure.
- Plain British English, no em dashes.

Tasks:
1. Run /council docs/spec-phase-0-1.md. Report the verdict, each dissent and the
   owner questions. Do not change the spec until I decide.
2. Interview me, with options, a recommendation and a source for each question:
   - the council's owner questions;
   - the seven questions in section 12 of the spec and the proposals in 2.4;
   - whether a session may run the teardown without asking, given that
     CLAUDE.md says a dev write that deletes anything waits for me;
   - whether "research before the interview" moves into CLAUDE.md, since it
     appears in both journals.
   If two of my answers conflict, show me the conflict and ask again.
3. Record answers that change the intent as revision 12 (D37 onwards) and
   commit that first. Then revise the spec and commit it. Mark the spec as
   accepted only if I say so.
4. If I accept the spec, fill the Architecture section of CLAUDE.md from it and
   keep the file under one page.
5. Try once more to price the items the spec lists as not verified (model
   tokens, hosted agent compute, logs). If they still cannot be found, leave
   them as unknown.
6. Run /cleanup and report. Apply only what I approve.
7. Write docs/journal/02-spec-gate.md from the template, commit and push.
8. Give me a summary, any questions, and a handover prompt for the next session
   (Stage 3, the implementation plan), then stop.
```

Two later messages changed the course of the session. While the council ran: "you have created
intent.md. when you are going to create specs.md". After dismissing the fourth round of questions:
"Once you have intent.md and specs.md are all done according to AI native SDLC by anthropic then
provide me handover session for new session and claenup. I will be happy with your recommendation
for all", followed by the list of items still open.

## 2. What happened, in order

1. Pre-flight passed. Entered plan mode, read the five documents, and the plan was approved.
2. Ran `/council`: six reviews in parallel, anonymised, six rebuttals, then the chair. Thirteen
   subagent runs, about 1.6 million subagent tokens. Verdict: Accept with changes, unanimous.
3. While the council ran, priced the three unverified items and opened sources for the interview.
   Task 5 was moved ahead of the interview, as the approved plan said.
4. Checked the council's technical claims against the documentation before asking anything.
5. Interview: three rounds of four questions were answered.
6. Stopped on a conflict between two answers (section 4) and put it back in a fourth round.
7. The owner dismissed the fourth round, then asked for the session's recommendation to stand on
   everything still open.
8. Committed the council review, intent revision 12 (D37 to D55) and the revised spec, after
   checking 71 citation keys and 22 new quotations.
9. Committed the three CLAUDE.md rules that had been decided (D40, D41, D55).
10. Ran `/cleanup`: eight findings; the owner approved the six corrections.
11. Asked for acceptance. The owner answered "Not yet". The spec's status and CLAUDE.md's
    Architecture section were left as they were.
12. Wrote this entry and pushed.

## 3. Owner decisions

| Question | Answer | Recorded in |
|---|---|---|
| Nightly teardown; full rebuild | The gateway only; the approved image only | intent D37, D38 |
| Token quota | 150,000 tokens a day, the owner's own figure, to keep cost at a minimum | intent D39 |
| Session teardown; approval credential | Gateway teardown without asking; a written rule and an accepted risk | intent D40, D41 |
| Gate additions; evidence | The four cheap ones; a GitHub Release for each promotion | intent D42, D43 |
| Eval gate | The Action plus scripted tests; thresholds measured first | intent D44 |
| Retrieval; agent additions | Keyword only; the four small ones | intent D45, D46 |
| Everything else open | The session's recommendation stands | intent D47 to D55 |
| Cleanup findings | All six corrections | commit 5e23e89 |
| Accept the revised spec | Not yet | this entry |

## 4. Surprises and how they were handled

- **The prices were there all along.** The pricing tool had failed in session 01. The public
  Azure Retail Prices API answered directly, and all three unpriced items are now in the spec.
- **The council's main finding was a collision inside the spec.** Deleting everything nightly
  broke the release chain, rollback and the monthly quota. All six members backed removing only
  the gateway, which bills for existing and nothing else does.
- **A session can approve its own gate.** A token with the `repo` scope can approve a deployment,
  and this session's token has it. Recorded as a written rule and an accepted risk (D41).
- **Two answers conflicted.** A quota of 150,000 tokens a day is smaller than the estimated size
  of one gate run under the eval answers. The session showed the conflict and asked again. The
  owner then delegated, so D39 keeps 150,000 and spike S6 measures a gate run.
- **Too many questions.** Sixteen questions in four rounds, each with a long briefing, was more
  than the owner wanted. The fourth round was dismissed.
- **The artifact name confused.** The owner asked when "specs.md" would be created; the spec
  already existed under the name D8 gives it.
- **Quotations from a summarising fetch cannot be trusted.** Each GitHub quotation was checked
  against the page text before it went into the spec.
- **Recommendations were recorded unseen.** D47 to D55 were written at the owner's instruction
  without the owner seeing each one. That is said in the intent and in the report to the owner.
- **Acceptance was withheld.** No reason was given. Nothing was marked accepted.
- **The chair could not keep to 900 words** with six members and thirteen questions.

## 5. What was produced

- [docs/council/03-spec-phase-0-1-review.md](../council/03-spec-phase-0-1-review.md): the review.
- [docs/intent.md](../intent.md): revision 12, decisions D37 to D55.
- [docs/spec-phase-0-1.md](../spec-phase-0-1.md): revised, awaiting acceptance; 71 sources.
- [CLAUDE.md](../../CLAUDE.md): three rules added. Architecture is still a TODO.
- [docs/research.md](../research.md): four statements corrected by the cleanup.
- This entry. No code, infrastructure, implementation plan or Azure write.

Cleanup record: nothing was removed or merged. Four statements in docs/research.md and two in
docs/intent.md were corrected to match revision 12.

## 6. Reusable lessons

- Run the research while the council runs. It cost no extra time.
- Ask the Retail Prices API directly when a pricing tool fails.
- Check every quotation against the page text, not against a tool's summary of the page.
- Ask fewer questions. Offer "take my recommendations for the rest" after the first round, and
  keep each briefing to a few lines. For the prompt next time: say how many rounds are welcome.
- Name the artifact files in the first status message, so the chain is visible to the owner.
- "Research before the interview" appeared twice and is now in CLAUDE.md (D55). Two more lessons
  have now appeared twice, in journal 01 and in this session's prompt: quote prices as a rate with
  a table of usage, and paste quotations. They are candidates for CLAUDE.md; the owner decides.

## 7. Next session

Owner, before the next session:
- Read docs/spec-phase-0-1.md, and D47 to D55 in section 14 of docs/intent.md, which were recorded
  on your instruction without your seeing each one.
- Decide whether you accept the spec, or what you want changed.
- Nothing to install. If a push is refused, run `gh auth switch --user ranaidrees`.

Next session (finish the Stage 2 gate, then Stage 3 only if the spec is accepted):
1. Pre-flight as in this session, and read this section.
2. Ask the owner for the decision on the spec. If changes are wanted, make them, record any that
   alter the intent as revision 13 (D56 onwards) and commit. Do not rerun the council unless the
   owner asks.
3. On acceptance: set the spec's status to accepted, fill the Architecture section of CLAUDE.md
   from it in about eight lines, keep the file under one page, and commit.
4. Stage 3, docs/implementation-plan-phase-0-1.md, starts only after step 3. CLAUDE.md says one
   stage per session, so ask the owner whether to continue or stop.

Still open: the daily token quota, which returns after spike S6 measures a gate run (D39); the
search tier, after spike S4 (D27); the six spikes, which belong to Phase 0; the accepted risks in
section 11 of the spec; two proposals from the cleanup (replace section 8 of docs/research.md with
a pointer if it drifts again; raise the chair's word limit).
