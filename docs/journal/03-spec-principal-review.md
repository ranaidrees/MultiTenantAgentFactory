# Session 03: Stage 2, principal review of the revised spec and the owner's decisions on it

Date: 2026-10-03 to 2026-10-04. Tool: Claude Code Desktop (Windows). Model: Claude Fable 5.1.
Commits: 43d6c0b (principal review 04), 8f41a76 (intent revision 13), 1afee48 (spec revision),
21edba3 (CLAUDE.md rule and journal template), 7b5de1a (cleanup), and the commit that adds this entry.
The spec is revised again and still not accepted; see section 7.

## 1. Opening prompt

The owner's first message, verbatim:

```text
please act as a AI principal engineer, who is expert IN AI Platform and Azure And Azure Foundry,
Langrpah, AI evaluation AI guardrails and AI Agent OPS and then understand and the intent.md and
critically review the Specs.md . Can you also validate that we have used AI native SDLC by
anthropic in spirit and on the path to implement AI Native SDLC in build, test and deployment
stage. Do you think our validation gate is fine. In Specs.md we should have high level Design
and high level architecture diagrams. It is important for me to understand the Agent framework
AgrntOps Deployed in Azure and how our agnet will map to it. Also Spcs.md should also tell how
the good looks like. what are the validations tests. Use Specs.md and see internet to see if we
are using latest tools and latest concpets of Azure and latest packages and latest versions. LLM
usually looks and works on the things which are old but if people are doing something better and
they are better approaches then please challenge the specs. Provide me a good prompt first
```

The session drafted the review prompt below from the repository and the owner ran it with
"start in plan mode". It is the reusable asset of this session.

