# Principal engineer review: AgentOps coverage of the intent, the spec and the plan

Date: 2026-10-10. Artifacts reviewed: docs/intent.md revision 18, docs/spec-phase-0-1.md and docs/implementation-plan-phase-0-1.md at commit 38d48e3, and then each document as it was revised for D89 to D96.
Method: one deep single-reviewer pass with a currency audit, not a council run (D70). The owner asked for it as a gate before Build. The owner's decisions on it are D89 to D96 in docs/intent.md revision 19.
Reviewer lens: AgentOps on Azure: evaluation before and after release, guardrails, typed contracts, observability and the maintain loop, on Microsoft Foundry hosted agents.
Reads: the three documents above, docs/journal/06-implementation-plan-gate.md section 7, docs/council/06-implementation-plan-phase-0-1-principal-review.md sections 1 and 2, CLAUDE.md.

How the pass was made. The session read the three documents itself and located every practice in each. Three research subagents, which wrote nothing, checked Microsoft's current pages for offline evaluation, online evaluation, and observability and guardrails. The session then matched every quotation it relies on to its page by script: 68 in the check itself, and the quotations of each revised document again before its commit. Each revised document was read by an independent reviewer who was given the decisions and not the reasoning, before the next document was derived from it (section 9 onwards).

## 1. Verdict

**The three documents agree on what they state, and four practices were stated nowhere or could not work as written.** The offline gate, the input guards, the citation check, tracing, budgets, versioning, approval and rollback are aligned and inside Phases 0 and 1. What was missing: typed schemas, a growth loop for the dataset that works while traces hold no message content, a scheduled evaluation of the served version, and an output filter that is claimed and shown refusing. None is a redesign. The owner adopted all four. The session priced them at one day; the independent reviewer of the spec priced them at two and three quarter days, and the owner extended Phase 1 to 20.75 working days (section 10).

## 2. Coverage table

Section numbers are those of the documents at 38d48e3. "Later" means later by a recorded decision.

| Practice | Phase | Intent | Spec | Plan | Aligned |
|---|---|---|---|---|---|
| Eval gate, evaluators, thresholds, baseline | 0 (spike S6), 1 | 6 item 7; D44, D56, D57, D76, D85 | 5.8, 5.11, 9.2 | 0.15, 1.9, 9.4 | Yes |
| Scripted write tests | 1 | D44, D67, D88 | 5.8, 9.2 | 1.3, 1.9, Appendix A | Yes |
| Continuous evaluation of live traffic | 6 | 3, 7 | 3.4, 5.6 | None | Yes, later |
| Scheduled evaluation of the served version | None | None | None | None | Missing |
| Trace evaluation of the write intents | 2 | D77 | 5.8 | 9.5 | Yes, later |
| Golden dataset: signed, versioned, hashed | 0, 1 | D57, D67 | 5.8, 5.11 | 0.14, 1.10, Appendix F | Yes |
| Growth of the dataset | 1, by hand | 10, one row | 5.11 | None | No |
| Pydantic, named and pinned | None | None | None | None | Missing |
| Graph state, extraction output, answer shape, tool results, release record, gate result | None | None | Prose only (5.1, 5.2, 5.9) | File names only (2) | Missing |
| Registry rows; tool inputs with no tenant_id | 1 | 6 item 5 | 4, 5.3, 9.2 | 1.1, 1.8, 9.2 | Yes |
| Prompt-injection screening and its negative test | 1 | 6 item 10; D31 | 5.7, 9.2 | 1.7 | Yes |
| Validation in code | 1 | 6 item 1 | 5.1, 9.2 | 1.3, Appendix A | Yes |
| Injection through retrieved content | 1 | D46, D78 | 5.7 | 1.7, Appendix A | Yes |
| Citation check | 1 | D46 | 5.1, 9.2 | 1.3, Appendix A | Yes |
| Output content filter, and a test that it refuses | None | None | None | None | Missing |
| Personal data in responses | 5 | 7, 13 | None | None | Yes, later |
| Tracing and attribution | 1 | 5; D54, D59 | 5.6, 9.2 | 1.6 | Yes |
| Cost and token budgets | 0, 1 | D21, D39, D84 | 5.4, 8, 9.2 | 0.2, 1.5, 1.12 | Yes |
| Versioning | 1 | D38, D46, D50 | 3.4, 5.1, 5.9 | 0.5, 1.4, 1.10 | Yes, bar one sentence |
| Approval | 0, 1 | D19, D41 | 5.9, 9 | 0.7, 1.10 | Yes |
| Rollback | 1 | 6 item 8 | 5.9, 9.2 | 1.11 | Yes |
| Drift in quality | 6 | 3, 7 | 5.6 | None | Yes, later |
| Safety evaluators and red teaming | 2, to be decided | D63 | 5.8, 11 | None | Yes, later |
| Human in the loop | 1 | 6 item 1 | 5.1, 9.2 | 1.3, Appendix A | Yes |
| Alerts and the maintain loop | 0, 1, 6 | 10 | 3.4, 5.4, 5.11 | 0.2, 1.12 | Partly |
| User feedback | 5 | 7 | None | None | Yes, later |

