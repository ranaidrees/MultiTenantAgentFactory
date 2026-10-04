# Principal engineer review: docs/implementation-plan-phase-0-1.md

Date: 2026-10-04. Artifact reviewed: docs/implementation-plan-phase-0-1.md as revised for D80 to D82 at commit 37e585e, and re-read at commit 65d6849 after the alignment corrections listed in section 14.
Method: one deep single-reviewer pass with a currency audit, not a council run (D70). The owner asked for it before deciding on acceptance. The owner's decisions on it are D83 to D88 in docs/intent.md revision 18: the first option of section 13 in each case, except D87, where the owner chose to extend the boxes.
Reviewer lens: AI platform engineering on Azure: Microsoft Foundry hosted agents, API Management, identity, evaluation, release engineering and delivery planning.
Reads: docs/intent.md revision 17 (D1 to D82), docs/spec-phase-0-1.md, docs/council/05-implementation-plan-phase-0-1-review.md, docs/journal/05-gateway-in-front-of-agent.md, CLAUDE.md.

How the pass was made. The session that revised the plan for D80 did not review its own work alone, which is the lesson of journal entry 05. A fresh reviewer, given the documents and the owner's list of what to cover and none of the session's reasoning, read all three documents and reported fourteen findings. The session then checked every one: 62 quotations from the files against the cited lines, and every quotation from an external page against the page text, both by script, with no failure. The currency audit was run by script against PyPI, the Microsoft container registry, npm, GitHub and the Azure resource providers. The session added the three-way alignment check the owner asked for and two findings of its own. Where the reviewer's estimate differs from the session's, which wrote the estimates, both are given. Line numbers are at commit 65d6849. Where something could not be verified the text says "unverified".

## 1. Verdict

**Accept with changes.** The step structure, the sums and the three corrections to D80 hold, and the pins are current. But the plan cannot yet be followed from step 0.4 onwards without the engineer inventing things: Phase 0 uses resources that later steps create, a clean clone cannot reach a served agent in one pipeline run, the gateway's metrics would emit nothing, and two test set-ups the exit criteria depend on do not exist. Every defect is a correction to the plan's text or a small decision for the owner. None is a redesign, and the spec stands.

The eight changes marked must should be made before any code is written from the plan. Five of them need a decision from the owner (section 13).

## 2. Findings ranked by impact

Findings F1 to F14 are the independent reviewer's, verified. F15 and F16 are the session's own.

**F1. Phase 0 steps use things that later steps create. Must.** (Reported as F2.)
Step 0.4, the first spike, needs a hosted echo endpoint with a public address (plan 185, "--target <echo url>"), and S1 test 5 needs salon-mcp behind the gateway, but Container Apps and the `registry` table first appear in step 0.10 (plan 259). Step 0.5 already needs "named values for the registry copy" (plan 194) and step 0.9 "writes a row in the `caller` partition of the `registry` table" (plan 244). Step 0.4 runs `maf agent version create`, but `agent.py` is a file of step 1.4 (plan 363). Spike S5's full rebuild must register a new agent identity, which is step 1.2's code. Week 1 uses `maf cost` (plan 228), which no step builds.
Why: spike S2 cannot start as written, on the second day of Phase 0.
Change: move the Container Apps environment, the echo app and `env-storage.bicep` into step 0.3; name `agent.py` and a minimal `maf tenant register` in step 0.4; give `maf cost` a step.

**F2. A clean clone cannot reach a served agent in one pipeline run. Must, owner decision.** (Reported as F1.)
The registry's agent row and caller rows wait for the agent to exist (plan 484 and 489), and the registry is "written only by the admin script" (spec 328). The agent's identity appears only when the first agent is created: "The project also gets an agent blueprint and agent identity when its first agent is created" (hosted agent permissions page). So the first pipeline run creates the agent, and its own smoke test and gates then meet 403 on the model route and on the agent route, because nothing has registered the identity and the pipeline cannot write the registry. The stranger test of spec 9.2, "one setup command and one pipeline run produce a served agent", becomes setup, a failed run, the admin script, and a second run.
Change: the owner chooses how the first registration happens (section 13, D83), and step 0.10 states the order `up` uses on a clean clone and on a full rebuild.

