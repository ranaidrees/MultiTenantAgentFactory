# Principal engineer review: the corrections that finish the plan before Build

Date: 2026-10-10 and 2026-10-11. Artifacts reviewed: docs/intent.md revision 19, docs/spec-phase-0-1.md and docs/implementation-plan-phase-0-1.md at commit cc9637d, and then each document as it was revised for D97 to D101.
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

## 5. Independent read of the intent, revision 20 at commit 3d51839

The reviewer was given the five decisions and not the reasoning. It confirmed all nine quotations word for word and found the decision numbers, the figures and the amendment notes consistent. It reported two must-level defects, both in D98, and ten should-level ones. All were accepted.

| Finding | What was done |
|---|---|
| M1. The pipeline identity could not sign in to the registry. Container Registry Repository Writer carries no right on the registry resource, and the roles reference lists "authenticate and login to the registry using `az` CLI via the `az acr login` command" under other roles | The pipeline identity also gets Container Registry Configuration Reader and Data Access Configuration Reader, which reads settings and grants nothing on images. The session checked the claim on the page before accepting it |
| M2. "The one thing a session does in the persistent group" was untrue: `up`, which a session may run after the owner's yes, writes audit records there and assigns the project identity's role on the registry | Restated as the one thing a session does there without the owner's yes; D98 now says which step makes each role assignment |
| S1. A failed spike and an unfinished one were not told apart, and two sentences hid a change to D80 and D81 behind a semicolon | Any miss, a plain failure included, goes to the owner. No spike takes its fallback by itself. The named fallbacks of earlier decisions stay the named ones |
| S2. Parallel sessions share one environment, one gateway and one daily quota, and nothing said so | Only one session at a time holds the dev environment; a risk line was added |
| S3. The condition on the owner's write role was stated as fact. Microsoft's page shows such a condition on a read role only, and copied on to a write role that example would leave writes unconditioned | The condition names the role's two write actions. It is shown and not assumed: at the first build, a build to a name outside `scratch/` must be refused (D31). The owner is told if it is not |
| S4. No cited page supported the pull by Container Apps | The registry's roles overview is cited, which names Azure Container Apps among the pullers the reader role is for |
| S5. D98 did not say where the rule is written, or whether the cost limits apply | CLAUDE.md's Azure writes rule carries the exception; the limits of D21 apply |
| S6. D20 and D9 lacked notes | Added |
| S7. The intent said this file holds what was done with every finding, and at that commit it did not | Section 8 below holds it |
| S8. D100's "one call" named no deployment, and did not say why the question is open or what a refusal changes | One call to each deployment; the page's note on the Foundry Agents Service is named; a refusal means the model is asked for JSON and the result is validated in code |
| S9. D101 did not say from which checkout `up` applies the template, what it must leave alone, or what applies a changed template of the persistent group | From the checkout it is run in, which is why one session holds the environment. It passes back salon-mcp's current image and the current quota. The owner applies a changed persistent template by running the bootstrap again |
| S10. "Three" collided with the spec, which already counts three fixed replies including the older one | Three new fixed texts, four in all with the reply of D46 |
| Could-level: two decisions cited for things they do not say; D97 did not carry "in one go" or say that the items D81 moved stay moved; the flag was alluded to and not named; who deletes a `scratch/` image; what keeps salon-mcp off its `scratch/` image | All corrected. No session deletes a `scratch/` image. The served-image check also fails if salon-mcp is found running one once a release exists, which is a small addition to plan step 1.11 and is the session's own |

Two things in this table went beyond what the owner was asked: the pipeline identity's second role, and the added clause in the served-image check. The second was withdrawn after the spec's read (section 6). The first is put to the owner in the session's summary.

## 6. Independent read of the spec at commit 1d35fb8

The reviewer was given the decisions and not the reasoning. It found every new quotation exact, the role set sufficient for each of the five uses it was asked to trace, and no diagram label made untrue. It reported four must-level defects and six should-level ones, all in or caused by the new text. All were accepted, and two of them reached back into the intent.

