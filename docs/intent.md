# Intent: Multi-tenant AgentOps platform on Azure with Agent Factory

Author: Rana Naveed Idrees. Status: accepted (revision 12). Owner decisions D1 to D55 recorded in section 14. Date: 2026-10-03.
Stage: 1 of 6 (Plan). Location: docs/intent.md. Next artifact: docs/spec-phase-0-1.md (Phases 0 and 1).
Council record: docs/council/01-intent-review.md (revision 4) and docs/council/02-intent-review.md (revision 7); for the spec, docs/council/03-spec-phase-0-1-review.md
Revision 7 records the Stage 0 harness interview (D7 to D12) and amends the text those decisions contradict; each amendment is marked with its decision number.
Revision 8 records the owner's response to council review 02 (D13 to D17). The review's other questions remain open for the spec.
Revision 9 applies three cleanup corrections approved by the owner: the isolation test phases in section 13, question 4 removed (answered by D6) and question 8 pointed at D11.
Revision 10 records D18 (the repository is to be made public).
Revision 11 records the Stage 2 design interview (D19 to D36), amends the text those decisions contradict, and applies two unopposed corrections from council review 02, debate 7. Each amendment is marked with its decision number or source.
Revision 12 records the Stage 2 spec gate (D37 to D55), held after council review 03 of the draft spec, and amends the text those decisions contradict. Each amendment is marked with its decision number. It also applies two cleanup corrections approved by the owner: the file name in the heading of section 7 and the mitigations in section 13.

## 1. Problem

Regulated organisations (financial services, credit ratings, insurance) can build one AI agent quickly, but struggle to run many agents for many clients or business units safely. Each team reinvents hosting, tools, knowledge bases, cost control, evaluation and audit. The result is inconsistent governance, weak tenant isolation, no evidence that a release is safe, no view of cost or value per client, and no repeatable path from idea to production.

## 2. Goal

Build a multi-tenant AgentOps platform on Azure that shows how agents are created, governed, released, observed and paid for at enterprise standard, and use it as the centrepiece portfolio project for AI Principal Engineer interviews in regulated industries.

The platform is the story. Agents are the evidence.

## 3. Outcome, delivered in phases

Each phase is usable and demonstrable on its own. Later phases build on earlier ones; nothing in a later phase is required for an earlier phase to work.

| Phase | Name | What exists at the end | Interview story it unlocks |
|---|---|---|---|
| 0 | Foundations | Repo, CLAUDE.md, REVIEW.md, skills, hooks, council agents, IaC skeleton, CI with OIDC, spikes resolved | "This is how I set up AI-native delivery with guardrails" |
| 1 | **MVP: governed agent, one tenant, tenant-ready** | Salon agent and salon-mcp (with RAG) for one tenant; tenant identity bound to the agent and enforced server-side; gateway token budget (D22, D23); OpenTelemetry with full attribution keys, with an optional export to Langfuse Cloud (D7, D32); offline eval gate; candidate promoted to served with approval, in dev (D19); Workbook dashboard | "Governance, cost and quality are enforced by the platform, not the agent" |
| 2 | Multi-tenant | Second tenant via the same IaC; automated isolation tests in CI; per-tenant views on the dashboard | "Isolation is proven by tests, not claimed" |
| 3 | Agent Factory | Spec schema, templates (agent, MCP server, RAG knowledge base), generator that opens a PR, CI policy checks; **regulated document Q&A agent** shipped through the same gate | "New agents inherit the golden path; humans approve, pipelines deploy" |
| 4 | Admin Console | FastAPI and React on Azure: provision tenants, request agents, request deployments, lifecycle dashboard | "One control plane for tenants and agents" |
| 5 | Tenant Portal | FastAPI and React, Entra External ID: ROI, outcomes, chat history with masking and audit, knowledge gaps | "Clients see value and evidence, never each other" |
| 6 | Depth | Ledger and statements, SLOs and error budgets, online evals and drift, self-hosted Langfuse (D7), knowledge self-service, siloed isolation | "Chargeback, reliability engineering, continuous quality" |

Rule: a phase starts only when the previous phase passes its exit criteria and the council gate.

Stop line (D30): the project counts as complete at the end of Phase 3. Phases 4 to 6 remain as an optional backlog.

## 4. Users

| Persona | Needs | Phase |
|---|---|---|
| Platform admin (me) | Provision tenants, create and deploy agents, see everything | 1 (scripts), 4 (console) |
| Approver | Review PRs and eval results, approve promotion | 1 |
| Tenant owner | ROI, usage, cost, chat history, manage own users | 5 |
| Tenant analyst | Read-only metrics and chat history | 5 |
| Tenant end customer | Talks to the tenant's agent only | 1 |
| Auditor or interviewer | Trace any action to request, artefact, test result and approver | 1 |

## 5. Core concepts

| Concept | Meaning |
|---|---|
| Tenant | Isolated customer or business unit with its own data, knowledge, budget and agents |
| Agent | LangGraph agent hosted on Foundry, owned by exactly one tenant |
| MCP server | Tenant-aware tool server used by agents |
| RAG knowledge base | A tenant's indexed documents, exposed to agents as an MCP tool |
| Template | Approved, versioned pattern the Factory fills in |
| Agent spec | One-page, schema-validated request for a new agent, MCP server or knowledge base |
| Attribution keys | tenant_id, agent_id, agent_version, environment, conversation_id, turn_id, graph_node, tool_name, model_deployment; present on every span the agent emits. Spans from salon-mcp and the gateway carry the keys they can know and join on the trace id (D54). Gateway metrics carry at most five of them, because the gateway metric policy allows five custom dimensions; the rest are span-only (council review 02, debate 7) |