```text
Act as an AI Principal Engineer reviewing this project. Your expertise: AI platforms on Azure,
Microsoft Foundry (hosted agents, evaluations, guardrails, tracing, AI gateway), LangGraph and the
Microsoft Agent Framework, MCP, AI evaluation, runtime guardrails and AgentOps. You are reviewing,
not building. This session is part of the Stage 2 gate of Anthropic's AI-Native SDLC playbook
(https://claude.com/blog/the-ai-native-sdlc-playbook): an independent review of the revised spec
before I decide whether to accept it. Do not create application code, infrastructure or the
implementation plan, and do not edit the spec or the intent until I decide.

Read first, in this order:
- CLAUDE.md (process rules; follow them all)
- docs/journal/02-spec-gate.md, section 7 (the handoff from the last session)
- docs/intent.md (revision 12, decisions D1 to D55; source of truth)
- docs/spec-phase-0-1.md (revised after council review 03; awaiting my acceptance)
- docs/council/03-spec-phase-0-1-review.md (what the council already found; do not repeat it)
- docs/research.md (the evidence base)

Pre-flight; report the result and stop if any item fails:
- /mcp shows azure, microsoft-learn and context7 connected.
- git is clean and in sync with origin at dea7217 or later.

Rules for this session:
- Start in plan mode: read, show me the review plan, wait for my yes.
- Read-only. No Azure writes. No edits under docs/ and no commits. Write the full review to a
  Markdown file in the session scratchpad and send it to me; give the summary in chat.
- Research before you ask (D55). Ask at most one round of up to four questions, each with
  options, a recommendation and a source. For everything else state your recommendation and
  carry on.
- Evidence with URLs: every claim that something is current, superseded, preview or GA cites an
  official page or sample you opened in this session, dated today. Paste quotations; do not
  retype them. Use Microsoft Learn MCP, Context7 and web search. If you cannot verify a claim,
  write "unverified".
- Challenge, do not relitigate. D1 to D55 stand unless you show that the facts behind one have
  changed or that a materially better approach exists. Then name the decision, what changed and
  what you propose.
- Judge by this project's goal (intent section 2) and constraints (intent section 9: solo
  engineer, £40 a month, five weeks), not by what a large enterprise would build.
- Plain British English, no em dashes. Tables over prose wherever things are compared.

Tasks:

1. Currency audit. For every tool, package, service, API version, feature status and model the
   spec names, check the official source today and fill one table: item; what the spec says;
   what the source says today; verdict (current, superseded, status changed, unverified); impact
   on the spec; URL. The list below is the minimum, not the whole:
   - Hosting: langchain_azure_ai.agents.hosting and ResponsesHostServer; FoundryCheckpointSaver
     and container protocol 2.0.0; the azd azure.ai.agents extension; hosted agent GA status,
     regions and sandbox sizes; the release-without-changing-production flow.
   - Framework: the current LangGraph version and its durable execution model; whether the
     Microsoft Agent Framework has overtaken LangGraph for Foundry hosted agents in tracing,
     evaluation or governance, with evidence either way.
   - Evaluation: microsoft/ai-agent-evals v3-beta against the azure-ai-evaluation SDK, Foundry
     cloud evaluations and continuous (online) evaluation for agents; the agent evaluators'
     preview status; evaluator availability in UK South; the AI Red Teaming Agent.
   - Guardrails: Foundry agent guardrails and rai_policy_name; Prompt Shields; task adherence
     and PII guardrails; tool-response screening; Defender for Cloud AI threat protection.
   - Gateway: API Management v2 availability in UK South today; llm-token-limit with a daily
     quota; llm-emit-token-metric dimensions; API Management support for MCP; the status of
     Foundry's AI Gateway integration.
   - Identity and MCP: Entra Agent ID, agent identity tokens and the xms_par_app_azp claim;
     Foundry MCP connections with agentic-identity authentication; the MCP specification
     version; the MCP Python SDK and FastMCP versions.
   - Observability: the OpenTelemetry GenAI semantic conventions (gen_ai.agent.id,
     gen_ai.agent.version, gen_ai.conversation.id, gen_ai.tool.name, gen_ai.request.model)
     against the spec's nine attribution keys; AzureAIOpenTelemetryTracer; Foundry's built-in
     agent monitoring against the Azure Workbook; Langfuse's OTLP endpoint.
   - Models: gpt-5.4-nano and gpt-5.4-mini version 2026-03-17 as Global Standard in UK South,
     their retirement dates, and whether a model router changes D50.
   - Release evidence: GitHub immutable releases; artifact attestations (SLSA provenance) for
     the image digest; image signing with Azure Container Registry.
   - Harness: Claude Code hooks, skills, subagents and plan mode as the spec describes them;
     the Azure MCP server version pinned in D10.

2. Critical review of the spec. Rank findings by impact. For each: title; section of the spec;
   why it matters; what the current source says; the change you propose; URL.
   a. Architecture and diagrams. The spec has one diagram (section 3.1). Say which views a
      high-level design needs and lacks. At least: context; deployment (resource groups,
      regions, what is torn down nightly); identity flow (section 4 as a picture); the booking
      conversation with the confirmation interrupt as a sequence diagram; the release pipeline
      with its gates; telemetry. Draft each missing diagram in Mermaid, as a proposal, so it can
      go into the spec unchanged if I accept it.
   b. AgentOps on Azure and how our agent maps to it. Explain in one page, with sources, the
      Foundry hosted agent lifecycle as Microsoft describes it today: resource, project, agent,
      version, selector and traffic rules, session, agent identity, state store, connections
      and tools, tracing, evaluations, guardrails, monitoring. Then a mapping table: our concept
      (tenant, agent, agent_version, salon-mcp, knowledge base, attribution keys, release,
      rollback, eval gate, guardrail, audit) to the Foundry object, the Entra object, the GitHub
      object, the Azure Monitor or Langfuse object, and the spec section. Mark every row where
      the mapping is missing, weak, or custom where a platform feature exists.
   c. What good looks like. Judge whether a stranger could tell from the spec what a good
      result is. Propose a "Definition of good" section: three golden conversations (book,
      cancel, FAQ with citation); quality targets with a rationale (pass rates, groundedness,
      the "I do not know" rate on unanswerable questions, booking success, p95 latency, tokens
      per conversation); platform control targets (section 9); developer experience targets
      (the stranger test, time to a served agent); AgentOps loop targets (time from a bad trace
      to a failing test). Say which starting figures in section 5.8 are placeholders and whether
      measuring thresholds from two runs in spike S6 is enough.
   d. Validation tests. Produce a test matrix: layer (unit, contract, integration, scripted
      conversation, judged eval, negative demonstration, rebuild and teardown, load, chaos); the
      test; what it proves; where it runs (pull request, candidate, nightly, exit
      demonstration); model in the loop or not; present in the spec (yes, partial, no). Name
      what is missing and whether it belongs in Phase 1 or later. Look in particular at:
      rollback as a drill rather than a sentence; a canary using Foundry's FixedRatio rules
      rather than an all-or-nothing selector move; eval dataset versioning; online evaluation
      and drift; red teaming; the poisoned-passage test, which the spec says passes by
      construction; the served-image check; the local two-tenant test.
   e. The validation gates. Assess each and say whether it is fine, naming its weakest point:
      the council gate (six members, one rebuttal, a chair); the eval gate (section 5.8); the
      approval gate (section 5.9 and D41); the exit criteria (section 9); the Phase 0 spikes as
      gates. For each, say whether it can be passed without the thing it claims to prove, and
      how.
   f. Challenge list. Where a better approach exists today than the one the spec chose, say so
      with evidence, cost and effort, and name the decision it touches.

3. AI-native SDLC conformance. For each stage of the playbook (Plan, Design, Build, Test,
   Deploy, Maintain) and each named practice (the artifact chain; plan accepted before code;
   CLAUDE.md; skills; hooks; subagents; parallel sessions; a feedback loop the agent can run
   itself; continuous evals that run whenever the agent's configuration changes; the bug-fix
   protocol with a test-edit hook; REVIEW.md review passes ranked by severity; hooks as approval
   gates; deploy, status and rollback exposed as MCP tools scoped per environment; tiered
   autonomy; control bands with tiered response; scheduled security scans), fill one table:
   practice; what the repository does today; what the spec plans; verdict (done, on the path,
   deviation, gap); what would close it; URL. Say plainly whether the project follows the
   playbook in spirit for Build, Test and Deploy, and where it deviates by choice (GitHub
   Actions rather than deployment through MCP; the written rules in D20 and D41 rather than
   hooks) whether the deviation is sound.

4. Output. In the review file: a verdict (accept, accept with changes, reject) with one
   sentence of reason; the findings of task 2 ranked by impact; the tables of tasks 1, 2d and 3;
   the diagrams of 2a; the mapping of 2b; the definition of good of 2c; then a numbered list of
   proposed spec changes, each marked must (blocks acceptance), should or could; and a list of
   the decisions that would need a new number (D56 onwards) with your recommended answer. In
   chat: the verdict, the top five findings in one line each, and your questions if any. Then
   stop. I decide what enters the spec, whether the council runs again, and whether the review
   is kept under docs/.
```

