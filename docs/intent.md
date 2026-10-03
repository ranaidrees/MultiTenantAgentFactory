# Intent: Multi-tenant AgentOps platform on Azure with Agent Factory

Author: Rana Naveed Idrees. Status: accepted (revision 9). Owner decisions D1 to D17 recorded in section 14. Date: 2026-10-03.
Stage: 1 of 6 (Plan). Location: docs/intent.md. Next artifact: docs/spec-phase-0-1.md (Phases 0 and 1).
Council record: docs/council/01-intent-review.md (revision 4) and docs/council/02-intent-review.md (revision 7)
Revision 7 records the Stage 0 harness interview (D7 to D12) and amends the text those decisions contradict; each amendment is marked with its decision number.
Revision 8 records the owner's response to council review 02 (D13 to D17). The review's other questions remain open for the spec.
Revision 9 applies three cleanup corrections approved by the owner: the isolation test phases in section 13, question 4 removed (answered by D6) and question 8 pointed at D11.

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
| 1 | **MVP: governed agent, one tenant, tenant-ready** | Salon agent and salon-mcp (with RAG) for one tenant; tenant identity bound to the agent and enforced server-side; APIM token budget; OpenTelemetry with full attribution keys, also exported to Langfuse Cloud in dev (D7); offline eval gate; dev to prod with approval; Workbook dashboard | "Governance, cost and quality are enforced by the platform, not the agent" |
| 2 | Multi-tenant | Second tenant via the same IaC; automated isolation tests in CI; per-tenant views on the dashboard | "Isolation is proven by tests, not claimed" |
| 3 | Agent Factory | Spec schema, templates (agent, MCP server, RAG knowledge base), generator that opens a PR, CI policy checks; **regulated document Q&A agent** shipped through the same gate | "New agents inherit the golden path; humans approve, pipelines deploy" |
| 4 | Admin Console | FastAPI and React on Azure: provision tenants, request agents, request deployments, lifecycle dashboard | "One control plane for tenants and agents" |
| 5 | Tenant Portal | FastAPI and React, Entra External ID: ROI, outcomes, chat history with masking and audit, knowledge gaps | "Clients see value and evidence, never each other" |
| 6 | Depth | Ledger and statements, SLOs and error budgets, online evals and drift, self-hosted Langfuse (D7), knowledge self-service, siloed isolation | "Chargeback, reliability engineering, continuous quality" |

Rule: a phase starts only when the previous phase passes its exit criteria and the council gate.

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
| Attribution keys | tenant_id, agent_id, agent_version, environment, conversation_id, turn_id, graph_node, tool_name, model_deployment; present on every span and gateway call |

## 6. Phase 1 MVP in detail (highest impact first)

1. **Salon agent** (LangGraph): intents book, reschedule, cancel, FAQ, out of scope; FAQ answered from the tenant's knowledge with citations; booking rules validated in code; confirmation interrupt before any write.
2. **salon-mcp** on Container Apps: search_faq, get_availability, create_booking, cancel_booking; tenant_id taken from the authenticated caller; idempotent writes; append-only audit log.
3. **One tenant** created by an IaC module that takes tenant_id as a parameter (so Phase 2 is a second invocation, not a rewrite): own index, storage partition, APIM product and token budget.
4. **Gateway**: APIM token limit and token metrics keyed by tenant and agent.
5. **Tenant-ready code**: tenant_id resolved from the authenticated agent identity, never from model output; unit tests prove a tool call cannot override it. Cross-tenant isolation tests arrive in Phase 2.
6. **Telemetry**: OpenTelemetry instrumented once, exported to Application Insights and Foundry tracing, carrying all attribution keys. In dev only, the same traces are also exported over OTLP to Langfuse Cloud (free tier, EU region), with synthetic data only (D7).
7. **Eval gate**: offline evals (quality, task success, safety, latency and cost per conversation) block promotion on regression.
8. **Release**: GitHub Actions with OIDC; dev automatic, prod via GitHub Environment approval.
9. **Dashboard v0**: Azure Workbook showing cost, tokens, latency percentiles and eval results by tenant and agent.