## 6. Phase 1 MVP in detail (highest impact first)

1. **Salon agent** (LangGraph): intents book, cancel, FAQ, out of scope (reschedule dropped by D31; cancel then book covers it); FAQ answered from the tenant's knowledge with citations, and an answer citing a passage that was not returned is refused (D46); booking rules validated in code; confirmation interrupt before any write.
2. **salon-mcp** on Container Apps: search_faq (keyword search in Phase 1, D45), get_availability, create_booking, cancel_booking; tenant_id taken from the authenticated caller; idempotent writes; append-only audit log; a cancellation needs the booking reference and the contact detail held on the booking (D33).
3. **One tenant** created by an IaC module that takes tenant_id as a parameter (so Phase 2 is a second invocation, not a rewrite), with script steps for what Bicep cannot create (D54): own Foundry project (D24), index, storage partition, gateway product and token budget. An admin script runs the module and writes an audit record (D25).
4. **Gateway**: a daily token quota (D39) and token metrics keyed by tenant and agent, on API Management Basic v2 in UK West (D22). A Phase 0 spike builds a standalone instance and desk-checks Foundry's AI Gateway (D23, D47).
5. **Tenant-ready code**: tenant_id resolved from the authenticated agent identity, never from model output; unit tests prove a tool call cannot override it, and a local two-tenant test proves one tenant's identity cannot reach another's data (D49). Cross-tenant isolation tests against real infrastructure arrive in Phase 2.
6. **Telemetry**: OpenTelemetry instrumented once, exported to Application Insights and Foundry tracing, carrying the attribution keys (D54). The same traces can also be exported over OTLP to Langfuse Cloud (free tier, EU region), with synthetic data only (D7). The export is off by default and outside the exit criteria (D32).
7. **Eval gate**: offline evals (quality, task success, safety, latency and cost per conversation) block promotion on regression. One judged harness gates promotion, with scripted tests for the write intents (D35, D44).
8. **Release**: GitHub Actions with OIDC. The pipeline deploys a candidate agent version to dev automatically; promotion to the served version waits for GitHub Environment approval (D19). Each promotion is published as a GitHub Release (D43). After a full rebuild only the approved image is restored (D38).
9. **Dashboard v0**: Azure Workbook showing cost, tokens, latency percentiles and eval results by tenant and agent.
10. **Runtime guardrail**: prompt-injection screening on the agent's model path, with a negative test (D31) and a poisoned-passage test (D46).

Phase 1 exit criteria: a stranger can clone the repo, run one setup command plus one pipeline, and see one tenant's agent served, metered, evaluated and observable, with every release traceable to an approved PR. Each control is shown working by a negative demonstration, not only shown to exist (D31). Tenant provisioning traces to its audit record, not to a PR (D25).

## 7. Later phases (summary, detail moves to docs/spec-phase-N.md when each phase starts)

- **Phase 2 Multi-tenant**: second tenant through the same IaC module; isolation tests for knowledge, bookings, telemetry and gateway budgets run on every PR.
- **Phase 3 Factory**: template filling from a validated spec, never free-form code generation; generation only opens a PR; policy checks (attribution keys present, writes behind confirmation, minimum eval cases, tenant enforcement); the Factory holds no deploy credentials.
- **Phase 4 Admin Console** (FastAPI and React): Manage (tenants, agents, MCP servers, knowledge bases, budgets) and Monitor (build, test, deploy, production, cost, latency); every action audit logged; deploy requests go through CI and approval.
- **Phase 5 Tenant Portal** (FastAPI and React): ROI from outcomes times tenant-entered assumptions, always labelled as estimates; resolution, conversion, peak and after-hours usage, feedback, knowledge gaps; chat history with PII masked by default, access audited, retention per tenant, UK GDPR export and erasure; served only through a tenant-scoped data API.
- **Phase 6 Depth**: usage ledger, versioned price book, shared cost allocation, monthly statements reconciled to the Azure invoice; per-tenant SLOs and error budgets; online evals and drift; self-hosted Langfuse replacing the Phase 1 cloud export (D7); knowledge self-service with eval re-run; siloed isolation option.

## 8. Non-goals
- Voice, telephony or real-time audio
- Self-service tenant sign-up
- Tenants creating or deploying agents themselves
- Per-tenant OAuth to calendars
- Free-form code generation by the Factory
- Production-grade HA
- Payment collection
- A live (prod) environment in Phases 0 and 1 (D19)

Removed in revision 7 by D9: "Anyone, including the admin, deploying to prod outside CI and approval".

## 9. Constraints
- Solo engineer; Phase 0 two weeks, Phase 1 three weeks, stop line at the end of Phase 3 (D30, replacing about 1 week, about 2 weeks and about 10 weeks for the full roadmap)
- Personal Azure subscription, UK South preferred, gateway in UK West (D22); the gateway is torn down when idle (D28, D37); ceiling of £40 a month for the dev environment (D21)
- Preview features allowed if pinned, isolated behind one module and listed with a GA path
- Every design claim traceable to a source; reuse before build