Three later messages changed the course of the session: a pasted summary of a podcast with
Robert C. Martin on AI-era engineering practice, with "Do we have great harness and see do we
have relevant uml diagrams. Do you think we should incorporate some of his advices"; then "so
what do you recommend now"; then the answers to one round of three questions.

## 2. What happened, in order

1. The owner asked for a review prompt. The session read the intent, the spec, council review 03,
   journal 02, the research note and the playbook, and drafted the prompt above.
2. The owner said "start in plan mode". Pre-flight passed. The plan was approved.
3. Four research subagents ran the currency audit in parallel, one cluster each (hosting and
   models; evaluation and guardrails; gateway, identity and MCP; observability, evidence and
   harness), while the session read the Foundry lifecycle pages itself. About 1.3 million
   subagent tokens.
4. Two audit results reversed statements in the spec. Both pages were re-read in full before use:
   a preview page documenting a 90/10 canary against two pages saying traffic splitting is
   unsupported, and the cloud evaluation page showing the GitHub Action is a wrapper over the
   project evaluation API.
5. The review was written to the scratchpad and sent: accept with changes, twelve findings.
6. The owner pasted the podcast summary and asked three questions. The session answered in chat
   and appended four engineering-practice additions to the review as its section 14.
7. The owner asked for a recommendation. One round of three questions: all recommendations
   applied; the review kept under docs/council/; no council rerun.
8. The review was copied into docs/council/04 and committed first, as the evidence revision 13
   cites. A 146 KB page a subagent had saved at the repository root was removed before the commit.
9. Intent revision 13 (D56 to D70) was written and committed.
10. The spec was revised in three batches, checked by script (every citation key resolves, code
    fences balance, no stale phrase remains) and committed. The check caught one false citation
    key, a package extra written in square brackets, which was reworded.
11. CLAUDE.md gained the D70 rule and the journal template its line; committed.
12. /cleanup reported four findings in docs/research.md; all four were approved and applied.
13. This entry, commit and push.

## 3. Owner decisions

| Question | Answer | Recorded in |
|---|---|---|
| Which of the review's proposed changes to apply | All recommendations; could items off except the Defender trial | intent D56 to D70 |
| Keep the review in the repository | Yes, as docs/council/04-spec-phase-0-1-principal-review.md | commit 43d6c0b |
| Rerun the council on the revised spec | No; the council reviews the implementation plan at the Stage 3 gate | intent D70, CLAUDE.md |
| Cleanup findings | All four | commit 7b5de1a |
| Accept the revised spec | Not yet asked; the owner has not read the revision | this entry |

## 4. Surprises and how they were handled

- **The Action is a thin wrapper.** The spec's problem that the eval Action "declares no output a
  script can read" dissolves once its code is read: it targets the agent version through the
  project evaluation API, which returns results. Spike S6 now runs both routes (D56).
