# Council review: intent (intent.md)

Date: 2026-10-03. Artifact reviewed: docs/intent.md, revision 7, commit eb59d95.
Method: five members reviewed independently, then each gave one rebuttal on the others'
anonymised reviews; the chair synthesised and added nothing of its own.

## Members
| Member | Lens | First verdict | Final verdict |
|---|---|---|---|
| Platform Architect | Buildable on Azure | Accept with changes | Accept with changes |
| Security and Compliance | Risk team acceptance | Accept with changes | Accept with changes |
| Simplifier | Smallest proof | Accept with changes | Accept with changes |
| Hiring Manager | Interview impact | Accept with changes | Accept with changes |
| Contrarian | Failures and assumptions | Accept with changes | Accept with changes |

## Verdict
**Accept with changes.** All five accepted with changes in both rounds. After rebuttal each ranked the same blockers first: prod has no enforced control (D9, D12), and the gateway claim is bypassable and cannot be created in UK South as written.

## Debates and resolutions
### 1. Prod has no enforced control (D9, D10, D12)
- **Contrarian, Hiring Manager, Security**: D9 contradicts D6; the only barrier to prod is the model's own estimate.
- **Hiring Manager, Security**: GitHub required reviewers are public-only on Free, Pro and Team (account plan unverified), so the section 6 item 8 gate cannot exist under D12; author and approver are one person.
- **Resolution**: All five converged on dev-only session writes enforced by Azure RBAC, not a hook or prose, and accept the D12 finding. Four want Azure MCP read-only; none objected. These reverse owner decisions: questions 1 to 3.

### 2. Gateway: bypass, region and cost
- **Contrarian, Platform Architect**: hosted agents get implicit model inferencing through the project endpoint, so the APIM budget binds only if agent code opts in.
- **Platform Architect**: no APIM v2 tier can currently be created in UK South; nightly teardown would lock the gateway out.
- **Simplifier**: one APIM instance for dev and prod, on cost; the other four object that policy edits would reach prod outside the release gate.
- **Resolution**: Converged: the intent's question 7 spike becomes a Phase 0 go or no-go with a negative test and a named fallback. Region and tier, tenancy unit and shared instance: questions 4 and 5.

### 3. Schedule and stop line
- **Contrarian**: two weeks contradicts docs/research.md section 7; re-baseline Phase 1 to four weeks.
- **Simplifier**: the budget should force cuts, not grow: drop the reschedule intent, gate on one eval harness, defer skills and verifier.
- **Resolution**: Both, with Hiring Manager, converged on a stop line at Phase 3: question 6. Phase 1 length: no resolution; raised to owner as question 7.

### 4. Caller identity
- **Platform Architect, Security**: nothing names who may call the agent, the principal at each hop, or how a customer is authorised to cancel a booking.
- **Resolution**: Hiring Manager and Contrarian concur: the intent states the Phase 1 caller model. Which caller: question 8.

### 5. Exit criteria and regulatory mapping
- **Hiring Manager**: exit criteria show controls exist, not that they work; add negative demos and a PRA SS1/23 control matrix.
- **Simplifier**: neither serves a section 6 exit criterion.
- **Contrarian**: accepts the demos, as does Security; defer the matrix to Phase 3, as SS1/23 excludes section 1's insurers and rating agencies.
- **Resolution**: Both add scope, so no resolution; raised to owner as question 9.

### 6. Langfuse and personal data (D7)
- **Simplifier**: nothing needs the export; D7's secrets break "one setup command".
- **Security**: "dev only, synthetic only" is unenforced.
- **Resolution**: All five converged: exporter off by default, outside the exit criteria. Gate it in IaC (Security, Platform Architect) or remove it (Simplifier): no resolution; question 10. Synthetic data only: question 11.

### 7. Contradictions and stale detail
- **Contrarian, Hiring Manager, Security**: section 13 claims isolation tests "from Phase 1"; section 6 item 5 says Phase 2.
- **Platform Architect**: gateway metrics take five low-cardinality dimensions, not nine keys; section 12 omits @azure/mcp and the hosting SDK.
- **Resolution**: Unopposed: correct section 13, limit metric dimensions to five with the rest span-only, complete section 12.

## Dissent
- No final verdict differs from the council's. Contrarian: it becomes Reject if the owner keeps D9 and D12 as written.
- Simplifier: one APIM instance for dev and prod, because two environments double the gateway cost; the other four disagree.

## Owner questions
1. May sessions write to prod? [No: dev only; prod by pipeline or logged break-glass] (raised by Contrarian, Hiring Manager, Security)
2. May Azure MCP start read-only, with writes through az and azd only? [Yes] (raised by Simplifier)
3. Public repository by Phase 1 exit, or a plan supporting required reviewers on private ones? [Public] (raised by Contrarian, Hiring Manager, Security)
4. Gateway: UK West Basic v2, or UK South classic Developer [UK West; price unverified]; one Foundry project per tenant [Yes]? (raised by Platform Architect)
5. May dev and prod share one APIM instance? [Yes; four members say no] (raised by Simplifier)
6. Where is the stop line? [End of Phase 3] (raised by Contrarian)
7. Phase 1: four weeks, or two weeks with cuts? (raised by Contrarian, Simplifier)
8. Phase 1 caller: Entra-authenticated test client only? [Yes; middle tier from Phase 4] (raised by Platform Architect)
9. Add negative demos, chat surface, per-phase scripts, control matrix, regulated Phase 1 skin [Yes] or runtime injection guardrail? (raised by Hiring Manager)
10. Langfuse export optional and outside exit criteria [Yes]; gated in IaC, or removed with Phase 6 self-hosting? (raised by Simplifier, Security)
11. Will any real person's data enter any environment before Phase 5? [No: synthetic only] (raised by Security)
12. Address rebuttal-only gaps: audit log and traces inside teardown scope; D9's £20 test not cumulative; AI Search tier unpriced? (raised by Security, Contrarian, Platform Architect)

## Evidence
- https://learn.microsoft.com/en-us/azure/developer/azure-mcp-server/tools/ (cited by Contrarian, Hiring Manager, Simplifier, Security for Azure MCP defaults)
- https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments (cited by Hiring Manager, Security for required reviewers)
- https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/hosted-agent-permissions (cited by Contrarian, Platform Architect for implicit inferencing)
- https://learn.microsoft.com/en-us/azure/api-management/api-management-region-availability (cited by Platform Architect for UK South v2 tiers)