## 10. Delivery approach (AI-native SDLC)

Follows Anthropic's AI-Native SDLC playbook: each stage commits one artifact the next stage reads, and the chain of commits is the audit trail.

| Stage | Artifact | Gate |
|---|---|---|
| Plan | intent.md | Council review plus owner acceptance |
| Design | spec-phase-N.md (D8) | Council review plus owner acceptance |
| Build | implementation-plan-phase-N.md (D8), code, tests | Plan accepted before code; CI green |
| Test | eval results | Eval gate thresholds |
| Deploy | PR with review findings | REVIEW.md passes, human approval |
| Maintain | new intent.md from breached control bands | Triage by owner |

**Council**: at every gate, six reviewer subagents review the artifact independently and give one anonymised rebuttal, then a chair synthesises a verdict with dissent recorded in docs/council/. Members: Platform Architect, Security and Compliance, AI Engineer (added by D15), Simplifier, Hiring Manager, Contrarian. Pattern reused from Karpathy's LLM Council as adapted for Claude Code subagents (see section 12).

**Tooling**: Claude Code on the web for intent to PR; Claude Code Desktop for one-time Azure bootstrap and live debugging; GitHub Actions is the standard release path. Sessions may also run Azure write commands under the assessment rule in D9, as amended by D13, D14, D20, D21 and D40. A session never approves a deployment (D41).

**Before Build can start (Phase 0 deliverables)**: CLAUDE.md, REVIEW.md, skills (tenant isolation, MCP security, telemetry attribution, IaC conventions), hooks (block test edits during fixes, block secrets; the Azure write assessment hook was removed by D20), council and verifier subagents, .mcp.json (Azure MCP, Microsoft Learn MCP, Context7; Playwright deferred to Phase 4, D10), cloud environment setup script, seed eval set, ADR folder.

**Delivered in Stage 0 (harness session, 2026-10-03)**: git repository and .gitignore, CLAUDE.md (process rules only), council subagents and the /council command, .mcp.json, the session journal in docs/journal/ and the /cleanup skill (D17). The remaining Phase 0 deliverables are specified in docs/spec-phase-0-1.md.

## 11. Research findings (summary)

Full report: docs/research.md.

**Already done?** Partly. Every runtime piece has an official sample; nobody publishes the integrated, governed, multi-tenant version with factory, consoles and per-tenant economics. That integration is the differentiator.

**Reuse map**

| Need | Take from | How |
|---|---|---|
| LangGraph on Foundry hosted agents | microsoft-foundry/foundry-samples; langchain-ai/langchain-azure hosting samples | Copy hosting pattern and HITL sample |
| AI gateway policies | Azure-Samples/AI-Gateway | Copy Bicep and policy XML |
| MCP on Container Apps with auth | Azure-Samples/python-mcp-demos | Fork as salon-mcp base |
| Eval gate | microsoft/ai-agent-evals; DeepEval | Use Action pinned by SHA; DeepEval only as the fallback gate (D35) |
| Factory structure | GoogleCloudPlatform/agent-starter-pack; Backstage templates | Copy structure and approval ideas, not code |
| Council | karpathy/llm-council; llm-council Claude Code skill | Adapt as subagents and a /council command |
| Spec and plan templates | Anthropic playbook; GitHub Spec Kit | Playbook artifact chain; borrow Spec Kit's clarifications section |
| Langfuse export (Phase 1, D7) | Langfuse Cloud free tier, OTLP endpoint | Second OTLP exporter, dev only |
| Langfuse self-hosted (Phase 6) | Langfuse Docker Compose | Single VM |

**Avoid**: Azure-Samples/langfuse-on-azure (archived, Langfuse v2); the older from_langgraph adapter.

**Key technical decisions so far**: langchain_azure_ai.agents.hosting; RAG behind MCP; tenant_id from caller identity only; APIM v2 tier for llm-token-limit; agent graph kept independent of the hosting adapter so it can also run on Container Apps if Foundry hosting changes.

## 12. Preview components and GA path

| Component | Status | Fallback |
|---|---|---|
| ai-agent-evals Action | v3-beta | Pin SHA; DeepEval-only gate |
| AI Search knowledge base MCP endpoint | Preview API | Own MCP server on GA search API |
| APIM fronting MCP servers on v2 | Preview | Direct Entra-authenticated MCP calls |
| Hosted agent private networking | Endpoint stays public | Entra auth; documented gap |
| Foundry AI Gateway integration (D23, D47) | Preview; desk check only unless the standalone path fails | Standalone API Management instance |
| Azure MCP server, @azure/mcp (council review 02, debate 7) | 3.0.0-beta.49 | Pinned version; az and azd |
| Hosting SDK: langchain-azure-ai hosting extra and the azd azure.ai.agents extension (council review 02, debate 7) | Extension is beta | Pinned versions; thin adapter so the graph can run on Container Apps |