| Finding | What was done |
|---|---|
| M1. Exit row 9.1 still said that each no-go has taken its fallback | Each miss has the owner's recorded choice: the named fallback, or another session |
| M2. "No session touches the persistent group, with one exception" dropped the qualifier that the intent carries | Without the owner's yes no session touches it. `up` waits for the owner's yes, and the spec says what `up` writes there when a session runs it |
| M3. `up` was to read "the current token quota" and pass it back, but the quota is on the registry's rows and in the gateway's own parameter, neither of which is in the template; and nothing said what image salon-mcp gets when there is no app | `up` passes back one value, salon-mcp's image: the image the app is running; on a full rebuild, the image the candidate job tagged with the commit of the latest release record; with no record, a placeholder. The quota is not in the template. The intent's D101 was corrected to match |
| M4. The served-image check was to fail on a `scratch/` image "once a release exists". A release record exists from Phase 0, before the pipeline first builds salon-mcp, and the check runs at the start of every pipeline run, so it would have failed the very run that replaces the image | The clause is withdrawn. The guard sits in the candidate job, which deploys salon-mcp from its release repository on every run and confirms that the app runs the image it pushed. Nothing checks it between runs, and the spec says so. The intent's D98 was corrected to match. This removes the addition the session had made in section 5 above |
| S5. Who assigns each role; the condition's two write actions; exact role names; that nobody holds a role that deletes an image | All stated in section 4 and 3.2 |
| S6. The gateway is not part of the template that `up` re-applies; and who applies a merged change before the pipeline tests it | Both stated: the change is applied from its pull request's branch as that step's proof, and the candidate job's what-if on `main` then shows no difference |
| S7. Five places still read as automatic fallbacks | Each restated: on a miss the owner chooses |
| S8. The pins and two other changes rest on no decision | One row in section 2.9 names them as the currency audit and corrections of fact. The rule that the graph opens its tool session for each turn moved from a table cell to section 5.2 |
| S9. The bootstrap was "created once" and yet to be run again | It is safe to run again, and its acceptance check says a second run changes nothing |
| S10. "The registry" used for the container registry in six places | "Container registry" throughout the new text and the cost table |
| Could-level: a throwaway name for the refused build; the ARM-token policy; S5's times now include the template's apply; two unwrapped lines; "meets" | All corrected. One was left: the diagram of section 3.2 shows the project identity's image pull and not salon-mcp's, an omission older than this revision, which no new text makes untrue |

The reviewer also noted that the workstation has no Docker, so `az acr login` there cannot store a credential, and the owner cannot pull or verify an image from it. The plan therefore places the attestation's verification in the promote job, on the runner, and the owner's reader role is for reading tags and manifests.

## 7. What the session found itself

Found while reading the plan for these corrections and while preparing step 0.2. Each is applied in the corrected plan.

| Gap | What was done |
|---|---|
| The storage module closes an account's firewall unless told otherwise; its source calls this the "New default case that enables the firewall by default". salon-mcp, the pipeline and the nightly job reach both storage accounts over public endpoints, because Phase 1 has no private networking (D52), and would have been refused | `networkAcls` is set to allow on both accounts, with the roles as the control. The module's default redundancy, `Standard_GRS`, is replaced by `Standard_LRS`. The spec's risk row on public endpoints now names the storage accounts, the search service and the container registry |
| Step 0.1's proof needs a fresh `ubuntu-latest` job, and no workflow existed until step 0.8 | `pr.yml` in its first form joins step 0.1 |
| Step 0.2 did not say which role assignments of section 9.3 it makes | Those on persistent resources, the gateway identity's metrics role among them, which step 0.5 had held |
| Container Apps needs an identity to pull salon-mcp's image with, and that identity needs a role on the registry | `id-maf-salon-mcp` gets the reader role, and the app's `registries` setting names it |
| The budget module needs a contact | The subscription's Owner role, so that no address is in the repository (D18) |
| The command line's library was unnamed | `argparse`, with no new dependency |
| One quotation in step 0.16 left a word out of the page's sentence | Made exact: "Claude uses this to decide when to apply the skill" |
| The workstation cannot verify an attestation, having no Docker for `az acr login` to store a credential in | The verification is the promote job's, on the runner |
| The session's own citation script paired quotation marks wrongly after a short quotation, and took the text between two code spans for a code span | Both faults fixed, and every document checked again. The counts in this file are from the corrected script |