The one sentence under Versioning: the spec said "Tool schemas are pinned in the eval dataset to catch drift" (5.9, and a row of section 11). The eval rows of the plan's Appendix F have no such field, and no step pinned anything.

## 3. Online evaluation and message content

D46 keeps message content out of traces. Against that:

- **Evaluation of live traffic needs it.** "Quality evaluators read the query and response from each trace." Microsoft's pages do not say how a continuous rule reads a hosted agent, and its tools page pairs "evaluation rules for prompt agents, or scheduled evaluation runs for hosted agents". It is also in preview: "Recurring evaluations (preview)".
- **A scheduled evaluation against the golden dataset does not.** An agent-target run will "Send test queries to deployed models, prompt agents, or hosted agents and evaluate the responses generated at run time". Microsoft names it as the drift check: "Scheduled quality and safety evaluation using test datasets to detect system drift".
- **Not confirmed:** that a run scheduled by Foundry can target an agent. The quality run in Microsoft's scheduled sample scores a column of recorded responses. The nightly workflow can make the documented one-off call itself.
- **Foundry's route from traces to a dataset** is the portal's "Add to dataset" or a generation job in `azure-ai-projects` 2.8.0, and "The job produces a versioned dataset registered in your project." Its rows hold "the description, query, and response for each". What it returns for traces without content is not stated; the expectation is nothing usable.
- **A fact for Phase 2.** Microsoft's page on recorded content, in preview, says "Starting September 30, 2026, Application Insights stops routing the attribute values to the existing tables for newly ingested data". The plan's section 9.5 expected to find them in the older tables.

## 4. What Microsoft's guidance names that the owner's list did not

| Practice | Source says | Handling |
|---|---|---|
| Checking the judge against a person | "Run a small evaluation on a sample dataset to validate that the rubric scores align with your own judgment before using it at scale." Given for rubric evaluators | Adopted as text: the owner reads 20 verdicts on the baseline day (D93) |
| A way to stop an agent | "Decide in advance how to quickly disable an agent if it malfunctions or causes harm." | Adopted as text: the default `down`, with its limits stated (D93) |
| Rubric evaluators | "Use rubric evaluators as your primary measure of agent quality" | Not used in Phase 1; a candidate for Phase 2 |
| Human evaluation and trace annotation | Both pages are marked preview | Not used; the owner signs outcomes (D67) |
| User feedback as telemetry | "Each feedback event links back to the originating trace and span"; preview | Phase 5; it carries scores and ids, so it fits D46 |
| Canary and A/B release | "An agent endpoint serves one version at a time and routes 100% of its traffic to that version." | Not built; the spec's traffic splitting note stands |
| An agent inventory | "Require every AI agent to be recorded in a single organizational inventory." | Phases 3 and 4; the registry table is the seed |

## 5. Gaps ranked by impact

The intent's D90 to D95 refer to these by number. The day figures in this section are the session's first estimates; section 10 gives the independent ones, which the owner adopted.

1. **Typed schemas.** Pydantic appeared in none of the three documents. Nothing defined the extraction output, the answer's shape or the two evidence records, or what the graph does with invalid model output. Recommendation: Pydantic 2 named and pinned, typed models at the model, tool and record boundaries, schemas compared on every pull request. Half a day, Phase 1. Facts found: FastMCP already refuses an undeclared argument ("on the generated schema and enforces it, so the same call fails"); a Pydantic graph state is validated only "on inputs to the first node in the graph, not on subsequent nodes or outputs"; both chosen models list structured outputs.
2. **The dataset growth loop could not work as written.** Spec 5.11 built a row from a trace, which holds no content under D46. The plan had no step for it, and the intent's "control bands" were defined nowhere. Recommendation: correct the text at no build cost; the drill in Phase 2 on the eval twin, a quarter of a day.
3. **No scheduled evaluation of the served version.** Recommendation: the nightly workflow runs the judged harness on a 20-row sample. A quarter of a day, Phase 1.
4. **The output content filter was neither claimed nor shown refusing,** against D31. It acts by default on the model deployments. Recommendation: a planted term in a completion blocklist, a test that expects the block, a fixed reply from the graph. A quarter of a day, Phase 1. Documented behaviour: "the completion only cuts off when the blocklist content is matched".
5. **No single table** said which practice is built in which phase. Recommendation: one table in the spec.
6. **Small, text only:** the owner's check of the judge, a named way to stop the agent, and the dataset's version in the project.