## 13. Risks
- Scope: five subsystems for one engineer; mitigated by strict phase gates and a self-contained MVP
- Single-tenant MVP hides isolation bugs until Phase 2; mitigated by tenant-ready code, override tests and a local two-tenant test in Phase 1 (D49)
- Cross-tenant leakage; mitigated by caller-derived tenant_id, override unit tests and a local two-tenant test in Phase 1 (D49), and cross-tenant isolation tests on real infrastructure from Phase 2
- SDK churn in Foundry hosting; mitigated by a thin adapter
- Cost growth per tenant; nightly gateway teardown (D37), free tiers where possible, £40 a month ceiling for dev (D21)
- PII in traces and transcripts; masking at the collector, retention and access audit
- ROI overclaiming; estimates labelled with assumptions
- Regional availability: hosted agents are listed for UK South; current models are offered there as Global Standard only (D34); API Management v2 tiers cannot currently be created in UK South, so the gateway is in UK West (D22). Checked in the documentation on 2026-10-03, still to confirm in the portal
- No enforced control between a session and a live environment (D20): accepted by the owner, and dormant while dev is the only environment (D19)
- A session uses the owner's GitHub login, which can approve a promotion (D41): accepted by the owner; the rule against it is written, not enforced

## 14. Questions for the owner

### Decided (2026-10-03)
- **D1**: MVP serves one tenant first, built tenant-ready; multi-tenancy becomes Phase 2. (Council dissent recorded: docs/council/01-intent-review.md) (Amended by D49.)
- **D2**: Factory's second agent is regulated document Q&A.
- **D3**: Admin Console and Tenant Portal use Python FastAPI and React.
- **D4**: Models are Azure OpenAI only: a small model for classification and extraction, a mid model for answers. Exact versions fixed after confirming UK South availability. (Completed by D34 and D50.)
- **D5**: Google Calendar sync deferred beyond Phase 1.
- **D6**: Claude plan is Pro or Max. Cloud sessions may hold a dev model key as an environment API credential (never visible to Claude). Production credentials exist only in GitHub Actions via OIDC. (Amended by D20.)
- **Defaults for Phase 1 unless the owner objects**: synthetic salon data; named stylists with an "any available" option; dev environment only (D19, replacing "dev and prod environments only"); eval thresholds proposed in spec and retuned after baseline.

### Decided in the Stage 0 harness interview (2026-10-03)
- **D7**: Langfuse moves into Phase 1 as an export only. Langfuse Cloud free tier (EU region; 50k units a month, 30 days of data, 2 users) is added as a second OTLP exporter in the dev environment, with synthetic data only and no new Azure infrastructure. Self-hosting and question 15 stay in Phase 6. This revises debate 3 in docs/council/01-intent-review.md. Sources: https://langfuse.com/pricing and https://langfuse.com/integrations/native/opentelemetry (Amended by D32.)
- **D8**: Artifact names and layout. The chain is docs/intent.md, then docs/spec-phase-N.md, then docs/implementation-plan-phase-N.md, then code, then PR. Files are kept per phase rather than overwritten. Phases 0 and 1 share docs/spec-phase-0-1.md and docs/implementation-plan-phase-0-1.md. "implementation-plan" replaces the playbook's "plan.md".
- **D9**: Claude Code sessions may run Azure write commands in any environment, including prod, through az, azd or the Azure MCP server. Before each write Claude states the estimated added monthly cost and the blast radius. It proceeds without asking only when all of these hold: the estimate is up to £20 a month; the command touches only this project's resource groups; it deletes nothing; it makes no role, policy or Entra change outside the bootstrap. Anything else waits for the owner's yes. This removes the non-goal on deploying to prod outside CI and replaces the earlier rule that only GitHub Actions deploys. GitHub Actions with OIDC and environment approval (section 6, item 8) remains the standard release path. No deny rules or blocking hooks for Azure commands were added in Stage 0; how the assessment is enforced mechanically is a question for the Phases 0 and 1 spec. Consequence to weigh in the spec: the "pipelines deploy, humans approve" story now rests on the standard path and the audit trail, not on a hard block. (Amended by D13, D20, D21 and D40.)
- **D10**: .mcp.json holds Microsoft Learn MCP, Context7 (added to the original list) and Azure MCP, pinned to @azure/mcp 3.0.0-beta.49 with write tools enabled under D9. Playwright is deferred to Phase 4.
- **D11**: Left open for the spec, to be raised in its clarifications section: the bookings and conversation store (Cosmos DB serverless or PostgreSQL), and questions 5, 6 and 7 below. The note in docs/research.md that the store was "later revised to PostgreSQL in spec" refers to a spec that does not exist and has no standing; the note was removed in revision 9. (Settled by D23, D25, D26 and D27.)
- **D12**: The repository is private under github.com/ranaidrees. Commits are authored as Rana Idrees with the account's GitHub noreply address.