- **Learn pages contradict each other.** Two pages say traffic splitting between agent versions
  is unsupported; a newer preview page documents a canary. Recorded as a discrepancy with a
  five-minute check in spike S5, not as a design change.
- **A promising route was legacy.** Agent applications put version promotion behind a control-plane
  permission, which would have let Azure enforce the gate. The page is marked legacy and the
  model is deprecated, so the council's finding stands and the check is recorded as rejected.
- **The sample base does the wrong kind of authentication.** python-mcp-demos' Entra sample is a
  user sign-in proxy, and it pins FastMCP 3 while the specification moved a major revision in
  July. It is now reused for layout only (D61).
- **A subagent wrote into the repository.** A fetched page landed at the repository root as an
  untracked file. Removed before any commit; a rule for audit briefs is proposed below.
- **The citation check earned its keep.** A package extra in square brackets read as a citation
  key. Reworded; the convention is proposed below.
- **The owner took the recommendations as a block again.** Questions were kept to one round of
  three, each with a recommended option.

## 5. What was produced

- [docs/council/04-spec-phase-0-1-principal-review.md](../council/04-spec-phase-0-1-principal-review.md): the review, 14 sections.
- [docs/intent.md](../intent.md): revision 13, decisions D56 to D70.
- [docs/spec-phase-0-1.md](../spec-phase-0-1.md): revised; seven Mermaid views; section 3.4 (platform mapping) and 5.11 (Definition of good) added; awaiting acceptance.
- [CLAUDE.md](../../CLAUDE.md): the D70 rule. Architecture is still a TODO.
- [docs/journal/TEMPLATE.md](TEMPLATE.md): one line added to section 6.
- [docs/research.md](../research.md): four statements aligned with revision 13.
- This entry. No code, infrastructure, implementation plan or Azure write.

Cleanup record: four statements in docs/research.md were corrected and one line added at its
top; nothing was removed or merged; one stray untracked file was removed before commit.

## 6. Reusable lessons

- A single deep review with a currency audit, run after a council review, found what six
  constrained members did not: the statistics of the eval gate, moved dependencies, an
  undocumented identity path, a wrapper where a harness was assumed. That is now the rule for
  revised artifacts (D70).
- Four parallel research subagents, each with the spec's exact claims and an order to paste
  quotations, finished a 40-item audit in about ten minutes. Re-read in full every page whose
  result reverses a design statement.
- Audit briefs must say "save nothing under the repository; use the scratchpad".
- Run the citation-key and code-fence check before every spec commit. It is a three-line script
  and it caught a false key. A Phase 0 pull request check could run it.
- In a document where square brackets are citation keys, write package extras in words.
- Offer "take all my recommendations" after the first round of questions; this owner takes it.
- Harness mechanism not needed this session (D70): the council, by design, and plan mode's
  exploration subagents, because the files had been read directly before the plan.

## 7. Next session

Owner, before the next session:
- Read the revised docs/spec-phase-0-1.md, in particular sections 3.4, 5.8, 5.11, 7 and 9, and
  D56 to D70 in section 14 of docs/intent.md, which were recorded on your instruction as a block.
- Decide whether you accept the spec, or what you want changed.
- Nothing to install. If a push is refused, run `gh auth switch --user ranaidrees`.

Next session (finish the Stage 2 gate; Stage 3 only if the spec is accepted and only if the owner
says to continue, since CLAUDE.md allows one stage per session):
1. Pre-flight as in this session, and read this section.
2. Ask the owner for the decision on the spec. If changes are wanted, make them, record any that
   alter the intent as revision 14 (D71 onwards) and commit. Do not rerun the council (D70).
3. On acceptance: set the spec's status to accepted, fill the Architecture section of CLAUDE.md
   from sections 3.1, 3.4 and 4 in about eight lines, keep the file under one page, and commit.
4. Stage 3, docs/implementation-plan-phase-0-1.md, starts only after step 3 and in its own
   session unless the owner says otherwise. Its council review is the next council run.

Still open: the daily token quota, which returns after spike S6 measures a gate run (D39); the
search tier, after spike S4 (D27); which eval route the gate step uses, after spike S6 (D56);
whether a trace evaluation runs from UK South (D58); the EU-region project for safety and red
teaming, decided in Phase 2 (D63); the price of Defender for AI Services after its trial (D66);
the six spikes, which belong to Phase 0; the accepted risks in section 11 of the spec; the
proposal from journal 02 to raise the chair's word limit.
