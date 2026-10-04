# Council review: implementation plan (implementation-plan-phase-0-1.md)

Date: 2026-10-04. Artifact reviewed: docs/implementation-plan-phase-0-1.md, commit 7dfc3c8 (status "awaiting the owner's acceptance").
Method: 6 members reviewed independently, then each gave one rebuttal on the others' anonymised reviews; the chair synthesised and added nothing of its own.

## Members
| Member | Lens | First verdict | Final verdict |
|---|---|---|---|
| council-contrarian | Contrarian | Accept with changes | Accept with changes |
| council-ai-engineer | AI Engineer | Accept with changes | Accept with changes |
| council-hiring-manager | Hiring Manager | Accept with changes | Accept with changes |
| council-architect | Platform Architect | Accept with changes | Accept with changes |
| council-security | Security and Compliance | Accept with changes | Accept with changes |
| council-simplifier | Simplifier | Accept with changes | Accept with changes |

## Verdict
**Accept with changes.** All six members held that verdict through both rounds: the step structure holds and every defect is a plan-text correction before code, not a redesign (AI Engineer, Architect, Simplifier). Three defects would stop week 1: the eval judge on resource A reopens the model-path bypass the story rests on (all six), the pull request what-if job cannot log in so nothing merges (five), and the gateway has no telemetry sink so S1, the Workbook and the D71 reconciliation produce nothing (four). Contrarian and Architect verified the PyPI pins, APIM v2 rates, cron timezone, AVM versions, projects API version, purge actions, `sha_pinning_required` and the immutable-releases endpoint.

## Debates and resolutions
### 1. The eval judge on resource A reopens the bypass
- **All six**: step 0.15 deploys the judge on resource A during every gate run (1.9), giving the candidate implicit inference access outside the gateway; spec 3.2 and S1 test 3 become "closed except when tested". Contrarian: the pipeline identity would also need deployment write and delete on A plus Foundry User at account scope, absent from spec 4 and plan 9.3.
- **AI Engineer** (rebuttal, unanswered): the gateway route sends judge tokens through `llm.xml` (plan 1.5); without its own registry row and counter key the judge is refused or consumes the tenant's quota and 1.9 reports failed-to-run. Architect's review made the same point. The route also requires Chat Completions on the connected deployment.
- **Resolution**: S6 tests the admin-connected model through the gateway first (preview, "might not be available in all regions"; pinned and listed as a dependency, Security); judge on resource A only as the S6 fallback, with any window recorded in ADR 0006 and spec 3.2. Fallback acceptability is owner question 1.

### 2. Pull request what-if cannot run
- **Contrarian, Hiring Manager, Architect, Security, Simplifier**: step 0.7 restricts `dev` to `main`; step 0.8 runs `azure/login` and what-if in `dev` on `pull_request` (`refs/pull/N/merge`). GitHub fails the job, so the required `pr-checks` never passes; loosening `dev` hands branches the pipeline identity before review.
- **Hiring Manager, Architect, Security**: a read-only identity federated to subject `repo:...:pull_request` (Architect accepted this over a `pr` environment); keeps what-if on the PR as D69 and spec lines 814, 836 require.
- **Simplifier, Contrarian**: Reader cannot run what-if; it has the same permission requirements as deployment (Simplifier) and `Microsoft.Resources/deployments/whatIf/action` is a separate action needing a custom role (Contrarian). Simplifier: anything sufficient gives PR triggers deploy-grade rights against spec line 889; run `az bicep build` or `bicep snapshot` without a login, move what-if to the candidate job, correct spec lines 814, 837, 990 in 0.18. Security accepts this; Contrarian, Hiring Manager and Architect dispute it because REVIEW.md pass 2 (line 816) relies on the pre-merge diff.
- **Resolution**: defect agreed; remedy not. Raised to owner as question 2.

### 3. No telemetry sink on the gateway; reconciliation counts instead of joins
- **Contrarian, Hiring Manager, Architect, Security**: `gateway.bicep` (0.5, 9.1) has no Application Insights logger or diagnostic with `metrics: true`; the custom-metrics setting is documented as a portal step. S1 criterion 1, both metric policies, the Workbook and `maf check reconcile` produce nothing. Step 1.5 compares audit rows with a request count, inflated by `initialize` and `tools/list`.
- **Contrarian** (accepted by Architect, Security, Simplifier): a system-assigned gateway identity is a new principal every morning, with daily re-grants, propagation delay and orphaned assignments.
- **Resolution**: converged. Add logger, diagnostic at 100 per cent sampling and Monitoring Metrics Publisher to 0.5 (AVM `loggers`, `serviceDiagnostics`); set the custom-metrics flag once in the bootstrap, S1 test 4 checking a non-portal route; frontend response payload bytes 0 at global scope so MCP streaming survives (Architect); gateway stamps a request id that salon-mcp writes to the audit row; reconcile by join filtered to `tools/call`. Gateway identity becomes user-assigned `id-maf-gateway` in `rg-maf-persist` (owner question 3).