### Decided after council review 02 (2026-10-03)
- **D13**: Amends D9 and answers question 1 of docs/council/02-intent-review.md. Sessions may still write to prod, but every prod write, through any tool, waits for the owner's yes after the cost and blast radius assessment. az and azd writes in dev keep the D9 low-risk rule and run without asking when it is met. The council's proposal (dev only, enforced by Azure RBAC) was not adopted.
- **D14**: Amends D10 and answers question 2 of review 02. Azure MCP write tools stay enabled, but every Azure MCP write, in any environment, waits for the owner's yes. This is a written rule in CLAUDE.md only; the owner chose not to add a Claude Code permission rule that would force a prompt.
- **D15**: The council has six members. An AI Engineer is added to cover agent design, retrieval, evaluation and runtime guardrails, the gap the smoke test exposed, and each member's brief now names the expertise it brings to this project and goal. The cost of a council run (about 1.6 million subagent tokens with five members) is accepted.
- **D16**: Repository visibility is under review (question 3 of review 02). The owner prefers private but will make the repository public if the GitHub plan restricts what the project needs. Facts: on GitHub Free, environments exist only on public repositories; on Free, Pro and Team, required reviewers and wait timers exist only on public repositories; protected branches on private repositories need GitHub Pro. Sources: https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/manage-environments and https://docs.github.com/en/get-started/learning-about-github/githubs-plans. Until the owner decides, the repository stays private and the spec must not assume a required-reviewer gate.
- **D17**: Two harness additions. Each session writes a journal entry in docs/journal/ from a template, and starts from the handoff in the previous entry. A /cleanup skill reports broken, stale, duplicated, unused or unnecessary files and changes nothing without the owner's approval; it runs before the final commit of each stage. No document is added outside the artifact chain, council reviews, journal and harness configuration unless the owner asks.
- **D18**: Settles D16 and answers question 3 of review 02. The owner decided to make the repository public, so that environments, required reviewers, protected branches and GitHub's free security scanning are available to the project. The repository and its commit messages were checked first for secrets and personal identifiers and none were found. The owner made the visibility change in GitHub settings on 2026-10-03, because a session is not permitted to publish a repository; the repository is now public. Azure subscription and tenant identifiers are to be kept in GitHub variables or secrets, not in files.

### Decided in the Stage 2 design interview (2026-10-03)
Question numbers in D19 to D36 refer to docs/council/02-intent-review.md unless they say "below". Prices are from the Azure Retail Prices API on 2026-10-03.
- **D19**: Environments and release gate. Dev is the only environment in Phases 0 and 1. The owner builds and demonstrates on dev, and a live environment is an optional later decision. The release gate sits inside dev: the pipeline deploys a candidate agent version, runs the eval gate against a session pinned to that candidate, and a GitHub Environment approval guards promotion to the served version. The owner is the single approver, which the spec states plainly, and administrator bypass of the environment rule is disabled. Question 5 does not arise, because there is one gateway instance. Sources: https://learn.microsoft.com/azure/foundry/agents/how-to/manage-hosted-agent#release-a-version-without-changing-production and https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments
- **D20**: Session identity and prod credentials. Amends D6, D13 and D14. Claude Code sessions use the owner's own Azure CLI login throughout. D6 is amended: the owner's login on the Desktop is a second prod credential beside GitHub Actions, and cloud sessions never hold a prod credential. The approval rules in D13 and D14 stay written rules only. The owner chose no hook, no Claude Code permission rule, no separate session identity, no activity log alert and no delete lock, because the priority is that Claude Code works without friction. Azure RBAC least privilege applies to workload and pipeline identities, not to sessions. This is an accepted risk: nothing mechanical stands between a session and a live environment. It is dormant while dev is the only environment (D19). The Azure write assessment hook is removed from the Phase 0 deliverables. Options considered: https://code.claude.com/docs/en/hooks and https://code.claude.com/docs/en/permissions
- **D21**: Budget. Amends D9 and answers the second part of question 12. The whole dev environment has a ceiling of £40 a month. A dev write runs without asking only if it meets the D9 test and the month's forecast total stays under £40; otherwise the session asks.
- **D22**: Gateway region and tier. Answers the first half of question 4. Azure API Management Basic v2 in UK West, because no v2 tier can currently be created in UK South. It costs £0.1551 for each hour it exists and cannot be paused, so it is created on demand and deleted and purged when idle, with a nightly teardown workflow as a backstop. Everything else stays in UK South. Sources: https://learn.microsoft.com/en-us/azure/api-management/api-management-region-availability and https://learn.microsoft.com/azure/api-management/soft-delete (Amended by D37.)
- **D23**: Model path. Answers question 7 below. A hosted agent can reach models through its project endpoint without any gateway, so a gateway budget binds only if that path is closed. A Phase 0 spike tries two designs against the same negative test, a standalone API Management instance and Foundry's own AI Gateway integration, and the choice follows the evidence. Sources: https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/hosted-agent-permissions and https://learn.microsoft.com/en-us/azure/foundry/configuration/enable-ai-api-management-gateway-portal (Amended by D47.)
- **D24**: One Foundry project per tenant. Answers the second half of question 4.
- **D25**: Tenant provisioning. Answers question 5 below, against the council's default. A tenant is provisioned by an admin script that wraps the parameterised IaC module of section 6, item 3, and writes an append-only record (who, when, tenant, parameter hash, deployment id) to the platform audit store. The Azure Activity Log is the second witness. Provisioning is therefore outside the PR and approval trail.
- **D26**: Data store. Settles D11 and question 8 below. Azure Table Storage holds bookings and the audit log. Conversation state uses Foundry's durable state store through FoundryCheckpointSaver, subject to a Phase 0 spike; the fallback is CosmosDBSaver on Cosmos DB serverless. Sources: https://learn.microsoft.com/rest/api/storageservices/authorize-with-azure-active-directory and https://github.com/langchain-ai/langchain-azure/blob/main/libs/azure-ai/README.md
- **D27**: Knowledge index. Answers question 6 below and the third part of question 12. One index per tenant on the Azure AI Search Free tier (3 indexes, 50 MB), with role-based access only. This is subject to a Phase 0 spike, because Microsoft's pages disagree on whether keyless access works on the Free tier. The fallback is the Basic tier created on demand at £0.0762 an hour. Sources: https://learn.microsoft.com/azure/search/search-limits-quotas-capacity and https://learn.microsoft.com/azure/search/search-security-enable-roles
- **D28**: Teardown and evidence. Answers the first part of question 12. Tearing down deletes the whole environment. Evidence survives in a small persistent resource group that teardown never touches: the Log Analytics workspace and Application Insights that telemetry is written to, the pipeline identity, and storage for exported audit records and eval results. (Amended by D37, D43 and D51.)
- **D29**: Synthetic data only in every environment until Phase 5. Answers question 11.
- **D30**: Schedule and stop line. Answers questions 6 and 7. Phase 0 is two weeks and Phase 1 is three weeks with the cuts in D31. If a time-box is at risk, scope is cut in an order the spec states and the date holds. The stop line is the end of Phase 3: the project counts as complete there, and Phases 4 to 6 remain as an optional backlog. (Completed by D48.)
- **D31**: Scope changes for Phases 0 and 1. Answers question 9. Added: exit criteria that show each control working through a negative demonstration, and a runtime prompt-injection guardrail with its own negative test. Removed: the reschedule intent. Not adopted from the council: per-phase demo scripts, a control matrix, a chat page and a regulated skin for Phase 1 (the regulated agent stays in Phase 3, D2). (Amended by D46.)
- **D32**: Langfuse. Amends D7 and answers question 10. The export stays in Phase 1 as an optional second exporter, off by default, switched on by a dev parameter and the owner's own Langfuse keys. It is outside the exit criteria and outside the one setup command.
- **D33**: Callers and customer authorisation. Answers question 8. The only callers of the Phase 1 agent are named Entra test identities, the owner and the pipeline identity, each holding the Foundry Agent Consumer role scoped to the agent. There is no public or anonymous caller; a middle tier that passes on an end user belongs to Phase 4. To cancel a booking a customer must give the booking reference and the contact detail held on the booking. salon-mcp checks both server-side, and the model never decides. Source: https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/hosted-agent-permissions
- **D34**: Models. Completes D4. Model deployments are Global Standard in UK South, because current models are not offered there as regional deployments and UK South is outside the EU data zone. Prompts may be processed in any Azure region, which D29 makes acceptable. The spec records this as a residency gap, with regional or provisioned deployment as the path for real data, and fixes the exact models. Source: https://learn.microsoft.com/azure/foundry/foundry-models/concepts/models-sold-directly-by-azure-region-availability (Completed by D50.)
- **D35**: Eval gate. One harness gates promotion: the microsoft/ai-agent-evals GitHub Action, pinned by commit SHA, comparing the candidate agent version with the served one. A DeepEval-only gate is the named fallback, built only if the Action fails its Phase 0 spike. Deterministic rules (tenant override, booking validation, confirmation before writes) are ordinary pytest tests. Source: https://github.com/microsoft/ai-agent-evals (Amended by D44.)
- **D36**: Phase 0 harness. The secrets hook, the test-edit hook, the four skills and the verifier subagent all stay in Phase 0. Only the Azure write assessment hook is removed (D20).