## 6. Beyond the stop line (D30)

Phases 5 and 6 hold continuous evaluation of live traffic, finer drift detection, SLOs, masking of personal data and user feedback. The project counts as complete without them. The scheduled evaluation is the one piece of "online evals and drift" that could come forward, because it needs no content. Safety evaluators and red teaming are inside the stop line, in Phase 2, and still not offered in UK South: UK South is in the page's lists for batch evaluation and data generation and not in its list for risk and safety evaluators, and "Cloud red teaming is currently available in the following regions: East US 2, France Central, Sweden Central, Switzerland West, and US North Central." D63 stands.

## 7. Currency audit

Checked on 2026-10-10 against PyPI, the Azure Developer CLI, Microsoft Learn and the Azure Retail Prices API.

| Item | Plan or spec | Today | Handling |
|---|---|---|---|
| uv | 0.12.23 | 0.13.0 installed | Step 0.1 records it |
| fastmcp | 4.0.10 | 4.1.0 | Step 0.1 re-resolves, as the plan says |
| langgraph | 1.2.12 | 1.2.14 | Step 0.1 |
| azure-ai-projects | 2.7.0 | 2.8.0, which the dataset generation pages require | Step 0.1 |
| azd `azure.ai.agents` extension | 1.0.0-beta.18 | beta.18 installed; beta.19 exists | Step 0.1 |
| Azure Developer CLI | 1.34.2 or later | 1.35.0 installed | None needed |
| Bicep CLI | Absent | 0.48.1 installed | None needed |
| Pydantic | Not named | 2.14.0 | Pinned by D90 |
| Tracing for hosted agents | Not stated | "Tracing is generally available for prompt and hosted agents" | Recorded in the spec |
| Evaluation by the Azure Developer CLI | Not known | "The Azure Developer CLI evaluation experience is currently in preview" | A third route for spike S6, tried only if both fail |
| Task adherence and intent resolution | Preview | Still marked preview | None |
| Docker Desktop on the workstation | Listed | Not installed (D89) | Spike images by `az acr build`, £0.0001 a vCPU second |

## 8. Decisions

D89 to D96 in docs/intent.md revision 19. In one line each: no Docker on the workstation (D89); typed contracts (D90); the output filter shown refusing (D91); a nightly evaluation of the served version (D92); the dataset growth loop corrected (D93); Phase 1 is 20.75 working days, with the two new builds third and fourth in its cut order (D94); one table of practice in the spec (D95); a fifth test identity for the filter's test (D96).

## 9. Independent read of the intent, revision 19 at commit 98bf704

The reviewer confirmed all 16 quotations and the price, and found the day figures consistent. It reported three must-level defects and nine should-level ones. All were accepted.

| Finding | What was done |
|---|---|
| M1. This file was cited and did not exist | Written before the spec was derived |
| M2. The nightly run failed on any failed check in the sample. At 95 per cent intent accuracy a 20-row sample fails about 64 nights in 100 | The run fails only on a structural fault or below a floor set from the baseline runs, about one false alarm in a hundred nights; it reports only until the baseline exists |
| M3. The filter test reused the low-quota identity, against D84, and would fail on a second run in a day | The owner chose a fifth test identity (D96) |
| S1. The statement that after `down` the agent "reaches neither a model nor a tool" overclaimed | Restated: this stops a faulty agent, not a misused identity, because the agent's endpoint and salon-mcp still answer directly (D71, D80) |
| S2. A served-only run was compared with the estimate for a whole gate run | Restated as a shared daily quota; a refusal is failed-to-run (D53) |
| S3. Two sentences on continuous evaluation went beyond their pages | Restated to what the pages say |
| S4. The trace does not name the caller | Tests and evaluation runs record their conversation ids |
| S5. The control bands omitted the registry check; the Defender alert ends with its trial | Both corrected |
| S6. Earlier decisions lacked their amendment notes | Added to D30, D31, D39, D48, D51, D57, D67, D69, D74, D76, D82, D84, D86 and D87 |
| S7. D93's work had no time, and the cut order was unstated | D93 states why it needs no build time; the owner placed the two builds in the cut order (D94) |
| S8. The reason given for claiming no blocklist on the agent's policy did not follow | Restated: the hosted agent guardrail page does not mention blocklists |
| S9. Drift was in Phase 6 in one place and Phase 1 in another | "Drift detection finer than the nightly sample" stays in Phase 6 |
| Could-level: the Docker quotation was not the reason; "section 6, item 5" was misdescribed; the references section was not extended | All three corrected |