**F3. The gateway's metric policies will emit nothing. Must.** (Reported as F4.)
The plan configures "an Application Insights logger and a diagnostic at 100 per cent sampling" (plan 194) and nothing more. Council review 05 had converged on two more settings that did not reach the plan (review 05, line 34). The pages require three things: "Enable custom metrics with dimensions in Application Insights" (emit-metric page); the `"metrics": true` property on the diagnostic (API Management and Application Insights page); and, for the MCP pass-through, "set the Number of payload bytes to log setting for Frontend Response to 0" (expose an existing MCP server page).
Why: S1 criterion 1, both metric policies, the Workbook, `maf check tokens` and four exit rows depend on these metrics.
Verified since: the pinned Azure Verified Module exposes `metrics: true` in `serviceDiagnostics`. Unverified: a Bicep property for "custom metrics with dimensions"; the template reference for `Microsoft.Insights/components` shows none, and the page describes a portal step.
Change: add `metrics: true` and the payload setting to step 0.5; set the Application Insights flag in the bootstrap (step 0.2), by a non-portal route if S1 test 4 finds one and otherwise as one recorded portal step, which is acceptable because the persistent group is created once.

**F4. The 2,000-token test identity is never created. Must, owner decision.** (Reported as F3.)
D39 relies on "A test identity with a quota of 2,000 tokens" (intent 241); the exit row is "shown with the low-quota test identity" (spec 1427); steps 0.6 and 1.5 use it (plan 203, 373). D82 creates exactly three test identities, which "hold nothing else" (intent 297), and an agent row "is the only kind the model and tool routes accept" (spec 625). No step creates a fourth identity or gives any identity a low-quota agent row. The row format has no quota field (plan 484), yet the judge's row is "with its own quota" (plan 488) with no figure. A quota per row is possible: for `token-quota`, "Policy expressions are allowed" (llm-token-limit page).
Why: "the gateway negative test" is never cut, and it is an exit row.
Change: the owner decides who the identity is (D84); add `quota` to the agent row in spec 5.3 and plan 9.2; `llm.xml` reads it by expression; give the judge's figure as a deploy parameter set from spike S6.

**F5. The nightly workflow cannot run its guardrail test, and may not tear down. Must.**
The guardrail negative test calls the agent "so in the nightly workflow it runs among the checks" (plan 404), but that workflow runs "in environment `nightly-teardown` with the teardown identity" (plan 260), the test identities are federated "for `:environment:dev` only" (plan 167), and the teardown identity has neither the role nor a caller row. No identity in that job can call the agent. Separately, the checks run "then `maf down`" and the new `maf check tokens` "fails the run when it passes that figure" (plan 442); nothing says the delete still runs after a failed check, so a failed check can leave the gateway up at £3.72 a day.
Change: run the guardrail test in a second job of `nightly.yml` in environment `dev`, as the consumer-only identity (see option (b) in section 7); make the delete step unconditional.