## 8. What was done with each finding of review 07, section 11

| Finding | What was done | Where in the plan |
|---|---|---|
| 1. Must. A night with no gateway would have ended failed-to-run | With no gateway, `maf check registry` and both steps of the `dev` job are skipped: each says so, exits zero and writes no record. That includes the 22:00 run on a day when the 18:30 run has already deleted the gateway | Steps 0.10 and 1.12 |
| 2. Must. `az acr build` had no stated runner, wrote to the persistent group, and the registry's role mode was unstated | D98. A session runs the builds, into `scratch/` only. The mode is fixed. The bootstrap makes six role assignments on the registry and the tenant module one. `AcrPush` is replaced | Section 3; steps 0.2, 0.3 and 0.4; section 9.3 |
| 3. Should. One job or three | Three: `checks`; `dev` after it, always, with a limit of 30 minutes; `delete` after both, always | Step 0.10 |
| 4. Should. Structured outputs were to be settled in a step that cannot call the model route | D100. One strict-schema call to each deployment joins test 1 of spike S1 as a record, and step 1.3 starts from the answer | Steps 0.6 and 1.3 |
| 5. Should. The gateway's copy was regenerated after steps 4, 8 and 9 only | After 4, 8, 9, 11 and 12 | Section 9.2 |
| 6. Should. `records.py` was first listed in step 1.1 | It is in step 0.8's files. Step 0.15 adds the gate result and step 1.1 the registry rows | Steps 0.8, 0.15 and 1.1 |
| 7. Should. The list of images built outside the pipeline was incomplete | One rule and one command in section 3, and every such image named: two in step 0.4, one each in 0.6 and 0.12, three in 0.15, and salon-mcp's in 1.1 | Section 3 |
| 8. Should. The nightly evaluation ran "by the route step 1.9 uses" | Always by `maf gate run`'s own calls | Step 1.12 |
| 9. Should. The order of the policy and the deployments, and what applies a changed module | Raw deployments from step 0.5, ordered with `dependsOn`. D101: `maf up` applies the environment's template on every run, and the plan says what it passes back | Steps 0.5, 0.10 and 1.7; section 9.1 |
| 10. Should. Cut item 4 dropped two tests | The end-to-end test and its login only. The unit test stays | Section 11.2 |
| 11. Should. The blocklist item was cited to the blocklist's page | A key of its own, and API version 2026-07-01 on both resources | Step 1.7; sections 9.1 and 13 |
| 12. Could. Stale labels | D94 and D97 beside D87 in section 11's heading and risk rows; D96 in step 0.2 | Sections 10 and 11; step 0.2 |
| 13. Could. The layout omitted files | `replies.py`, the two exporters and `tests/fixtures/` | Section 2 |
| 14. Could. Two fixed replies or three | D101: three new fixed texts, four in all. The spec's section 12 is corrected | Spec section 12 |
| 15. Could. Details an engineer would have had to invent | The comparer's one form; `extra='forbid'` with a unit test; the filter test's prompt, token limit and retry every 30 seconds; the manual trigger's two inputs; the order of the outcomes | Steps 1.1, 1.3, 1.7 and 1.12; section 9.6 |

The third estimate, of about four days, is answered by D97: the day figures are sizes. Section 11.3 of the plan is new and says which Phase 0 steps wait for which, and which may run in parallel sessions.

## 10. Sources opened in this session

Every quotation in this file was matched to its page by script on 2026-10-11.

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
- https://raw.githubusercontent.com/Azure/bicep-registry-modules/main/avm/res/storage/storage-account/main.bicep
- https://raw.githubusercontent.com/Azure/bicep-registry-modules/main/avm/res/cognitive-services/account/main.bicep
- https://learn.microsoft.com/azure/azure-resource-manager/bicep/resource-dependencies
- https://learn.microsoft.com/azure/container-apps/get-started
- https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax
- https://docs.github.com/en/actions/reference/workflows-and-actions/expressions
- https://code.claude.com/docs/en/skills