### 4. The eval gate neither runs nor measures what it claims
- **Contrarian, Hiring Manager, AI Engineer** (accepted by Architect, Simplifier): S6 needs at least 14 judged runs in a day at 161,000 tokens each (spec 8) against a 150,000 daily quota; 1.9 treats the 403 as failed-to-run, so the spike stalls on run one.
- **AI Engineer** (accepted by Contrarian, Hiring Manager, Security): thresholds derive from the echo graph plus FAQ path, so out-of-scope, book and cancel noise is unmeasured; judge model and version unnamed. Appendix F's `expected_intent`, `allowed_citations` and "no tool call" are read by no evaluator in 9.4; groundedness needs `context`, absent from F.4. O18 and O50 are jailbreak-shaped; a guardrail 400 is neither pass nor failed-to-run.
- **Resolution**: converged. Raise the quota parameter for the spike day only and restore it (figure differs; owner question 4). S6 proves plumbing only; thresholds derive from five runs on the bootstrap release (D38) before any promotion is gated; pin the judge. `maf gate decide` checks intent and tool calls deterministically from the candidate's spans and parses citations against `allowed_citations`; the export expands `passages` into `context`. Move O18 and O50 to `test_guardrail.py`; a guardrail 400 inside a gate run is failed-to-run.

### 5. D58 against D46, and the indirect-attack shield claim
- **AI Engineer** (finding accepted by Contrarian, Hiring Manager, Architect): step 1.6 sets content recording off; trace evaluators return `score=None` without `gen_ai.input.messages`, present only "when content recording is enabled". Proposes recording on for the pinned eval session only.
- **Security**: `enable_content_recording` is a tracer constructor parameter, so the switch lives on the version, and the candidate version is the one promoted (1.4); evaluate on a non-promoted eval version or drop D58 for Phase 1. Simplifier: a cut or deferral.
- **AI Engineer** (accepted by Contrarian, Architect, Security): the shield in resource A's policy (1.7) screens agent prompts and responses; passages travel to resource B, whose deployments have no named policy.
- **Resolution**: shield claim corrected; the Phase 1 control is structural plus the poisoned-passage test (owner question 6). Content recording: no resolution; owner question 5.

### 6. Automation identities exceed the spec's role table
- **Security** (confirmed by Hiring Manager, Architect, Simplifier): plan 9.2 and 9.3 give the pipeline Foundry Project Manager; spec line 398 grants `agents/write` only. 9.3 grants `searchServices/delete` across `rg-maf-dev` while 0.10's proof expects it refused. Contrarian: connection-write by Project Manager is unverified.
- **Architect, Security** (accepted by Contrarian, Hiring Manager, Simplifier): the teardown identity cannot read Log Analytics, the audit table, release records or named values, which three of four nightly checks need.
- **Simplifier**: purge requires "Contributor access to the API Management instance", so `maf-gateway-teardown` fails its own proof; use API Management Service Contributor at `rg-maf-dev` and drop `maf-agent-reader` by running the served-image check under the pipeline identity. For the pipeline, Foundry User at project scope suffices, no fifth custom role; Security proposes a custom role (`agents/read`, `agents/write` at project scope).
- **AI Engineer, Hiring Manager, Architect, Security**: a detective check must not run under the identity it polices; policy-edit rights let a holder authenticate as the gateway's identity, reaching resource B (Architect: keep the fallback scoped to one gateway resource, line 226). Security (rebuttal): move the purge into `maf up` under the pipeline identity; keep the nightly identity delete-only.
- **Resolution**: converged on removing Project Manager and `searchServices/delete` (S4 fallback only, scoped to that resource) and adding the read roles. Pipeline role shape and purge design unresolved; owner questions 7 and 8.

### 7. MCP pass-through entity needs a preview API version
- **Architect, Simplifier** (accepted by all others): steps 0.5 and 9.1 fall back to a raw resource at `2024-05-01`; MCP servers are `service/apis@2025-09-01-preview` with `type: 'mcp'` and `mcpProperties`; Learn publishes a complete passthrough template.
- **Resolution**: converged. Raw child of the AVM service at that version, `transportType: 'streamable'`, endpoint `/mcp`, `subscriptionRequired: false`; add the version to spec section 7 in 0.18 with `az rest` as fallback; list the preview dependency (intent section 9).

