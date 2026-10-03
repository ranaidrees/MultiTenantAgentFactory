# Council review: Stage 1 (intent.md)

Date: 2026-10-03. Artifact reviewed: intent.md revision 4. Result: revision 5.
Method: five members reviewed independently, then responded to each other; chair synthesised. Members argue from a fixed role.

## Members
| Member | Lens |
|---|---|
| Platform Architect | Is the design coherent, standard and buildable on Azure? |
| Security and Compliance | Would a regulated firm's risk team accept it? |
| Simplifier | What is the smallest thing that proves the point? Cut, defer, reuse. |
| Hiring Manager | What will impress an interview panel for an AI Principal Engineer? |
| Contrarian | What will fail, and what is being assumed without evidence? |

## Verdict
**Accept with changes.** The direction is sound and differentiated, but revision 4 was a feature list, not a deliverable. It is restructured into an MVP plus phases, with blocking questions for the owner.

## Debates and resolutions

### 1. Scope is too large for one engineer
- **Simplifier**: Revision 4 had five subsystems (runtime, factory, admin console, tenant portal, billing) in 9 weeks. If week 6 slips, nothing is demonstrable.
- **Hiring Manager**: The factory and multi-tenancy are the differentiators; a single-agent demo is commodity.
- **Contrarian**: Partial progress across five subsystems is worse than one finished slice.
- **Resolution**: Phase 1 MVP is a governed multi-tenant agent, complete and demonstrable alone. Multi-tenancy is in the MVP because it is cheap to include early and very expensive to retrofit. The Factory becomes Phase 2.

### 2. Should the Factory be in the MVP?
- **Hiring Manager**: Yes; it is the headline.
- **Simplifier**: A factory needs at least two working agents to extract templates from. Building templates before one agent works is guessing.
- **Platform Architect**: Agent Starter Pack and Backstage both template proven services, not hypothetical ones.
- **Resolution**: Phase 2. Phase 1 code is written template-ready (attribution keys, tenant enforcement and eval layout in one place) so extraction is mechanical. **Dissent (Hiring Manager)**: wants a thin factory demo in Phase 1. Raised to owner as question 1.

### 3. Langfuse in the MVP
- **Simplifier**: App Insights plus Foundry tracing cover traces, tokens and latency. Langfuse adds a VM and another login.
- **Contrarian**: Langfuse is the better eval and trace UI.
- **Platform Architect**: OpenTelemetry instrumentation means adding Langfuse later is configuration, not code.
- **Resolution**: Deferred to Phase 5.

### 4. Tenant provisioning by UI in the MVP
- **Security**: In regulated firms, environment changes go through change control.
- **Simplifier**: A script is enough for two tenants.
- **Resolution**: Phase 1 provisions by IaC script; the UI arrives in Phase 3 and calls the same IaC. Proposed default: provisioning also goes through PR and approval (question 5).

### 5. Isolation proof
- **Security**: Without automated cross-tenant tests, "multi-tenant" is a claim, not evidence.
- **Resolution**: Isolation tests are a Phase 1 deliverable and run in CI on every PR.

### 6. Dependency on Foundry hosted agents
- **Contrarian**: The hosting library already changed once. If it breaks during interviews, the demo dies.
- **Platform Architect**: Keep the graph free of hosting code; one thin adapter.
- **Resolution**: Agent graph independent of the adapter, runnable on Container Apps as fallback.

### 7. Billing and ROI timing
- **Simplifier**: Statements and ROI before any client exists is premature.
- **Hiring Manager**: Cost per tenant is a strong signal; full billing is not needed to show it.
- **Resolution**: Phase 1 shows cost and latency per tenant and agent on a Workbook. Ledger and statements move to Phase 5; ROI to Phase 4.

### 8. Reinventing the wheel
- **Contrarian**: The intent risked custom-building things that exist: council, spec templates, eval gate.
- **Resolution**: Reuse map added (intent section 11). Playbook artifact chain kept; Spec Kit's clarifications practice borrowed; council adapted from an existing Claude Code skill.

### 9. Unverified assumptions flagged
- UK South availability of hosted agents and the chosen models: confirm in the portal before Phase 0 ends.
- APIM v2 cost on a personal subscription: price it before Phase 1.
- Whether llm-token-limit works with Foundry hosted agent model calls routed through APIM: spike in Phase 0 (question 7).

## Questions raised to owner
See intent.md section 14. Questions 1 to 3 block spec.md.

## Owner decisions (2026-10-03)
- D1 Smaller MVP: one tenant first. **Recorded dissent (Security, Platform Architect)**: multi-tenancy is expensive to retrofit. **Mitigation adopted**: Phase 1 code is tenant-ready (tenant_id bound to agent identity, IaC module parameterised by tenant, override tests), and Phase 2 is dedicated to the second tenant and isolation tests.
- D2 Second agent domain: regulated document Q&A.
- D3 Console stack: Python FastAPI and React.
Intent accepted as revision 6.
