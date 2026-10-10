# Principal engineer review: the corrections that finish the plan before Build

Date: 2026-10-10. Artifacts reviewed: docs/intent.md revision 19, docs/spec-phase-0-1.md and docs/implementation-plan-phase-0-1.md at commit cc9637d, and then each document as it was revised for D97 to D101.
Method: one deep single-reviewer pass with a currency audit, not a council run (D70). It applies the findings that section 11 of docs/council/07-agentops-coverage-principal-review.md left open, and records the owner's decisions on them, D97 to D101 in docs/intent.md revision 20.
Reviewer lens: can an engineer who has never seen the conversation build Phase 0 from the plan alone, with every claim about the platform checked against the page it rests on.
Reads: CLAUDE.md; docs/journal/07-agentops-coverage-check.md, section 7; review 07, sections 10 and 11; the intent's section 14 from D89; the plan's sections 1 to 4, 9 and 11 and steps 0.4, 0.10, 1.1, 1.3, 1.7, 1.9 and 1.12; the spec's sections 2.5, 5.7, 5.8, 5.10 and 5.12.

How the pass was made. The session read the named sections itself. It opened Microsoft's pages on the container registry's role modes, which session 07 had not, and the page for a blocklist's items. It resolved every pin again against PyPI, the Azure Developer CLI's extension registry, the Bicep module registry and GitHub. It put four questions to the owner. Every quotation it relies on was matched to its page by script, by paragraph and over the whole of each revised document. Each revised document was read by an independent reviewer, who was given the decisions and not the reasoning, before the next was derived from it.

## 1. Verdict

**Review 07's finding 2 holds, and the fault is wider than the reviewer said.** The plan named roles from both of the registry's role modes and stated neither mode. In the mode Microsoft says will become the default, the pipeline's `AcrPush` would not have been honoured, and the owner's Owner role would not have let a session build an image. The other ten findings are faults of text, and every one stands. Nothing found is a redesign. Five decisions record what changes, D97 to D101.

## 2. Decisions

D97 to D101 in docs/intent.md revision 20. In one line each: the day figures are sizes and not boxes, a spike is settled by its criteria, and only the owner invokes a cut order (D97); a session may build images into the registry's `scratch/` repositories, and the registry uses the newer role mode with repository roles (D98); CLAUDE.md's architecture summary stands as changed in session 07 (D99); structured outputs are tried in spike S1 as a record (D100); `up` applies the environment's Bicep on every run, and the fixed replies are three (D101).

| Question put to the owner in this session | Answer |
|---|---|
| What replaces the time-boxes, now that the day figures do not bind | The go criteria decide. A spike that has not met them by the end of its session is reported, and the owner chooses the fallback or another session. The cut orders stay written and only the owner invokes them |
| Which role mode for the registry, and how far the build exception reaches | The newer mode. Session builds go to `scratch/` only, and the owner's write role is limited to that prefix by a condition |
| Where it is learned whether structured outputs work through the gateway | In spike S1, test 1, as a record |
| What may be installed on a workstation with 5 GB free on its system drive | The workspace's packages, with uv's cache and the environment both on the project's drive; the Azure Developer CLI extension's beta.19 in place of beta.18 |

## 3. The registry's role mode, checked against Microsoft's pages

Review 07 reported the claim as the reviewer's own and unchecked. It is now checked.

| Claim | What the page says | Page |
|---|---|---|
| The registry has two role modes, and the newer one is to be the default | "ABAC-enabled mode (`rbac-abac`) will become the default role assignment permissions mode for Azure Container Registry in the future." | Repository permissions |
| The legacy roles do nothing in the newer mode | "Legacy ACR roles such as `AcrPull`, `AcrPush`, and `AcrDelete` are not honored in ABAC-enabled registries." | Repository permissions |
| Owner and Contributor give no rights on images there | Such roles "grant only control plane permissions to create, update, and delete the registry itself, without granting data plane permissions to the repositories and images in the registry" | Repository permissions |
| A build in the registry has no access of its own | "For Quick Tasks, you must use the caller's identity to authenticate the Quick Task to the source registry." The flag is `--source-acr-auth-id [caller]` | Repository permissions |
| The caller needs the writer and the catalogue lister roles | "For quick builds and quick runs, the identity (caller) invoking the quick task must have the `Container Registry Repository Reader/Writer/Contributor` and `Container Registry Repository Catalog Lister` roles assigned to it in order to perform data plane operations." | Roles reference |
| Hosted agents work with either mode and prefer the repository roles | Container Registry Repository Reader is "preferred because it models the pull as a data action"; `AcrPull` is listed after it | Hosted agent permissions |
| A role can be limited to repositories under a prefix | The page gives a condition on the repository name with `StringStartsWithIgnoreCase` and a prefix that ends in a slash | Repository permissions |
| The Bicep module can set the mode and a condition | `roleAssignmentMode` allows `AbacRepositoryPermissions` and `LegacyRegistryPermissions`; a role assignment takes `condition` and `conditionVersion` | The module's README and source |

What follows for the plan. Step 0.3 already granted the project's identity the reader role of the newer mode, while section 9.3 gave the pipeline identity `AcrPush`, the push role of the legacy mode. Whichever mode the registry took, one of the two would have failed: the agent's image pull in the legacy mode, or the pipeline's push in the newer one. And a session's `az acr build` would have been refused in the newer mode whoever ran it, because no step gave any login a role on the registry's images. D98 fixes the mode and the roles.