Phase 1 exit criteria: a stranger can clone the repo, run one setup command plus one pipeline, and see one tenant's agent served, metered, evaluated and observable, with every release traceable to an approved PR.

## 7. Later phases (summary, detail moves to spec.md when each phase starts)

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

Removed in revision 7 by D9: "Anyone, including the admin, deploying to prod outside CI and approval".

## 9. Constraints
- Solo engineer; Phase 0 about 1 week, MVP about 2 weeks; full roadmap about 10 weeks
- Personal Azure subscription, UK South preferred, tear down when idle
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

**Tooling**: Claude Code on the web for intent to PR; Claude Code Desktop for one-time Azure bootstrap and live debugging; GitHub Actions is the standard release path. Sessions may also run Azure write commands under the assessment rule in D9, as amended by D13 and D14.

**Before Build can start (Phase 0 deliverables)**: CLAUDE.md, REVIEW.md, skills (tenant isolation, MCP security, telemetry attribution, IaC conventions), hooks (Azure write assessment per D9, block test edits during fixes, block secrets), council and verifier subagents, .mcp.json (Azure MCP, Microsoft Learn MCP, Context7; Playwright deferred to Phase 4, D10), cloud environment setup script, seed eval set, ADR folder.

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
| Eval gate | microsoft/ai-agent-evals; DeepEval | Use Action pinned by SHA; DeepEval for graph tests |
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

## 13. Risks
- Scope: five subsystems for one engineer; mitigated by strict phase gates and a self-contained MVP
- Single-tenant MVP hides isolation bugs until Phase 2; mitigated by tenant-ready code and override tests in Phase 1
- Cross-tenant leakage; mitigated by caller-derived tenant_id, override unit tests in Phase 1 and cross-tenant isolation tests from Phase 2
- SDK churn in Foundry hosting; mitigated by a thin adapter
- Cost growth per tenant; nightly teardown, free tiers where possible
- PII in traces and transcripts; masking at the collector, retention and access audit
- ROI overclaiming; estimates labelled with assumptions
- UK South availability of hosted agents and models to confirm in the portal

## 14. Questions for the owner

### Decided (2026-10-03)
- **D1**: MVP serves one tenant first, built tenant-ready; multi-tenancy becomes Phase 2. (Council dissent recorded: docs/council/01-intent-review.md)
- **D2**: Factory's second agent is regulated document Q&A.
- **D3**: Admin Console and Tenant Portal use Python FastAPI and React.
- **D4**: Models are Azure OpenAI only: a small model for classification and extraction, a mid model for answers. Exact versions fixed after confirming UK South availability.
- **D5**: Google Calendar sync deferred beyond Phase 1.
- **D6**: Claude plan is Pro or Max. Cloud sessions may hold a dev model key as an environment API credential (never visible to Claude). Production credentials exist only in GitHub Actions via OIDC.
- **Defaults for Phase 1 unless the owner objects**: synthetic salon data; named stylists with an "any available" option; dev and prod environments only; eval thresholds proposed in spec and retuned after baseline.