### Decided at the Stage 2 spec gate (2026-10-03)
Question numbers in D37 to D55 refer to docs/council/03-spec-phase-0-1-review.md, unless they say "spec question", which means section 12 of the draft spec (commit e13557b). The owner answered questions 1 to 10 directly, then asked for the session's recommendation to stand on everything still open; D47 to D55 record those recommendations. Prices are from the Azure Retail Prices API on 2026-10-03.
- **D37**: Nightly teardown removes only the gateway. Amends D22 and D28 and answers question 1. The nightly workflow deletes and purges the gateway, and the Basic search service if spike S4 falls back to it, because nothing else in the environment bills for existing. Agent versions, identities and rollback targets therefore survive. A full teardown stays as a manual command, used for the rebuild spike and the stranger test. Evidence still lives in the persistent group. Sources: https://learn.microsoft.com/azure/foundry/agents/concepts/hosted-agents and https://learn.microsoft.com/azure/api-management/soft-delete
- **D38**: A rebuild restores the approved image. Answers question 2 and the first point of question 13. On a full rebuild, `up` redeploys only the image digest named in the latest release record and writes a restore record. It never builds the agent from the working tree. Release records are keyed by image digest. On a clean clone nothing is served until the first pipeline run, which is recorded as the bootstrap release and judged on absolute thresholds, there being no served baseline. Source: https://learn.microsoft.com/azure/foundry/agents/how-to/manage-hosted-agent
- **D39**: Token budget. Answers question 7 and the token figures in spec question 4. The gateway enforces a daily quota of 150,000 tokens for each tenant and agent, and 20,000 tokens a minute. The owner chose 150,000 to keep cost at a minimum. The quota is a deploy parameter. A monthly figure of 2,000,000 is reported from persisted metrics with an alert, because the counter lives in the gateway and is expected to restart when the gateway is purged. A test identity with a quota of 2,000 tokens makes the refusal demonstration cheap. One gate run is estimated, not measured, at about 161,000 tokens; spike S6 measures it, and if two runs do not fit in a day the owner chooses again with the measured figure. Source: https://learn.microsoft.com/en-us/azure/api-management/llm-token-limit-policy
- **D40**: Session teardown. Amends D9 and D13. A session may run the repository's gateway teardown in dev without asking. A full teardown still waits for the owner's yes, and no session touches the persistent group. This is a written rule in CLAUDE.md, with no hook behind it (D20).
- **D41**: Approval credential. Extends D20 and answers question 3. A token with the `repo` scope can approve a pending deployment, and sessions use the owner's GitHub login. A session never approves or rejects a deployment; the owner approves in the browser. This is a written rule only and an accepted risk: the gate records a deliberate act by the owner's account, not proof that a person made it. The council's proposal of a separate session token was not adopted. Source: https://docs.github.com/en/rest/actions/workflow-runs?apiVersion=2022-11-28#review-pending-deployments-for-a-workflow-run
- **D42**: Gate additions. Answers question 4. Adopted: both GitHub environments restricted to `main`; a check, in the pipeline and nightly, that the served image is the one in the latest approved release record; a separate teardown identity holding a purge-only custom role; every Action pinned by full commit SHA, with a read-only default token. The spec also gains a role table for each identity and states that GitHub, not Azure, enforces the promotion gate, because one Foundry permission both creates a version and moves the served selector. Not adopted: holding salon-mcp revisions at zero traffic until promotion, which stays in Phase 2, and a name suffix for each rebuild. Sources: https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/hosted-agent-permissions and https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments
- **D43**: Release evidence. Amends D28 and answers question 8. Each promotion publishes its release record, eval summary and refusal results as assets of a GitHub Release, with immutable releases switched on. The copy in the persistent storage account stays. An immutable blob container was not adopted, in keeping with D20's choice of no delete lock. Source: https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/immutable-releases
- **D44**: Eval gate. Amends D35 and answers questions 5 and 6 and the eval figures in spec question 4. The microsoft/ai-agent-evals Action stays the only judged harness and judges the single-turn intents. Booking and cancelling, which stop at a confirmation the Action cannot answer, are gated by scripted conversations on the pinned candidate that assert the end state. Both must pass. Thresholds are measured before they are fixed: spike S6 runs one version twice and one subtle regression, the thresholds follow the measured noise, and the spec then states the smallest regression the gate detects. The dataset has 20 rows for each judged intent. Cost and latency stay in the gate, computed from spans. Pass rates of 0.80 and a p95 latency of 20 seconds are starting figures only. Source: https://learn.microsoft.com/azure/foundry/how-to/evaluation-github-action
- **D45**: Retrieval. Answers question 9. `search_faq` uses keyword search in Phase 1. The embedding deployment, the precomputed vectors and the call from salon-mcp to the gateway leave Phase 1. Vectors return in Phase 3 with the document agent (D2). The AI Engineer's objection, that this is an unmeasured quality cut, is recorded.
- **D46**: Agent additions. Amends D31 and answers question 10. Added: a validator node that refuses an answer citing a passage that was not returned; pinned model versions; ten unanswerable questions in the eval set; one poisoned-passage test that asserts no tool call. Not adopted: a recall measure, a labelled attack set, and content recording in eval sessions. Source: https://learn.microsoft.com/azure/foundry/openai/how-to/working-with-models#model-deployment-upgrade-configuration
- **D47**: Model path spike. Amends D23 and answers the first part of question 11 and spec question 5. Spike S1 builds the standalone gateway. Foundry's AI Gateway gets a desk check for an API or Bicep route, because its only documented setup is through the portal and the gateway is rebuilt from the repository every working day. It is built only if the standalone path fails the bypass test and such a route exists. Spike S2 runs before S1. Source: https://learn.microsoft.com/en-us/azure/foundry/configuration/enable-ai-api-management-gateway-portal
- **D48**: Cut orders. Completes D30 and answers the second part of question 11 and the third point of question 13. If the Phase 0 time-box is at risk, scope is cut in this order: the cloud environment setup script, the verifier subagent, the telemetry and IaC skills, the test-edit hook. Cut items move to after Phase 1; D36 otherwise stands. Never cut: the bootstrap, CI with OIDC, repository protections, the secrets hook and scanning, the seed eval set, and spikes S1, S2, S5 and S6. In Phase 1 the Langfuse exporter (D32) is the first cut.
- **D49**: Local two-tenant test. Amends D1 and answers question 12. Phase 1 adds a local test with two registered identities and two tenants' data: one tenant's identity cannot read or write the other's bookings or index. It needs no second tenant in Azure. Isolation tests against real infrastructure still arrive in Phase 2.
- **D50**: Models. Completes D4 and D34 and answers spec question 1. gpt-5.4-nano for classification and extraction and gpt-5.4-mini for answers, both version 2026-03-17, Global Standard in UK South, retiring on 2027-09-21. No embedding model in Phase 1 (D45). The subscription's quota for both models was checked on 2026-10-03. Sources: https://learn.microsoft.com/azure/foundry/foundry-models/concepts/models-sold-directly-by-azure-region-availability and https://learn.microsoft.com/azure/foundry/openai/concepts/model-retirement-schedule
- **D51**: Persistent group. Extends D28 and answers spec question 2. The persistent group also holds the audit table, the container registry, the workload identity for salon-mcp, the teardown identity, the Workbook and the cost budget.
- **D52**: salon-mcp. Answers spec questions 3 and 7. The agent calls salon-mcp directly with its own Entra identity; a Foundry toolbox connection is the fallback if spike S2 fails. salon-mcp has a public endpoint in Phase 1, protected by an Entra token on every call. Private networking stays out of scope and is recorded as a gap.
- **D53**: Points raised only in rebuttal. Answers question 13. The spec addresses all six: the cold start (D38); the Langfuse exporter in the cut order (D48); one source of truth for the identity registry, the table written by the admin script, with the gateway's copy generated from it and checked; model quota (D50); eval runs that use relative dates and reserved slots and report a quota refusal as an error, not a regression; and safety evaluation. Risk and safety evaluators are not offered in UK South, so spike S6 tests whether one can run from a UK South project; if not, safety in the gate rests on the guardrail's negative test and the poisoned-passage test. Source: https://learn.microsoft.com/azure/foundry/concepts/evaluation-regions-limits-virtual-network
- **D54**: Unopposed corrections from review 03, debate 7. All nine attribution keys are required on spans the agent emits; spans from salon-mcp and the gateway carry the keys they can know and join on the trace id. The tenant module is split into Bicep and script steps, because Bicep cannot create a search index. The bootstrap names the Entra app registrations for the gateway and salon-mcp audiences. Source: https://learn.microsoft.com/azure/search/search-get-started-bicep
- **D55**: Process. "Research before the interview" becomes a rule in CLAUDE.md, having appeared as a lesson in two journal entries.