One thing remains unverified until the first build: that a quick build made with the caller's identity respects a condition on the caller's writer role. If the build of step 0.4 is refused for that reason, the owner is asked before the condition is removed (D98).

## 4. Currency audit

Checked on 2026-10-10 against PyPI, the Azure Developer CLI's extension registry, the Bicep public module registry, GitHub and the Azure resource provider's own list. The re-resolved pins were locked together in a throwaway workspace on Python 3.11.7 before any document named them: 199 packages resolve.

| Item | Plan at cc9637d | Today | Handling |
|---|---|---|---|
| uv | 0.13.0 installed; 0.12.23 first named | 0.13.0, released 2026-10-09 | The pin |
| fastmcp | 4.0.10 | 4.1.0, released 2026-10-08. 4.0.11 carried security fixes. 4.1.0 changes one behaviour this design meets: "**HTTP sessions** now expire after the MCP SDK's 30-minute idle default instead of never." | Pinned at 4.1.0. The graph's tool client opens its session for each turn, so an idle session is never reused (plan step 1.3) |
| langgraph | 1.2.12 | 1.2.14 | Pinned |
| azure-ai-projects | 2.7.0 | 2.8.0 | Pinned |
| pydantic | 2.14.0 | 2.14.0 | No change |
| ruff | 0.16.10 | 0.17.0, released 2026-10-09 | Pinned. Not in session 07's list |
| azure-monitor-opentelemetry | 1.8.10 | 1.8.11 | Pinned. Not in session 07's list |
| opentelemetry-sdk | 1.45.0 | 1.45.1 | Pinned. Not in session 07's list |
| The other fifteen Python pins of plan section 3 | As named | Unchanged | None |
| Azure Developer CLI `azure.ai.agents` extension | 1.0.0-beta.18 | 1.0.0-beta.19, released 2026-10-08, needs azd 1.34.2 or later. Nothing in its notes touches the commands this plan uses | Pinned, and installed in step 0.1 |
| Azure Developer CLI | 1.35.0 | 1.35.1 is offered | Not installed; nothing needs it |
| `astral-sh/setup-uv` | v10.2.0 | v10.3.0, commit 1c37ad07a6a961277cf70c0d37d6f313000f5884 | Pinned in step 0.1 |
| `actions/checkout` | v7.0.1 | Unchanged | None |
| Azure Verified Module for API Management | 0.14.4 | 0.15.0, with one breaking change: "Updated the diagnostic implementation to avoid automatically enabling both metrics and logs when only one is specified." | Step 0.5 uses 0.15.0 and sets both explicitly, as it already meant to |
| The other Azure Verified Modules of plan section 9.1 | As named | Unchanged | None |
| Foundry's ARM API version | 2026-07-01 | The provider lists 2026-09-01 and 2026-09-15-preview for accounts and projects. Microsoft's template reference, the page that documents each property, lists 2026-07-01 as its newest stable version and 2026-07-15-preview as its newest preview | Stays at 2026-07-01, because the plan cites the template reference for every property it sets. Looked at again in step 0.3 with the Bicep linter's recent-versions rule |
| The blocklist item's resource | Cited to the blocklist's page | Its own page: properties `pattern` and `isRegex` | A key of its own; 2026-07-01 on both |

## 5. Sources opened in this session

Every quotation in this file was matched to its page by script on 2026-10-10.

- https://learn.microsoft.com/azure/container-registry/container-registry-rbac-abac-repository-permissions
- https://learn.microsoft.com/azure/container-registry/container-registry-rbac-built-in-roles-directory-reference
- https://learn.microsoft.com/azure/container-registry/container-registry-rbac-built-in-roles-overview
- https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles/containers
- https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/hosted-agent-permissions
- https://learn.microsoft.com/azure/foundry/agents/how-to/deploy-hosted-agent
- https://learn.microsoft.com/en-us/azure/templates/microsoft.cognitiveservices/accounts/raiblocklists
- https://learn.microsoft.com/en-us/azure/templates/microsoft.cognitiveservices/accounts/raiblocklists/raiblocklistitems
- https://learn.microsoft.com/en-us/azure/templates/microsoft.cognitiveservices/accounts
- https://learn.microsoft.com/azure/azure-resource-manager/bicep/overview
- https://learn.microsoft.com/azure/foundry/openai/how-to/structured-outputs
- https://raw.githubusercontent.com/Azure/bicep-registry-modules/main/avm/res/container-registry/registry/README.md
- https://raw.githubusercontent.com/Azure/bicep-registry-modules/main/avm/res/api-management/service/CHANGELOG.md
- https://raw.githubusercontent.com/Azure/azure-dev/main/cli/azd/extensions/registry.json
- https://raw.githubusercontent.com/Azure/azure-dev/main/cli/azd/extensions/azure.ai.agents/CHANGELOG.md
- https://github.com/PrefectHQ/fastmcp/releases/tag/v4.1.0
- https://pypi.org/pypi/{name}/json, one call for each package named in plan section 3
- https://mcr.microsoft.com/v2/bicep/avm/res/{module}/tags/list, one call for each module named in plan section 9.1
