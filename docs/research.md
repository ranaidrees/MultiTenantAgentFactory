# Research: AgentOps on Azure (reference implementations, patterns and build plan)

Date: 2026-10-03. Purpose: evidence base for docs/intent.md. Star counts and dates are as captured in October 2026; effort estimates are my own, not measured figures. Where this note and docs/intent.md section 14 disagree, the intent is right.

Summary: the idea is partly done, not novel in its parts but uncommon as a whole. Every individual component (LangGraph on Foundry hosted agents, MCP on Container Apps, APIM as AI gateway, Langfuse on Azure, Foundry eval gates in GitHub Actions) has an official sample or template, but no public repo found combines them into one governed, multi-tenant platform with a spec-driven Agent Factory and a lifecycle dashboard. That integration, plus the regulated-industry controls, is the differentiator.

## TL;DR

- **Partly done.** Microsoft ships separate samples for each layer (foundry-samples hosted agents, Azure-Samples/AI-Gateway, python-mcp-demos, microsoft/ai-agent-evals), and Google's agent-starter-pack proves the "template plus CI/CD plus eval" factory idea on GCP. Nobody found publishes the integrated Azure version with tenant isolation, HITL booking and a generator that opens PRs.
- **The stack is mostly GA, with important preview edges.** Foundry hosted agents went GA on 9 July 2026 (per Microsoft's Foundry blog), but the LangGraph adapter `from_langgraph` (azure.ai.agentserver.langgraph) has been superseded in official samples by `langchain_azure_ai.agents.hosting`; the Foundry eval GitHub Action is still `v3-beta`; AI Search knowledge base MCP endpoints use a preview API version; and the official Azure Samples Langfuse template is archived and pinned to Langfuse v2.
- **Biggest early decisions:** build in UK South (listed by Microsoft Learn as a hosted agent region, confirm in portal); keep your own MCP server rather than relying on APIM's REST-to-MCP feature for RAG; run Langfuse cheaply (single VM with Docker Compose) rather than the production Terraform module, which provisions AKS and HA databases.

## 1. Done, partly done, or novel?

| Area | Status | Evidence |
|---|---|---|
| LangGraph on Foundry hosted agents | Done (official samples) | foundry-samples and langchain-azure hosting samples, including a HITL interrupt sample |
| MCP servers on Container Apps | Done (azd templates) | Azure-Samples/python-mcp-demos |
| APIM as AI gateway with token budgets | Done (labs) | Azure-Samples/AI-Gateway, 30+ labs with Bicep and policies |
| Langfuse on Azure | Partly done, aging | langfuse-on-azure archived (v2); official Terraform module is production-scale AKS |
| Offline eval gate in CI | Done, preview | microsoft/ai-agent-evals v3-beta |
| RAG exposed as MCP | Done (first-party feature) | Azure AI Search knowledge bases expose an MCP endpoint |
| Booking agent with deterministic validation and HITL | Hobby examples only | Small community repos; no reference-grade sample |
| Agent Factory from spec to PR | Partly done elsewhere | Google agent-starter-pack (GCP), Backstage golden-path templates |
| Integrated, multi-tenant, regulated AgentOps platform on Azure | Not found | This project's contribution |

What interviewers will value is not that each service was used, but the seams: identity propagation from gateway to agent to MCP tool, tenant_id enforced in code rather than prompts, and evidence that a release passed a gate.

## 2. Top 5 repos and templates

| Rank | Repo | Owner | Stars | Licence | Last activity | Maturity | Why |
|---|---|---|---|---|---|---|---|
| 1 | https://github.com/microsoft-foundry/foundry-samples (samples/python/hosted-agents/langgraph) | Microsoft | 453 | MIT | Updated 1 Oct 2026 | Official template | Canonical way to host LangGraph on Foundry; uses langchain_azure_ai.agents.hosting and azd |
| 2 | https://github.com/Azure-Samples/AI-Gateway | Microsoft | about 986 | MIT | Not confirmed | Lab / demo | Bicep and policy XML for token limits, token metrics, MCP governance, Foundry model gateway |
| 3 | https://github.com/langchain-ai/langchain-azure (samples/hosting/langgraph-hosted-agents) | LangChain | 146 | MIT | Package 1.2.9, 24 Aug 2026 | Official template | Hosting samples including MCP tools, Foundry Toolbox, App Insights observability and HITL |
| 4 | https://github.com/Azure-Samples/python-mcp-demos | Microsoft | 182 | Not confirmed | Not found | Template | FastMCP on Container Apps with azd plus OAuth (Keycloak or Entra); reused for its azd and Container Apps layout only, since its Entra sample is a user sign-in proxy and it pins FastMCP 3 (intent D61) |
| 5 | https://github.com/microsoft/ai-agent-evals | Microsoft | about 89 | Not confirmed | Not found | Preview (v3-beta) | GitHub Action that runs Foundry evaluators against a dataset; the deployment gate |

Honourable mentions:
- **GoogleCloudPlatform/agent-starter-pack** (about 6.4k stars, Apache-2.0): best public model of an agent factory (templates, Terraform, CI/CD, eval). In maintenance mode; learn the structure, do not fork (GCP-specific).
- **langfuse/langfuse-terraform-azure** (about 36 stars, MIT, updated 28 Sep 2026): official, deploys Langfuse v4 on AKS with ClickHouse, Postgres HA, Managed Redis, App Gateway, Key Vault. Right for production, too costly for a personal subscription.
- **Azure-Samples/langfuse-on-azure** (59 stars, MIT): archived 29 July 2025, pins Langfuse v2. Do not use.
- **DarkDragonEl/golden-path-agent-template**: community repo with the same shape (LangGraph agent, MCP tool contract, approval gate, eval harness as CI gate, Backstage templates). Small and unproven; useful cross-check.
- **MSFT-Innovation-Hub-India/LangGraph-Foundry-HostedAgent-TravelAgent**: uses the older from_langgraph adapter; useful background only.

Estimated effort saved by reusing ranks 1 to 5: roughly 1.5 to 2.5 weeks of infrastructure and plumbing.

## 3. Recommended reference stack

| Layer | Recommendation | Status | Notes |
|---|---|---|---|
| Agent runtime | Foundry hosted agents, Responses protocol | GA (9 July 2026) | Per-session isolated sandboxes, dedicated Entra agent identity, idle timeout 2 to 60 minutes. UK South and UK West listed as regions (confirm). |
| LangGraph adapter | langchain_azure_ai.agents.hosting (ResponsesHostServer) | Current | Prefer over azure.ai.agentserver.langgraph.from_langgraph |
| HITL | LangGraph interrupt() plus a durable checkpointer | Stable | Microsoft says use a durable checkpointer in production |
| MCP servers | FastMCP on Azure Container Apps, Streamable HTTP | GA platform | Container Apps with internal ingress for private MCP endpoints |
| AI gateway | APIM Basic v2 or Standard v2 with llm-token-limit and llm-emit-token-metric | Policies GA | llm-token-limit not on Consumption tier; counter key can be tenant plus agent; rate limit returns 429, quota 403 |
| MCP governance | APIM in front of external MCP servers | Check status | Tools and resources supported, not prompts; MCP 2025-06-18 or later; v2 support announced as preview |
| RAG | Azure AI Search index called from the MCP server's search tool | GA | Knowledge base MCP endpoint is a preview API alternative |
| Config and bookings | Table Storage (intent D26); Cosmos DB serverless was the alternative | GA | Conditional writes or unique keys for double-booking protection |
| Observability | OpenTelemetry once, exported to App Insights and Langfuse OTLP endpoint | GA | One collector with two exporters |
| Langfuse hosting | Single VM with Docker Compose, or Langfuse Cloud free tier | OSS | Compose lacks HA and backups; acceptable for a demo if stated |
| Offline eval | microsoft/ai-agent-evals as the judged harness, with scripted tests for the write intents; the Foundry project evaluation API, which the Action wraps, as the fallback (intent D35, D44, D56) | Action in preview | Foundry evaluators for groundedness, intent resolution, task adherence |
| Online eval | Foundry continuous evaluation; Langfuse LLM-as-judge | GA | |
| IaC and CD | azd plus Bicep, GitHub Actions with OIDC, environments with required reviewers | GA | |
| Security | Managed identity, Key Vault, Content Safety Prompt Shields, Entra auth on MCP | GA | Microsoft's pages disagree on whether a hosted agent's endpoint can be made private (spec 2.5); Phase 1 uses no private networking |

## 4. RAG exposed as an MCP tool

**Established?** Yes. Every Azure AI Search knowledge base exposes an MCP endpoint, and Foundry IQ connects agents to it through an MCP tool connection using the project managed identity. Community MCP servers wrapping AI Search also exist.

| Concern | RAG behind MCP | Retriever inside graph |
|---|---|---|
| Reuse across agents | Strong; Factory agents get search for free | Each agent reimplements or imports |
| Tenant isolation | Enforced once, server-side, from caller identity | Enforced in every agent |
| Latency | Extra network hop plus JSON-RPC | Lowest |
| Observability | Needs trace context propagation | Single trace by default |
| Versioning | Independent of agents; risk of silent change | Locked to agent release |
| Citations | Must return structured source metadata | Native |

Recommendation: keep search behind the MCP server; propagate W3C trace context, pin tool schema versions in evals, and derive tenant_id from the authenticated caller, never from a model-supplied argument alone.

**MCP security guidance for regulated environments**
- The MCP specification's security best practices name confused deputy, token passthrough and session hijacking as core risks; servers must not accept tokens not issued for them.
- OWASP MCP Top 10 (beta): token mismanagement, scope creep, tool poisoning, supply chain, command injection, prompt injection via context, insufficient auth, lack of audit, shadow MCP servers, context over-sharing.
- OWASP MCP Security Cheat Sheet: pin tool definitions, least privilege per server and tool, human approval for destructive calls, central logging of all tool invocations.
- Practical controls: Entra tokens with audience validation; separate read and write scopes; writes only after confirmation; per-tenant rate limits; every call logged with tenant, agent identity, tool, argument hash and outcome.

## 5. Booking and transactional agents

1. LLM proposes, code disposes: the model extracts intent and slots; deterministic validators check hours, duration and conflicts.
2. Confirmation interrupt before any write, showing the normalised booking; needs a durable checkpointer.
3. Idempotency key derived from durable workflow state, checked server-side before writing.
4. Atomic conflict check at the store, because availability read earlier may be stale.
5. Compensation, not rollback, when an external write fails after an internal one.
6. Append-only audit log of proposal, confirmation, write result and any override.

Examples are thin: AnujRawat0608/Scheduling-agent (LangGraph, Postgres checkpointer, approval node, Google Calendar); KirtiJha/langgraph-interrupt-workflow-template (approve, edit, reject). Research: "Robust Agent Compensation" (arXiv:2605.03409) on compensation pairs for agent tools; treat as research, not practice.

## 6. Agent Factory patterns

- Google agent-starter-pack: one command generates an agent from a template with Terraform, CI/CD, evaluation and tracing.
- Backstage Software Templates: the platform engineering norm for golden paths; community work adds approval gates and lets agents open PRs.
- Safe deployment pattern: generation only opens a PR; branch protection and CODEOWNERS; CI runs lint, tests, policy checks and eval gate; prod via an environment with required reviewers.
- For this project: fill templates from a validated spec schema rather than free-form code generation; policy checks in CI (tenant_id present, writes behind interrupt, minimum eval cases); the Factory never holds deploy credentials.

## 7. Original build order (superseded by docs/intent.md phases)

Week 1 local slice; week 2 end to end in Azure; week 3 gateway and identity; week 4 observability and eval gate; week 5 Factory; week 6 dashboard and hardening.

## 8. Risks and open questions

- Region: hosted agents are listed for UK South, current models are Global Standard there, and the gateway is in UK West (intent D22, D34). Still to confirm in the portal.
- Adapter churn: pin versions; isolate the adapter behind one module.
- Preview dependencies: ai-agent-evals v3-beta, AI Search knowledge base MCP, APIM MCP on v2 tiers; keep fallbacks.
- Private networking: Microsoft's pages disagree on whether a hosted agent's endpoint can be made private (spec 2.5); Phase 1 uses none, relies on Entra auth and says so. Callers reach the agent through the gateway as a pass-through, gated by a Phase 0 spike; the path is governed, not closed (intent D80).
- Cost: only the gateway bills for existing, so it is removed nightly (intent D37), under a ceiling of £40 a month (D21). Rates and usage tables are in section 8 of docs/spec-phase-0-1.md. A Langfuse VM is a Phase 6 matter.
- Token limit accuracy: llm-token-limit counts prompt and completion tokens, counters are per gateway, concurrent requests can briefly exceed limits.
- Langfuse data residency: traces contain personal data; mask at the collector.
- Model path: a standalone API Management gateway, tested by a Phase 0 spike, with Foundry's AI Gateway as a desk check (intent D23, D47).
- Toolbox or direct calls to MCP servers: the documented path, a project connection with agentic-identity authentication and an audience, is tested first; the direct call from the graph is what spike S2 proves (intent D52, D60); the connection targets the gateway's MCP pass-through (intent D71).

## Sources
- Foundry hosted LangGraph agents: https://learn.microsoft.com/en-us/azure/foundry/how-to/develop/langchain-hosted-agents
- Foundry samples: https://github.com/microsoft-foundry/foundry-samples
- LangChain Azure: https://github.com/langchain-ai/langchain-azure
- AI Gateway labs: https://github.com/Azure-Samples/AI-Gateway
- APIM llm-token-limit: https://learn.microsoft.com/en-us/azure/api-management/llm-token-limit-policy
- Python MCP demos: https://github.com/Azure-Samples/python-mcp-demos
- Agent evals Action: https://github.com/microsoft/ai-agent-evals
- AI Search agentic retrieval and MCP: https://learn.microsoft.com/en-us/azure/search/agentic-retrieval-how-to-retrieve
- Foundry IQ: https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/foundry-iq-connect
- Foundry private networking: https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/virtual-networks
- Langfuse Terraform Azure: https://github.com/langfuse/langfuse-terraform-azure
- Langfuse OpenTelemetry: https://langfuse.com/integrations/native/opentelemetry
- OWASP MCP Top 10: https://owasp.github.io/www-project-mcp-top-10/
- OWASP MCP Security Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/MCP_Security_Cheat_Sheet.html
- Agent Starter Pack: https://github.com/GoogleCloudPlatform/agent-starter-pack
- Golden path agent template: https://github.com/DarkDragonEl/golden-path-agent-template
- Scheduling agent example: https://github.com/AnujRawat0608/Scheduling-agent
- LangChain human in the loop: https://docs.langchain.com/oss/python/langchain/human-in-the-loop