### 8. Schedule, step order, supply chain and items raised late
- **Simplifier** (slack confirmed by Contrarian, Hiring Manager; 0.17 by Contrarian, AI Engineer, Architect, Security): weeks sum to 5.0 and 5.5 days; step 0.17 wires mutmut against modules written in 1.1 and 1.3; spec line 1270 places the gate in Phase 1. Fold into 1.3. **Hiring Manager** disputes: intent line 124 makes them Phase 0 deliverables; wire on a planted stub, measure in 1.3. Owner question 9.
- **Security** (accepted in part by Contrarian, AI Engineer, Architect, Simplifier): pin the base image by digest, add `docker` to Dependabot, disable `claude-plugins-official` auto-update (on by default, undoing D64). Contrarian: `uv` is listed, so that part fails. Simplifier: defer the image scanner, not in the intent's exit criteria. Owner question 10.
- **Hiring Manager** (accepted by Security): step 1.13's README gains a "follow one release" section for the auditor's walk; no new document (D31). Converged.
- Rebuttal-only, unanswered: **Hiring Manager**: step 0.10 runs `maf down`, which purges the gateway, then `maf check registry` against its named values; it can never pass (spec line 924 same order). **Architect**: line 181 prices the gateway at nine hours a day but the only teardown is 22:00; a 09:00 `up` gives about £42 a month, above the ceiling (spec line 1182). **Simplifier**: the `maf` CLI (ten subcommands, 75 mentions) has no estimate line. Owner question 11.

## Dissent
None.

## Owner questions
1. If the admin-connected judge route fails in S6, is "bypass detected" acceptable for the model path during gate runs? [Yes, with the usage-comparison check] (raised by Contrarian)
2. Pull request checks: a separate read-only identity federated to the `pull_request` subject with a custom role carrying `whatIf/action` (Hiring Manager, Architect, Security, Contrarian), or login-free checks with what-if in the candidate job and spec lines 814, 837, 990 corrected (Simplifier, Security)? [No shared default]
3. Move the gateway to a user-assigned identity in `rg-maf-persist`, as a spec change in step 0.18? [Yes] (raised by Contrarian)
4. Raise the daily quota for the S6 spike day only? [Contrarian: 3,000,000, about £3.40; Hiring Manager: 2,000,000, restored after]
5. Content recording for D58: on for the pinned eval session only, revising D46 (AI Engineer) [Yes, eval session only], or a non-promoted eval version or drop D58 for Phase 1 (Security)?
6. Retrieved-content injection: structural control plus the poisoned-passage test as the Phase 1 answer, shield claim corrected? [Yes] (raised by AI Engineer)
7. Pipeline identity: custom role with `agents/read` and `agents/write` at project scope (Security) or built-in Foundry User at project scope (Simplifier)?
8. Teardown identity: API Management Service Contributor at `rg-maf-dev` (Simplifier) [Yes], the fallback scoped to one gateway resource (Architect), or purge in `maf up` under the pipeline identity with the nightly identity delete-only (Security)?
9. Mutation and contracts gate: Phase 1 week 1 (Simplifier) [Yes], or wired in Phase 0 on a planted stub and measured in 1.3 (Hiring Manager)?
10. Image scanning and digest pins in Phase 0 step 0.8 (Security) [Phase 0; CI minutes only], or scanner deferred (Simplifier)?
11. Rebuttal-only findings to confirm: judge registry row and counter key (AI Engineer); step 0.10 order (Hiring Manager); gateway hours against the cost ceiling, evening schedule or session-close `maf down` rule (Architect); an estimate line for the `maf` CLI (Simplifier).

