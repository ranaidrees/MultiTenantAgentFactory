# Council review: Stage 2 spec, Phases 0 and 1 (spec-phase-0-1.md)

Date: 2026-10-03. Artifact reviewed: docs/spec-phase-0-1.md, draft, commit e13557b.
Method: six members reviewed independently, then each gave one rebuttal on the others'
anonymised reviews; the chair synthesised and added nothing of its own.

## Members
| Member | Lens | First verdict | Final verdict |
|---|---|---|---|
| Hiring Manager | Interview impact | Accept with changes | Accept with changes |
| Security and Compliance | Risk team acceptance | Accept with changes | Accept with changes |
| Simplifier | Smallest proof | Accept with changes | Accept with changes |
| Platform Architect | Buildable on Azure | Accept with changes | Accept with changes |
| Contrarian | Failures and assumptions | Accept with changes | Accept with changes |
| AI Engineer | Agent design, retrieval, evaluation and runtime guardrails | Accept with changes | Accept with changes |

## Verdict
**Accept with changes.** All six members held this verdict in both rounds. Nightly teardown, the approval gate, the eval gate and the token quota do not yet support the spec's claims. The Contrarian says the changes block acceptance.

## Debates and resolutions

### 1. Nightly teardown breaks the release chain
- **Hiring Manager, Platform Architect, Contrarian, AI Engineer**: after `down`, `up` serves an ungated version and 5.9's rollback target is gone; `up` should restore the last approved digest.
- **Simplifier**: only the gateway bills hourly (section 8); remove it alone each night.
- **Resolution**: all six back gateway-only teardown, which reopens D28 (question 1). Digest restore on full rebuilds: no resolution (question 2).

### 2. Approval is not a privilege boundary
- **Security and Compliance**: one identity serves `dev` and `dev-promote`, and `agents/write` both creates a version and moves the selector, so any `dev` job can promote. Sessions hold the approver's credential; salon-mcp deploys before approval.
- **Platform Architect**: add a role table; state that GitHub, not Azure, enforces the gate.
- **Simplifier, Contrarian**: 5.9 defers salon-mcp revisions to Phase 2.
- **Resolution**: clarify with the role table and that statement. Other remedies: questions 3 and 4.

### 3. The eval gate cannot gate
- **Contrarian, AI Engineer**: the Action sends single `query` rows, so book and cancel stop at the interrupt, and 30 rows cannot detect a realistic regression. Diagnosis undisputed.
- **Simplifier**: keep one harness (D35), claim gross regressions only, drop cost, latency and the eval view now.
- **Hiring Manager, Platform Architect, AI Engineer**: intent section 6 requires those; use spans.
- **Resolution**: no resolution on the remedy; questions 5 and 6.

### 4. The monthly quota resets at each purge
- **Hiring Manager, Platform Architect, Contrarian**: counters live in the purged gateway (survival unverified), and the 403 demonstration needs two million tokens; use a daily quota and a low test quota.
- **Resolution**: the other three agreed; changing the owner's monthly figure is question 7.

### 5. Evidence durability
- **Hiring Manager**: section 9 evidence is private, local or expiring; publish it as a GitHub Release.
- **Security and Compliance**: a Release stays editable; use an immutable blob container.
- **Simplifier**: beyond 9.2; section 11 accepts this exposure (D20).
- **Resolution**: no resolution; question 8.

### 6. Retrieval, grounding and guardrail depth
- **Simplifier**: keyword search from the start, deleting hop 7 (cut 2 in section 10).
- **AI Engineer**: an unmeasured quality cut. Grounding is asserted, not enforced; the guardrail is shown to exist, not to work.
- **Hiring Manager, Contrarian, Simplifier**: D31 asks for one negative test, so the attack set is scope growth; the last two say the same of recall@k.
- **Resolution**: no resolution; questions 9 and 10.

### 7. Sequencing and contradictions inside the spec
- **Simplifier, Platform Architect, Contrarian**: path B is portal-only, so fails go criterion 4 on paper; S1 needs the token S2 finds.
- **Platform Architect**: "nine keys on every span" (9.2) contradicts 5.6 (also Simplifier); one Bicep module cannot do data-plane steps; the bootstrap omits app registrations.
- **Hiring Manager**: "proven by tests" oversells the tenant boundary.
- **Resolution**: unopposed: run S2 before S1; scope the nine keys to agent spans; split the module into Bicep and script steps; name the registrations; reword the tenant claim unless question 12 is approved. Path B: question 11.

## Dissent
None. All six final verdicts match the council's; no contested position was decided against a member.

## Owner questions
1. Reopen D28 so nightly teardown removes only the gateway? [Yes] (Simplifier)
2. On full rebuilds, may `up` restore only the last approved digest, recorded as a restore? [Yes] (Hiring Manager and three others; Simplifier opposes)
3. May sessions hold a GitHub credential that can approve promotion? [No: browser only] (Security and Compliance)
4. Which gate additions enter Phase 1: environments restricted to `main`, a served-version check, a teardown identity with a purge-only custom role [Yes], a zero-traffic salon-mcp revision, full-SHA pins (Security and Compliance, Platform Architect); a name suffix per rebuild (Contrarian; opposed by Hiring Manager, Platform Architect, Simplifier)?
5. Gate writes with model-in-the-loop pytest beside the Action? [Yes] (AI Engineer)
6. Thresholds from repeat runs (Hiring Manager, AI Engineer) or gross regressions only (Simplifier)? Keep cost, latency and the eval view?
7. Is a daily enforced quota with a reported monthly figure acceptable? [Yes] (Platform Architect, Contrarian)
8. Evidence store: GitHub Release [Yes] (Hiring Manager) or immutable blob container [add it] (Security and Compliance)?
9. Keyword-only retrieval from the start? [Yes] (Simplifier)
10. Which additions enter Phase 1: validator node, recall@k, unanswerable questions, attack set, poisoned-passage test, model version pin, content recording in eval sessions? (AI Engineer; Security and Compliance opposes recording)
11. Amend D23 so path B is a desk check, built only if path A fails? [Yes] (Simplifier, Platform Architect, Contrarian) Add a Phase 0 cut order? (Contrarian)
12. May Phase 1 add a local two-tenant pytest? [Yes] (Hiring Manager)
13. Raised in rebuttal, undebated; which must the spec address? No release record or baseline on a clean clone (Hiring Manager); UK South lacks safety evaluators (Security and Compliance); Langfuse exporter missing from the cut order (Simplifier); registry held twice, no source of truth (Platform Architect); model quota unverified (Contrarian); eval runs share partition, counter and drifting dates (AI Engineer).

## Evidence
- https://learn.microsoft.com/azure/foundry/agents/how-to/manage-hosted-agent (cited by Hiring Manager, Platform Architect and Contrarian for deletion not being rollback)
- https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/hosted-agent-permissions (cited by Security and Compliance and Platform Architect for `agents/write` covering version and selector)
- https://learn.microsoft.com/en-us/azure/api-management/llm-token-limit-policy (cited by Hiring Manager, Platform Architect and Contrarian for per-gateway counters)
- https://learn.microsoft.com/azure/foundry/how-to/evaluation-github-action (cited by Contrarian and AI Engineer for single `query` rows)
- https://learn.microsoft.com/en-us/azure/foundry/configuration/enable-ai-api-management-gateway-portal (cited by Simplifier, Platform Architect and Contrarian for path B being portal-only)
- https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/immutable-releases (cited by Security and Compliance for Releases staying editable)
- https://learn.microsoft.com/azure/foundry/concepts/evaluation-regions-limits-virtual-network (cited by Security and Compliance for UK South and safety evaluators)