**F6. The agent API omits the conversation operation the approval round trip needs. Must.**
Step 0.9 defines "six operations that mirror the agent endpoint" (plan 242): the Responses call and five session operations. The confirmation interrupt depends on a conversation: "State is persisted by `FoundryCheckpointSaver` and keyed by the `conversation.id` from the Responses request" (Microsoft's human-in-the-loop sample), and the sessions page says to create it "via `POST .../endpoint/protocols/openai/conversations`". A caller that creates a conversation through the route would get 404 from the gateway's own API definition, and step 0.12's rule would then drop the route (plan 278) for an omission in configuration, not on evidence.
Change: add the conversations operation in step 0.9; state how callers thread turns; exclude a gateway 404 from the drop rule. Unverified: whether a hosted agent accepts a conversation id the client chose without that call.

**F7. The five baseline runs do not fit the daily quota. Must, owner decision.**
One gate run is "estimated, not measured, at about 161,000 tokens" (intent 241). D76 raised the quota to 3,000,000 for the S6 spike day only (intent 286), and moved the five baseline runs to step 1.9, where "`maf gate baseline` runs the judged harness five times against the bootstrap release" (plan 420) under a daily quota of 150,000.
Change: the owner decides (D85): the raised quota on the baseline day too, or one run a day.

**F8. The pipeline identity's roles are not defined, and nothing deploys salon-mcp. Must.**
The spec leaves the plan "the exact custom role definitions for each identity in section 4" and lists for the pipeline "deploy salon-mcp and gateway policies in the environment group; write release records" (spec 427). Plan 9.3 gives it only Foundry User on the project. Missing: push to the registry, the what-if and deploy role, read on logs, table rights for the scripted tests' reserved slots, write on the release-records container. Step 0.15 says the pipeline holds Foundry User at account scope "for the run" (plan 300), while step 0.3 grants project scope only. The spec says salon-mcp is "Deployed by the candidate job" (spec 358); no workflow step does it.
Change: add the pipeline's rows to 9.3, settle the account-scope grant in step 0.15, and add a salon-mcp build and deploy step to 1.10.

**F9. The estimates are not credible, and Phase 0 has nothing left to cut. Should, owner decision.** (Reported as F11.)
The arithmetic is right: 5.0 and 5.5 make 10.5; 5.25, 5.75 and 5.0 make 16.0. The estimates are another matter, and here the reviewer and the session differ. The session wrote the route's estimate.

| Item | Plan | Independent reviewer | Why |
|---|---|---|---|
| The `maf` command line | about 4 days | about 7 | Thirteen modules and about twenty-four commands, with tests; `maf` work also sits in steps the plan's list omits (0.4, 0.9, 0.13, 0.14, 0.17, 1.4, 1.7, 1.12, 1.14) |
| The agent route | 1.5 days | 2 to 2.5 | Step 0.9's half-day holds an API, a policy, a command, a test and an ADR; step 1.14's check alone is about half a day |
| Step 0.16 | 0.25 day | not credible | Two hooks with proofs, four skills, REVIEW.md, the plugin and `.mcp.json`; the half-day D81 saved was accounting |
| Steps 0.15 and 0.18 | 1 and 0.5 day | not credible | S6 gained three records for D80; 0.18 holds six ADRs, a spec update, thirteen exit rows and a council run |

The plan says of itself "Phase 0 has almost nothing left to cut" (plan 693). D30's rule, cut scope and hold the date, cannot then hold the Phase 0 date.
Change: the owner decides how the plan carries this (D87).

**F10. Text that decisions made stale. Should.** (Reported as F9.)
- D72: REVIEW.md pass 2 still asks a pull request that "the Bicep what-if shows only the intended changes" (plan 1000).
- D76: "run the sound version five times" in step 0.15 (plan 300); "thresholds.json (from S6)" (plan 103, 419); and in the spec "replaced by measured ones after spike S6" (1103), "Measured in S6" (1112), "until S6 measures the noise" (1528).
- D79: "Measured first in step 0.17" (plan 578), where the steps say 1.3; the layout names "github-actions and uv ecosystems" without docker (plan 53).
- D33: "each holding the Foundry Agent Consumer role scoped to the agent" (intent 232) and "The same route, checks, role and scope as hops 1a and 1b" (spec 413), against project-scope roles for the owner and the pipeline.
- D80: "Endpoint stays public" (intent 165), which the spec now records as a disagreement between Microsoft's pages; "which the reconciliation check of step 1.14 reports", said without its condition (plan 365).
- The alignment corrections left one residue: intent 168 and 178 still call the hosting libraries beta.
- The ADR template says "Numbers follow the run order" (plan 945); S5 runs before S3 and S4.
- docs/research.md, lines 65 and 115, still says the endpoint stays public.

**F11. The plan's own proofs plant bypasses that fail the nightly check. Should.** (Reported as F10.)
Step 1.5 proves the tool check "failing after one direct call" (plan 373); step 1.14 has "one direct call by `id-maf-test-caller` at the agent's own address" (plan 390); step 1.4 uses a direct `azd ai agent invoke` as proof. Audit rows are append-only, so every pipeline day fails that night. On the agent path "reports" is never said to fail the run or not. "Since its last run" (plan 388) needs a stored watermark, and the nightly identity holds no write role.
Change: mark planted calls and exclude them; say that the tool path fails the run and the agent path reports; use a fixed window, not a watermark.

**F12. Two additions to the spec claim more than their decisions. Should.**
The regulated-requirements table says "Model path closed by the model resource's role assignments (S1)" (spec 1552), while section 2.5 says of the same question that the "bypass is detected" fallback "is the likelier outcome" (spec 134). The same table lists "least-privilege identities" with the gap "None" (spec 1551), against D20, which accepts that sessions use the owner's login as subscription Owner.
Change: word both as conditional on S1 and name the gap.

**F13. Two items step 0.9 says it records cannot be observed as written. Should.**
"Whether a session opened through the route is scoped to the caller" (plan 247) needs two principals, and step 0.9 has one: "The test identities hold nothing until step 1.2" (plan 169). The fallback join key, "a request id header the policy stamps", needs the container to see inbound headers, and step 0.9 changes no agent file.
Change: move the session-scope check to step 1.14, where `id-maf-test-norow` asks Foundry directly for a session `id-maf-test-caller` opened through the route and expects refusal; have the echo agent of step 0.4 return the headers it saw.

**F14. Smaller gaps. Could.**
- "One API for every agent" (plan 383) needs agent names that are unique across tenants; the plan fixes one name, `salon-agent`.
- The permissions page says "A connection is created for Application Insights, which the project uses to emit telemetry for its agents"; step 0.3's Bicep creates none. Unverified whether traces flow without it.
- The Workbook has "four views" in the spec (800) and six in the plan (442).
- The tool rate limit is keyed by "tenant and agent together" in the spec (677) where D71 says "a per-tenant tool rate limit" (intent 278).

**F15. D67 is met for one exit row only. Should, owner decision.** (The session's.)
D67 says "The scripted write tests and the automatable exit rows of the spec's section 9 are feature files". The spec's own text makes that claim only of the scripted write conversations (spec 897). The plan has feature files for the three golden conversations and, since this revision, for the agent route's exit row; every other automatable exit row is an e2e pytest.
Change: the owner narrows D67 to what is built, or adds the feature files (D88).

**F16. Currency. Could.** (The session's; section 9.)
`fastmcp` 4.0.11 has appeared since the pin at 4.0.10. A newer generally available Foundry API version, 2026-09-01, exists beside the pinned 2026-07-01. Step 0.1 re-resolves both.

## 3. D72 to D82 across the three documents

Checked by script: 34 facts (phase lengths, the spike total, both rate limits, the token figures, the teardown times, the gateway's price, API versions, every shared pin, the eval row counts, the identity counts, both cut orders) agree wherever more than one document states them, after the corrections of section 14. Each of D71 to D82 is cited in all three documents. D73, D74, D75, D77, D78 and D81: nothing found. D82: F4 and F5. D72, D76, D79, D33 and D80: the residues in F10.

## 4. The spec's changes for D71 to D79, made after acceptance

Read as the difference between the accepted spec and the revision for D79. Faithful in substance. Left stale for D76: section 5.11 and one row of section 11 (F10). Beyond the decision: the two rows of the regulated-requirements table (F12), and the tool limit's key (F14).

## 5. D80's three corrections

- Why the agent path stays open: consistent in intent 186 and 292, spec 495 to 499, and the plan's risk row. Residue: intent 165.
- The join key as the condition for reporting a bypass: consistent in intent 186 and 292, spec 412, 725 to 738 and 1429, plan 250 and 388. Residues: plan 365 and 420 name the check without the condition.
- Evaluation runs as the stated exception: consistent in spec 218 and 714 to 717, plan 389 and 420. Residue: intent 74, where "every AI call crosses one governed boundary" comes before the exception in the same item.

## 6. Unverified items and whether their spikes can settle them

Can settle, as written: whether the quota counter survives a purge, and whether resource A can run with no chat deployment (S1); the token for a custom audience, and `disableLocalAuth` (S2, once F1 is fixed); roles on the free search tier (S4); the two `FixedRatio` rules (S5); the Action's result and the evaluator's model key (S6).

Cannot, as written:
- The tokens in one gate run. S6 measures a spike graph that D76 says lacks three of the four intents, yet the figure decides D39.
- The "unregistered identity" of S2 and of S1 test 5. No principal is named that can obtain a salon-mcp token in week 1.
- S4's third criterion, "deleted and recreated the same day" (spec 1241). Step 0.13 has no method for it.
- The D80 rows of spec 2.5: the approval round trip (F6) and the join key (F13).
- The judge's identity. Step 0.15 registers the project's identity, but the page says only "Foundry resolves the connection endpoint and authentication, including API key, managed identity, or OAuth 2.0 authentication". S6 should record the object id the gateway sees.
- The encoding of the registry copy. Plan 485 says it "is settled in spike S1, test 4"; tests 1, 2 and 5 need it first.

## 7. The three options carried from session 05

| Option | Recommendation | Cost | Risk of not doing it |
|---|---|---|---|
| (a) A nightly comparison of caller rows with Foundry role assignments | Not in Phase 1 if S1 finds a join key, because a direct call is then reported anyway. Build it if the outcome is "kept, without a join key", because nothing else watches the path. Run it in `up` under the owner's login, so the nightly identity gains no right | About 0.5 day: it must read inherited assignments at project, account and subscription scope, and exempt the two test identities that are out of step on purpose | Low while one owner holds every role; a principal with the role and no row is refused at the gateway and answered at the agent's own address |
| (b) A consumer-only identity for the pipeline's calls through the route | Do it in Phase 1. `id-maf-test-caller` already holds only Foundry Agent Consumer, a caller row and the `dev` credential; step 1.14 already builds the sign-in; Appendix A already says "the caller is a registered test identity". It also gives F5 its caller | About 0.25 day | On every run the gateway handles a token that can create versions and that "carries the `sessions/read` and `sessions/write` data actions for cross-user access" (sessions page). Unverified: whether a consumer can open a session pinned by `version_ref`; the permission "covers all runtime interactions with the agent", which is not specific to pinning. S6 confirms it |
| (c) Responses generated through the route and evaluated as a dataset | Not the Phase 1 default: D56 fixed the route and S6 is full. Name it now as S6's fallback, if an unserved version cannot be targeted or if evaluation turns cannot be told from bypasses. It is documented: "Evaluate precomputed responses in a JSONL file by using the `jsonl` data source type" | About 1 day, 220 paced calls in a gate run, and an amendment to D56 | On gate days several hundred turns do not cross the gateway; they need a reliable run id, or the agent-path check is reduced to "reports and does not fail" |

## 8. Schedule and estimates

The sums are correct and were checked by script against the step headings. The estimates are in F9. Two more points. The agent route's line, 1.5 days, was written by the session that made the decision possible, and the independent figure is 2 to 2.5; the owner should weigh the independent one. And Phase 1's cut order puts the route second, so an overrun of a day in week 2 removes the route: the time D81 bought protects the test, not the build.

## 9. Currency audit: the pins

Run by script on 2026-10-04.

| Pin | Result |
|---|---|
| 23 PyPI packages (plan section 3) | All exist, none yanked, 22 are the latest. `fastmcp` 4.0.11 is newer than the pinned 4.0.10 |
| Hosting protocol libraries | The hosting extra of `langchain-azure-ai` 1.2.10 requires `azure-ai-agentserver-core<3.0,>=2.1.0b2`, `-responses<3.0,>=2.1.0b2` and `-invocations<2.0,>=1.1.0b1`. Stable 2.2.0, 2.2.0 and 1.2.0 exist. The plan's 2.2.0 was right and the spec's 2.1.0b2 was stale; corrected in section 14 |
| 12 Azure Verified Module tags (plan 9.1) | All exist and all are the latest |
| 10 action SHAs (plan 9.9) | All match their tags and are the latest releases. `azure/bicep-deploy` was used but unpinned; corrected in section 14 |
| ARM API versions | `accounts/projects` 2026-07-01, API Management 2025-09-01-preview, workbooks 2023-06-01 and role definitions 2022-04-01 are all listed by the providers. A newer generally available Foundry version, 2026-09-01, exists |
| Search REST API | 2026-04-01 is the version the page documents |
| `@azure/mcp` | 2.0.5 is the latest stable; the `latest` tag on npm is 3.0.0-beta.49 |
| azd `azure.ai.agents` extension | 1.0.0-beta.18 is the latest; still "Foundry agents (Beta)" |

Spec section 7 and plan section 3 now agree on every shared pin.

## 10. Step numbering

Steps 0.1 to 0.18 are contiguous: the gap at 0.9 is filled by the test 6 step. Step 1.14 sits between 1.5 and 1.6, is explained where it appears, and every reference to it resolves. Two wrong references remain: plan 578 and plan 945 (F10).

## 11. Could an engineer who has never seen the conversation implement each step?

- Not from the plan alone: 0.4 and 0.6 (F1); 0.5 (F3); 0.8 and 1.10 (F8); 0.9 (F1, F6, F13, and the copy's JSON schema is promised at plan 244 but section 9.2 gives only a row key); 0.10 (F2, F5); 0.11 (F1); 1.2 and section 9.2 (F2, F4); 1.5 (F4, and its proof uses a sign-in that step 1.14 defines later); 1.7 (F5); 1.9 (F7); 1.14 (F11).
- As written: 0.1, 0.2, 0.3, 0.7, 0.12 (after F6), 0.13, 0.14, 0.16, 0.17, 1.1, 1.3, 1.4, 1.6, 1.8, 1.11, 1.12, 1.13.
- Test identities: D82's three are created in step 0.2, published as GitHub variables in 0.7, given rows and roles in section 9.2, and used in 1.5, 1.13 and 1.14. The low-quota identity has no creation step (F4).

## 12. Proposed changes

| Change | Mark | Needs the owner |
|---|---|---|
| Reorder Phase 0 so that each step has what it uses (F1) | Must | No |
| Decide and state how the first agent identity is registered on a clean clone (F2) | Must | D83 |
| The three metric settings in steps 0.2 and 0.5 (F3) | Must | No |
| Create the low-quota identity; a quota field on the agent row; the judge's figure (F4) | Must | D84 |
| The nightly guardrail test in a `dev` job; an unconditional delete (F5) | Must | With D86 |
| The conversations operation; a 404 outside the drop rule (F6) | Must | No |
| The quota on the baseline day (F7) | Must | D85 |
| The pipeline's roles; a salon-mcp deploy step (F8) | Must | No |
| How the plan carries the estimates (F9) | Should | D87 |
| The stale text, in all three documents and docs/research.md (F10) | Should | No |
| Planted calls excluded; report against fail; a fixed window (F11) | Should | No |
| The two rows of the regulated-requirements table (F12) | Should | No |
| Step 0.9's two records moved or made observable (F13) | Should | No |
| The pipeline's calls made as the consumer-only identity (option (b)) | Should | D86 |
| D67 narrowed or met (F15) | Should | D88 |
| Options (a) and (c) recorded with their conditions | Should | With D86 |
| Unique agent names; the Application Insights connection; the Workbook's views; the tool limit's key (F14) | Could | No |
| `fastmcp` and the Foundry API version at step 0.1 (F16) | Could | No |

## 13. Decisions that would need a new number (D83 onwards)

The first option in each is the reviewer's recommendation.

- **D83, the first registration (F2).** (1) The candidate job registers the agent's identity itself, running the admin script's register step under the pipeline identity with a write role on the `registry` table and an audit record; the pipeline can already deploy the gateway's policies and salon-mcp, so this adds no trust it does not hold. (2) The owner registers after the first run, and the exit row becomes one setup command, one pipeline run, one registration and a re-run of the gates.
- **D84, the low-quota identity (F4).** (1) A fourth managed identity, `id-maf-test-quota`, federated to `dev`, with an agent row and a 2,000-token quota; D82 amended from three to four. (2) `id-maf-test-norow` is given that agent row as well, which saves an identity and blurs what its name says.
- **D85, the baseline day (F7).** (1) The 3,000,000-token quota of D76 applies on the day the five baseline runs are made, and is restored after, about £3.40. (2) One baseline run a day over five days.
- **D86, the pipeline's calls (option (b), F5).** (1) The smoke test, the scripted write tests and the guardrail tests, in the candidate job and nightly, call the agent as `id-maf-test-caller`; the pipeline identity keeps Foundry User for deploying and holds no caller row; conditional on S6 showing that a consumer can open a `version_ref` session. Options (a) and (c) are recorded with their conditions and not built. (2) The pipeline calls as itself, as the spec says today.
- **D87, the estimates (F9).** (1) Keep D81's boxes; add a checkpoint at the end of Phase 0 week 1, where actual days are compared with the estimates and the owner re-plans; and name step 0.17 as Phase 0's next cut, which amends D79. (2) Extend the boxes now, to about 12 and 18 working days. (3) Leave the plan as it is.
- **D88, D67 (F15).** (1) D67 is narrowed to what is built: the golden conversations, the scripted write tests and the agent route's exit row are feature files; the other automatable exit rows are e2e tests named in the plan. (2) Feature files for every automatable exit row, about half a day that Phase 1 does not have.

## 14. Corrections already made in this session, on the owner's instruction to align the documents

Commits cacfe55 (spec) and 65d6849 (plan). These were findings of this review, applied before it was written because they needed no decision.

- Spec section 7: the hosting protocol libraries at their stable releases, as the plan pins them.
- Spec section 4: the owner holds Foundry Project Manager on the tenant project, which deploying requires ("You need the Foundry Project Manager role at the project scope to deploy a hosted agent") and which the plan already grants.
- Spec section 3.4: the teardown identity deletes and reads, with no purge (D74). Spec section 6.1: the bootstrap also creates the gateway's identity (D73).
- Plan section 3: no what-if on pull requests (D72); the third package of the hosting extra pinned.
- Plan section 9.9: `azure/bicep-deploy` pinned by SHA, which the repository's setting requires.
- Plan section 9.3: `Microsoft.ApiManagement/service/namedValues/read` for the teardown role; it is a separate action from `service/read`, and the nightly registry check needs it.
- Plan step 1.12: the monthly 2,000,000-token figure and its alert (spec 5.4, D39), which no step had built.
- Plan step 0.5: a first form of `maf down`, because the gateway exists from week 1 and the nightly workflow does not arrive until step 0.10.

## 15. Sources opened in this session

Microsoft Learn, under https://learn.microsoft.com:
- /en-us/azure/foundry/agents/concepts/hosted-agent-permissions
- /azure/foundry/agents/how-to/manage-hosted-sessions
- /azure/foundry/agents/how-to/isolate-sessions-per-user
- /azure/foundry/agents/how-to/deploy-hosted-agent
- /azure/foundry/agents/how-to/configure-agent
- /en-us/azure/foundry/agents/how-to/virtual-networks
- /azure/foundry/concepts/rbac-foundry
- /azure/foundry/control-plane/register-custom-agent
- /azure/foundry/observability/how-to/cloud-evaluation-targets
- /azure/foundry/observability/how-to/cloud-evaluation-datasets
- /azure/foundry/observability/how-to/evaluate-admin-connected-models
- /azure/cloud-adoption-framework/ai-agents/integrate-manage-operate
- /azure/api-management/rate-limit-by-key-policy
- /azure/api-management/validate-azure-ad-token-policy
- /azure/api-management/emit-metric-policy
- /en-us/azure/api-management/llm-token-limit-policy
- /azure/api-management/api-management-howto-app-insights
- /azure/api-management/expose-existing-mcp-server
- /azure/api-management/agent-to-agent-api
- /azure/api-management/api-management-policy-expressions
- /azure/api-management/api-management-gateways-overview
- /azure/api-management/how-to-server-sent-events
- /azure/api-management/api-management-howto-properties
- /en-us/rest/api/apimanagement/diagnostic/create-or-update?view=rest-apimanagement-2024-05-01
- /en-us/rest/api/searchservice/indexes/create
- /en-us/azure/role-based-access-control/permissions/integration
- /en-us/azure/templates/microsoft.insights/components
- /en-us/cli/azure/ad/user
- /en-us/entra/fundamentals/security-defaults
- /en-us/entra/workload-id/workload-identity-federation-create-trust-user-assigned-managed-identity
- /azure/developer/github/connect-from-azure-openid-connect

Elsewhere:
- https://raw.githubusercontent.com/microsoft-foundry/foundry-samples/main/samples/python/hosted-agents/langgraph/responses/07-human-in-the-loop/README.md
- https://raw.githubusercontent.com/Azure/bicep-registry-modules/main/avm/res/api-management/service/README.md
- https://raw.githubusercontent.com/Azure/login/master/README.md
- https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-azure
- PyPI JSON API, https://pypi.org/pypi/{name}/json, one call for each pinned package
- Microsoft container registry tag lists, https://mcr.microsoft.com/v2/bicep/{module}/tags/list
- npm registry, https://registry.npmjs.org/@azure%2Fmcp
- GitHub REST API, tags and releases of each pinned action
- Azure Retail Prices API, https://prices.azure.com/api/retail/prices
- `az provider show` for Microsoft.CognitiveServices, Microsoft.ApiManagement, Microsoft.Insights and Microsoft.Authorization (read only)

Not opened or not verified: whether a hosted agent accepts a client-chosen conversation id without the conversations call; a Bicep property for Application Insights custom metrics with dimensions; whether traces flow from a project with no Application Insights connection; whether Foundry Agent Consumer can open a `version_ref` session; which identity the evaluation service presents on a gateway connection. The eval Action's source and the Claude Code references of plan section 13 were not re-opened.