The owner was also told that the session had been wrong to say nothing in Phase 0 uses Docker: the spikes of plan steps 0.4, 0.6 and 0.15 build images before the pipeline exists, which is why D89 names `az acr build`.

## 10. Independent read of the spec at commit 7a85a41

The reviewer opened all 20 new keys and eight re-used ones and found every quotation exact. It reported four must-level defects and twelve should-level ones in the spec, six sentences that went beyond their sources, and an estimate nearly three times the session's. All were accepted.

| Finding | What was done |
|---|---|
| M1. The nightly run's "structural fault" included a tool call the row forbids. A misread intent makes the code call a read tool, so a sound version would have failed on about one night in three | A structural fault is now a cited id that was not returned, or a call to a tool that writes. An unexpected read tool counts against the intent's rate. D92 was corrected |
| M2. A quota refusal had no defined outcome, and on a day with a pipeline run the sample would be refused | Four named outcomes: passed, structural fault, below a floor, failed to run. The daily quota set after spike S6 covers one gate run and the sample |
| M3. An older sentence said the deployment's filter "is not a second layer", against the control D91 claims | Restricted to model calls Foundry makes for an agent. The graph's calls are ordinary inference calls through the gateway, and the filter's test is the proof; this is listed as unverified until that test runs |
| M4. Nothing said who writes the filter identity's agent row, or the low-quota identity's since D84, so a clean clone would fail | The admin script writes both when the tenant is created. The row also opens the tool route, where salon-mcp refuses both identities for want of the agent marker claim |
| S1. The floor could not be computed for a perfect baseline, and one false alarm in a hundred was per rate | The rate is capped at 0.95 and the one in a hundred is shared across the rates |
| S2. The nightly job's logins, order and once-a-day guard were unstated | Two logins in one job; the evaluation runs after the checks; a run that finds the day's record does nothing |
| S3, S4. Exit rows against the cut order; the pipeline's gate list omitted the filter's test | Both rows say "unless cut"; the test is in the gate list and in the diagram, and blocks promotion |
| S5. The negative demonstration could not run as written | Started by hand against a fixture, marked as a demonstration, after the floors exist |
| S6. Structured outputs accept only a subset of JSON Schema; a cut completion is also invalid JSON; `refuse`'s own call was unhandled; shared models implied a shared package | The models keep to the subset ("All fields must be required"); the finish reason is read first; `refuse` falls back to a fixed decline; no shared package, the committed schemas are what the packages share |
| S7. The reason given for not using the personal-data filter was wrong | Restated for each of the two policies |
| S8. One new row moves a 50-row rate by two points and does not fail the gate | A fault that code can check becomes a failing test first; a must-pass rule for single rows is left to Phase 2 |
| S9 to S12. Stale figures; this read unrecorded; "up waits"; the practice table's phases and five missing rows | All corrected; the table gained audit and release evidence, the dashboard, the registry and identity, bypass detection and the Defender trial |
| Beyond their sources: FastMCP's sentence describes a later release than the one pinned; "a third route has appeared"; the README; a hash as a dataset version; "scores and ids"; the tracing page's status | Each restated to what its page says. The unit test stays the control for the tenant rule; three questions joined spike S6's recorded list |

**The estimates.**

| Decision | Session | Independent | Reason given |
|---|---|---|---|
| D90, typed contracts | 0.5 | 1.0 | Schema export and the snapshot check, structured outputs within the strict subset, the refusal path and its tests |
| D91, the output filter's test | 0.25 | 0.5 | The policy and blocklist, a test with its own login and retry, handling in every model node |
| D92, the nightly evaluation | 0.25 | 1.0 | A single-version mode, the seeded sample, the floors, the guard, the quota case, the Workbook, the demonstration |
| D93 and D96 | 0 | 0.25 | Conversation ids, the agreement count, one identity with its row and login |
| Total | 1.0 | 2.75 | |