### Decided in the Stage 0 harness interview (2026-10-03)
- **D7**: Langfuse moves into Phase 1 as an export only. Langfuse Cloud free tier (EU region; 50k units a month, 30 days of data, 2 users) is added as a second OTLP exporter in the dev environment, with synthetic data only and no new Azure infrastructure. Self-hosting and question 15 stay in Phase 6. This revises debate 3 in docs/council/01-intent-review.md. Sources: https://langfuse.com/pricing and https://langfuse.com/integrations/native/opentelemetry
- **D8**: Artifact names and layout. The chain is docs/intent.md, then docs/spec-phase-N.md, then docs/implementation-plan-phase-N.md, then code, then PR. Files are kept per phase rather than overwritten. Phases 0 and 1 share docs/spec-phase-0-1.md and docs/implementation-plan-phase-0-1.md. "implementation-plan" replaces the playbook's "plan.md".
- **D9**: Claude Code sessions may run Azure write commands in any environment, including prod, through az, azd or the Azure MCP server. Before each write Claude states the estimated added monthly cost and the blast radius. It proceeds without asking only when all of these hold: the estimate is up to £20 a month; the command touches only this project's resource groups; it deletes nothing; it makes no role, policy or Entra change outside the bootstrap. Anything else waits for the owner's yes. This removes the non-goal on deploying to prod outside CI and replaces the earlier rule that only GitHub Actions deploys. GitHub Actions with OIDC and environment approval (section 6, item 8) remains the standard release path. No deny rules or blocking hooks for Azure commands were added in Stage 0; how the assessment is enforced mechanically is a question for the Phases 0 and 1 spec. Consequence to weigh in the spec: the "pipelines deploy, humans approve" story now rests on the standard path and the audit trail, not on a hard block.
- **D10**: .mcp.json holds Microsoft Learn MCP, Context7 (added to the original list) and Azure MCP, pinned to @azure/mcp 3.0.0-beta.49 with write tools enabled under D9. Playwright is deferred to Phase 4.
- **D11**: Left open for the spec, to be raised in its clarifications section: the bookings and conversation store (Cosmos DB serverless or PostgreSQL), and questions 5, 6 and 7 below. The note in docs/research.md that the store was "later revised to PostgreSQL in spec" refers to a spec that does not exist and has no standing; the note was removed in revision 9.
- **D12**: The repository is private under github.com/ranaidrees. Commits are authored as Rana Idrees with the account's GitHub noreply address.

### Decided after council review 02 (2026-10-03)
- **D13**: Amends D9 and answers question 1 of docs/council/02-intent-review.md. Sessions may still write to prod, but every prod write, through any tool, waits for the owner's yes after the cost and blast radius assessment. az and azd writes in dev keep the D9 low-risk rule and run without asking when it is met. The council's proposal (dev only, enforced by Azure RBAC) was not adopted.
- **D14**: Amends D10 and answers question 2 of review 02. Azure MCP write tools stay enabled, but every Azure MCP write, in any environment, waits for the owner's yes. This is a written rule in CLAUDE.md only; the owner chose not to add a Claude Code permission rule that would force a prompt.
- **D15**: The council has six members. An AI Engineer is added to cover agent design, retrieval, evaluation and runtime guardrails, the gap the smoke test exposed, and each member's brief now names the expertise it brings to this project and goal. The cost of a council run (about 1.6 million subagent tokens with five members) is accepted.
- **D16**: Repository visibility is under review (question 3 of review 02). The owner prefers private but will make the repository public if the GitHub plan restricts what the project needs. Facts: on GitHub Free, environments exist only on public repositories; on Free, Pro and Team, required reviewers and wait timers exist only on public repositories; protected branches on private repositories need GitHub Pro. Sources: https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/manage-environments and https://docs.github.com/en/get-started/learning-about-github/githubs-plans. Until the owner decides, the repository stays private and the spec must not assume a required-reviewer gate.
- **D17**: Two harness additions. Each session writes a journal entry in docs/journal/ from a template, and starts from the handoff in the previous entry. A /cleanup skill reports broken, stale, duplicated, unused or unnecessary files and changes nothing without the owner's approval; it runs before the final commit of each stage. No document is added outside the artifact chain, council reviews, journal and harness configuration unless the owner asks.

### Needed before later phases (council's proposed default in brackets)
5. Tenant provisioning through PR and approval, or admin action plus audit log? [PR and approval, via IaC]
6. Index per tenant or shared index with tenant filter? [Index per tenant]
7. Model calls via APIM or Foundry's native gateway connection? [APIM, after a spike]
8. Conversation store: see D11.
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