## Evidence
- https://learn.microsoft.com/en-us/azure/foundry/observability/how-to/evaluate-admin-connected-models (all six: admin-connected judge, preview)
- https://learn.microsoft.com/en-us/azure/foundry/observability/how-to/evaluation-permissions (Contrarian: Foundry User at account scope)
- https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/ai-gateway (Contrarian, Architect, Simplifier: Bicep connection route)
- https://github.com/microsoft-foundry/foundry-samples/blob/main/infrastructure/infrastructure-setup-bicep/01-connections/apim-and-modelgateway-integration-guide.md (Simplifier: connection by Bicep)
- https://learn.microsoft.com/azure/foundry/concepts/evaluation-evaluators/agent-evaluators (AI Engineer: evaluators read only `query` and `response`)
- https://learn.microsoft.com/azure/foundry/concepts/evaluation-evaluators/rag-evaluators (AI Engineer: groundedness needs `context`)
- https://learn.microsoft.com/azure/foundry/observability/how-to/cloud-evaluation-targets (AI Engineer: `{{sample.output_items}}`)
- https://learn.microsoft.com/azure/foundry/observability/how-to/cloud-evaluation-deployed-interactions (AI Engineer, Contrarian: `score=None` without content)
- https://learn.microsoft.com/azure/foundry/how-to/develop/langchain-traces (AI Engineer, Contrarian, Hiring Manager, Architect, Security: content recording)
- https://raw.githubusercontent.com/microsoft/ai-agent-evals/main/README.md (AI Engineer: field pass-through unverified)
- https://learn.microsoft.com/azure/foundry/agents/how-to/add-hosted-agent-guardrails (AI Engineer, Contrarian, Architect: shield scope, 400 before the agent runs)
- https://learn.microsoft.com/azure/foundry/openai/concepts/content-filter-prompt-shields (AI Engineer: document attacks)
- https://learn.microsoft.com/azure/ai-foundry/openai/concepts/content-filter-document-embedding (AI Engineer: `<documents>` tagging)
- https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments (Hiring Manager, Architect, Simplifier, AI Engineer: branch rules, `refs/pull/*/merge`)
- https://github.blog/changelog/2021-02-17-github-actions-limit-which-branches-can-deploy-to-an-environment/ (Contrarian: job fails on mismatch)
- https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows (Architect: `GITHUB_REF` for PRs)
- https://docs.github.com/en/rest/deployments/environments (Hiring Manager: environment API)
- https://docs.github.com/en/actions/reference/security/oidc (Security, Hiring Manager: `pull_request` subject)
- https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/deploy-what-if (Architect, Simplifier: what-if permissions)
- https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles/general (Contrarian: Reader is `*/read`)
- https://learn.microsoft.com/en-us/azure/role-based-access-control/permissions/management-and-governance (Contrarian: `whatIf/action`)
- https://learn.microsoft.com/en-us/azure/api-management/emit-metric-policy (Contrarian, Hiring Manager, AI Engineer: logger prerequisites)
- https://learn.microsoft.com/en-us/azure/api-management/llm-emit-token-metric-policy (Architect: prerequisites)
- https://learn.microsoft.com/en-us/azure/api-management/api-management-howto-app-insights (Contrarian, Hiring Manager, Security: integration, "not intended to be an audit system")
- https://learn.microsoft.com/azure/api-management/monitor-mcp-servers (Security, AI Engineer: `tools/call` filter)
- https://raw.githubusercontent.com/Azure/bicep-registry-modules/main/avm/res/api-management/service/README.md (Contrarian, Hiring Manager: `loggers`, `serviceDiagnostics`)
- https://learn.microsoft.com/en-us/azure/api-management/api-management-howto-use-managed-service-identity (Contrarian, Architect: user-assigned support; policy-edit warning)
- https://learn.microsoft.com/azure/api-management/manage-mcp-servers-rest-api (Architect, Simplifier, Contrarian, AI Engineer: `apis@2025-09-01-preview`)
- https://learn.microsoft.com/azure/api-management/expose-existing-mcp-server (Architect, Simplifier: passthrough template, payload bytes)
- https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/mcp-authentication (Architect: MCP auth)
- https://learn.microsoft.com/en-us/azure/api-management/soft-delete (Simplifier, Contrarian, Architect, Security: purge needs Contributor)
- https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles/integration (Simplifier: API Management Service Contributor)
- https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/hosted-agent-permissions (Security, Simplifier: Foundry User scope)
- https://learn.microsoft.com/azure/foundry/concepts/rbac-foundry (Security, Contrarian, Architect: Project Manager capabilities)
- https://docs.github.com/en/code-security/reference/supply-chain-security/supported-ecosystems-and-repositories (Security: Dependabot ecosystems)
- https://docs.github.com/en/code-security/dependabot/working-with-dependabot/dependabot-options-reference (Contrarian: `uv` listed)
- https://code.claude.com/docs/en/plugins/install (Security, Contrarian, AI Engineer, Architect, Simplifier: auto-update on by default)
- https://owasp.github.io/www-project-mcp-top-10/ (Security: MCP04)
- https://prices.azure.com/api/retail/prices?currencyCode='GBP'&$filter=serviceName eq 'API Management' and armRegionName eq 'ukwest' and contains(skuName, 'v2') (Contrarian: APIM v2 rates)
- https://mcr.microsoft.com/v2/bicep/avm/res/api-management/service/tags/list (Architect: AVM versions)
- https://learn.microsoft.com/en-us/azure/templates/microsoft.cognitiveservices/accounts/projects (Architect: projects API `2026-07-01`)
- https://docs.github.com/en/rest/actions/permissions (Architect: `sha_pinning_required`)
- https://raw.githubusercontent.com/actions/attest/main/README.md (Architect: attestation)
- M:\ClaudeCodeProjects\MultiTenantAgentFactory\docs\implementation-plan-phase-0-1.md and docs\spec-phase-0-1.md (Simplifier, Hiring Manager, Architect: line references)