The owner was asked and adopted the independent figures: Phase 1 is 20.75 working days (D94). Holding 19 days, and moving the nightly evaluation to Phase 2, were not adopted.

The spec's seven diagrams were rendered after the correction. Each is valid, matches its block in the file and shows every label, checked by script; one changed, the release pipeline, which gained the filter's test.

## 11. Sources opened in this session

Every quotation in this file was matched to its page by script on 2026-10-10.

- https://learn.microsoft.com/azure/foundry/concepts/observability
- https://learn.microsoft.com/azure/foundry/observability/how-to/how-to-monitor-agents-dashboard
- https://learn.microsoft.com/azure/foundry/observability/how-to/troubleshooting
- https://learn.microsoft.com/azure/foundry/observability/how-to/cloud-evaluation-targets
- https://learn.microsoft.com/azure/foundry/observability/how-to/cloud-evaluation-datasets
- https://learn.microsoft.com/azure/foundry/observability/how-to/traces-to-dataset
- https://learn.microsoft.com/azure/foundry/observability/how-to/traces-sensitive-content
- https://learn.microsoft.com/azure/foundry/observability/how-to/human-evaluation
- https://learn.microsoft.com/azure/foundry/observability/how-to/trace-annotations
- https://learn.microsoft.com/azure/foundry/observability/how-to/log-end-user-feedback
- https://learn.microsoft.com/azure/foundry/observability/how-to/evaluation-dataset-synthetic
- https://learn.microsoft.com/azure/foundry/observability/concepts/trace-agent-concept
- https://learn.microsoft.com/azure/foundry/observability/how-to/trace-agent-framework
- https://learn.microsoft.com/azure/foundry/observability/quickstarts/quickstart-evaluate-hosted-agent
- https://learn.microsoft.com/azure/foundry/mcp/available-tools
- https://learn.microsoft.com/azure/foundry/concepts/built-in-evaluators
- https://learn.microsoft.com/azure/foundry/concepts/evaluation-evaluators/agent-evaluators
- https://learn.microsoft.com/azure/foundry/concepts/evaluation-evaluators/rubric-evaluators
- https://learn.microsoft.com/azure/foundry/concepts/evaluation-evaluators/custom-evaluators
- https://learn.microsoft.com/azure/foundry/concepts/evaluation-evaluators/risk-safety-evaluators
- https://learn.microsoft.com/azure/foundry/concepts/evaluation-regions-limits-virtual-network
- https://learn.microsoft.com/azure/foundry/concepts/ai-red-teaming-agent
- https://learn.microsoft.com/azure/foundry/how-to/evaluation-github-action
- https://learn.microsoft.com/azure/foundry/guardrails/guardrails-overview
- https://learn.microsoft.com/azure/foundry/guardrails/intervention-points
- https://learn.microsoft.com/azure/foundry/agents/how-to/add-hosted-agent-guardrails
- https://learn.microsoft.com/azure/foundry/agents/concepts/hosted-agents
- https://learn.microsoft.com/azure/foundry/openai/how-to/use-blocklists
- https://learn.microsoft.com/azure/foundry/openai/how-to/structured-outputs
- https://raw.githubusercontent.com/langchain-ai/langchain-azure/main/libs/azure-ai/README.md
- https://learn.microsoft.com/azure/foundry/openai/concepts/content-filter-personal-information
- https://learn.microsoft.com/azure/foundry/foundry-models/concepts/models-sold-directly-by-azure
- https://learn.microsoft.com/azure/templates/microsoft.cognitiveservices/accounts/raipolicies
- https://learn.microsoft.com/azure/container-registry/container-registry-quickstart-task-cli
- https://learn.microsoft.com/azure/cloud-adoption-framework/ai-agents/governance-security-across-organization
- https://learn.microsoft.com/azure/well-architected/ai/operations
- https://docs.langchain.com/oss/python/langgraph/use-graph-api
- https://gofastmcp.com/llms-full.txt
- https://docs.pydantic.dev/latest/concepts/json_schema/
- https://raw.githubusercontent.com/Azure/azure-sdk-for-python/main/sdk/ai/azure-ai-projects/samples/evaluations/sample_scheduled_evaluations.py
- https://pypi.org/project/pydantic/