### Needed before later phases (council's proposed default in brackets)
5. Tenant provisioning: see D25.
6. Index per tenant or shared index: see D27.
7. Model path: see D23.
8. Conversation store: see D26.
9. Default transcript retention? [90 days]
10. Shared cost allocation method? [Token share]
11. Price book source? [Azure Retail Prices API, pinned monthly]
12. SLO targets? [Derived from Phase 1 baseline]
13. Statements at cost or with margin? [At cost, margin as a toggle]
14. "Resolved without staff": agent outcome alone or confirmed by feedback? [Agent outcome, feedback shown alongside]
15. Langfuse: one project per tenant or tags? [Tags]

## 15. References
- Anthropic, The AI-Native SDLC playbook: https://claude.com/blog/the-ai-native-sdlc-playbook
- Karpathy, LLM Council: https://github.com/karpathy/llm-council
- LLM Council as a Claude Code skill: https://github.com/tenfoldmarc/llm-council-skill
- GitHub Spec Kit: https://github.com/github/spec-kit
- Claude Code cloud environments: https://code.claude.com/docs/en/cloud-environments
- OWASP MCP Top 10: https://owasp.github.io/www-project-mcp-top-10/
- APIM llm-token-limit: https://learn.microsoft.com/en-us/azure/api-management/llm-token-limit-policy
- Foundry hosted agents networking: https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/virtual-networks
- Langfuse pricing and regions: https://langfuse.com/pricing
- Langfuse OpenTelemetry endpoint: https://langfuse.com/integrations/native/opentelemetry
- Microsoft Learn MCP Server: https://learn.microsoft.com/en-us/training/support/mcp
- Context7: https://github.com/upstash/context7
- Azure MCP Server tools and start options: https://learn.microsoft.com/en-us/azure/developer/azure-mcp-server/tools/
- Claude Code subagents: https://code.claude.com/docs/en/sub-agents
- Claude Code MCP configuration: https://code.claude.com/docs/en/mcp
- Hosted agent permissions: https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/hosted-agent-permissions
- Hosted agent release without changing production: https://learn.microsoft.com/azure/foundry/agents/how-to/manage-hosted-agent#release-a-version-without-changing-production
- API Management v2 region availability: https://learn.microsoft.com/en-us/azure/api-management/api-management-region-availability
- APIM llm-emit-token-metric: https://learn.microsoft.com/en-us/azure/api-management/llm-emit-token-metric-policy
- Foundry AI Gateway: https://learn.microsoft.com/en-us/azure/foundry/configuration/enable-ai-api-management-gateway-portal
- GitHub deployments and environments: https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments
- Foundry model region availability: https://learn.microsoft.com/azure/foundry/foundry-models/concepts/models-sold-directly-by-azure-region-availability
- API Management soft-delete and purge: https://learn.microsoft.com/azure/api-management/soft-delete
- GitHub REST, review pending deployments: https://docs.github.com/en/rest/actions/workflow-runs?apiVersion=2022-11-28#review-pending-deployments-for-a-workflow-run
- GitHub immutable releases: https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/immutable-releases
- Foundry evaluation in GitHub Actions: https://learn.microsoft.com/azure/foundry/how-to/evaluation-github-action
- Foundry evaluation region support: https://learn.microsoft.com/azure/foundry/concepts/evaluation-regions-limits-virtual-network
- Azure Retail Prices API: https://learn.microsoft.com/rest/api/cost-management/retail-prices/azure-retail-prices
