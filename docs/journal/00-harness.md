# Session 00: Stage 0, delivery harness

Date: 2026-10-03. Tool: Claude Code Desktop (Windows). Model: Claude Opus 5.5.
Commits: eb59d95 (harness and intent revision 7), 76be8a0 (council review 02 and this journal), 238ceec and one
later commit (journal corrections). Remote: https://github.com/ranaidrees/MultiTenantAgentFactory (private).

## 1. Opening prompt

```text
We will deliver this project using Anthropic's AI-Native SDLC playbook
(https://claude.com/blog/the-ai-native-sdlc-playbook): each stage commits one
artifact that the next stage reads, and the commit history is the audit trail.

This session is Stage 0: set up the delivery harness only. Do not create
application code, Python packages, infrastructure or the spec yet.

Read first:
- docs/intent.md (accepted; source of truth, including decisions D1 to D6)
- docs/research.md
- docs/council/01-intent-review.md

Rules for this and every later session:
- Ask, do not assume. If something is unclear, ask me before acting.
- Reuse before build: check official docs and samples with web search and cite URLs.
- Never run Azure write or deploy commands. Only GitHub Actions will deploy.
- Writing style: plain British English, no em dashes.

Tasks:
1. Interview me about anything in the intent that affects the harness or the
   next stage. Include this open question: should Langfuse move from Phase 6
   into Phase 1? Record my answers as new decisions in docs/intent.md.
2. Initialise git if needed, add a sensible .gitignore for Python, Node and
   Azure, and ask me before creating a private GitHub repo with gh.
3. Create a minimal CLAUDE.md (under one page) holding process rules only:
   the artifact chain (docs/intent.md, docs/spec.md,
   docs/implementation-plan.md, code, PR), one stage per session, plan mode
   first, ask rather than assume, evidence with URLs, reuse before build, never
   deploy from a session. Mark architecture and commands as TODO for later.
4. Create the review council, adapted from karpathy/llm-council and
   tenfoldmarc/llm-council-skill:
   - .claude/agents/: council-architect, council-security, council-simplifier,
     council-hiring-manager, council-contrarian, council-chair.
   - .claude/commands/council.md: given an artifact path, run the five members
     as independent subagents; each returns a verdict, top five issues ranked
     by impact, evidence links and owner questions; then one anonymised
     rebuttal round; the chair writes docs/council/NN-<stage>-review.md with
     verdict, debates, resolutions, dissent and owner questions. The chair
     never adds scope.
5. Add project-scoped MCP servers in .mcp.json: Microsoft Learn MCP and
   Context7 for current documentation, Azure MCP for read-only use. Verify each
   one lists its tools. Ask before adding anything else.
6. Check local tools and report versions: python, uv, node, git, gh, az, azd,
   docker. List what is missing and ask before installing anything.
7. Run /council on docs/intent.md as a smoke test of the council.
8. Commit with a clear message, show me a summary of what was created and any
   questions, then stop.
```

A second request arrived mid-session: keep a log of each session so the process can be explained
to an interviewer and turned into a template. That request created this journal.

## 2. What happened, in order

1. Read the three documents and listed the folder (docs only, no git).
2. Checked local tool versions, git identity and gh logins. Read-only.
3. Researched before building: the playbook, both council repositories, Claude Code docs for
   subagents, commands and MCP, and the docs for the three MCP servers and Langfuse.
4. Interview round 1 (harness): Langfuse, artifact layout, Azure bootstrap, deploy guard.
5. The owner's answers reversed the "never run Azure writes" rule, so interview round 2 pinned
   down scope, approval model, cost ceiling and Azure MCP mode.
6. Interview round 3 (repository and next stage): GitHub account, Playwright, data store, open
   council defaults. A follow-up fixed the commit author.
7. Initialised git on `main` with a repo-local author. Wrote `.gitignore`.
8. Wrote `.mcp.json` and verified each server with a small MCP client (initialize, then tools/list).
9. Wrote `CLAUDE.md`, recorded D7 to D12 in the intent as revision 7, wrote six council agents and
   the `/council` command.
10. Committed the harness (eb59d95) so the council had a revision to cite.
11. Ran the council on `docs/intent.md`: five reviews, one anonymised rebuttal round, chair synthesis.
12. Checked the council's two headline findings against the sources directly.
13. Added this journal, its template and a CLAUDE.md rule. Adjusted `/council` from what the smoke
    test showed. Committed.
14. Created the private repository after a second gh device login, and pushed.
15. The owner answered the council: prod writes and Azure MCP writes need approval every time, the
    repository stays private unless the plan gets in the way, and the council's cost is accepted.
16. Installed azd with winget. Added a sixth council member and named each member's expertise.
17. Created the `/cleanup` skill and ran it once, report only. Recorded D13 to D17 as intent
    revision 8. Committed and pushed.
18. The owner approved three things. Applied cleanup items 1 to 3 (intent revision 9, commit
    254806e). Ended two stale `azmcp.exe` processes and removed the user-level `azure` MCP entry.
    The owner decided to make the repository public (D18, revision 10); the session was not
    permitted to change visibility, so that step is left to the owner.

## 3. Owner decisions

| Question | Answer | Recorded in |
|---|---|---|
| Langfuse into Phase 1? | Yes, export only: Langfuse Cloud free tier, dev, synthetic data | intent D7 |
| Artifact layout | Per-phase files; `implementation-plan` replaces `plan` | intent D8 |
| Azure writes from sessions | Allowed in any environment after a cost and blast radius assessment; auto-run when low risk (up to £20 a month, own resource groups, no deletes, no role, policy or Entra change outside bootstrap) | intent D9 |
| Azure MCP mode | Write tools enabled; Playwright deferred to Phase 4 | intent D10 |
| Data store and questions 5 to 7 | Left open for the spec | intent D11 |
| Repository and author | Private, under github.com/ranaidrees; commits as Rana Idrees (noreply) | intent D12 |
| Session journal | One entry per session from a template | CLAUDE.md, this folder |
| Council questions 1 and 2 | Prod writes and Azure MCP writes always need the owner's yes; dev stays auto-run when low risk; written rule only | intent D13, D14 |
| Council expertise | Add an AI Engineer; name what each member brings | intent D15 |
| Repository visibility | Public, after seeing which GitHub gates a private repository lacks | intent D16, D18 |
| Clutter | A `/cleanup` skill that reports first and changes only with approval | intent D17 |
| Install azd | Yes (1.35.0 installed). Docker not installed | this entry |

## 4. Surprises and how they were handled

- **The prompt and the intent disagreed.** The prompt said `docs/spec.md` and
  `docs/implementation-plan.md`; the intent said per-phase `spec.md` and `plan.md`; the playbook
  says `plan.md`. Asked, then recorded D8.
- **The owner reversed a rule stated in the prompt** (no Azure writes). Asked a second round to
  remove ambiguity, recorded D9, and removed the non-goal it contradicted rather than leave the
  intent self-contradictory.
- **Three GitHub accounts.** gh was logged in as two accounts, neither the one wanted. A browser
  sign-in does not log the CLI in. The fix is the gh device login (`gh auth login --web`), which
  the owner completes in the browser with a one-time code. The first code expired unused; the
  second succeeded, and the private repository was created and pushed.
- **The existing user-level Azure MCP server was broken.** Two stale `azmcp.exe` processes locked
  the npx cache, so `@latest` could not upgrade (EBUSY). Pinning an exact version in `.mcp.json`
  avoided it and is better supply-chain practice anyway.
- **New agents were not registered when first needed.** Two attempts to start `council-*` agents
  failed, so the smoke test ran each member as a general-purpose subagent told to read its own
  definition file. Prompts and protocol were tested; tool limits were not. The agent types did
  register later in the same session, with the intended tool lists, so the definitions are valid.
- **Passing reviews inline was wasteful.** The orchestrator would retype every review five times.
  Reviews now travel as files in a temporary folder; `/council` was updated.
- **The council costs real budget.** One run used about 1.6 million subagent tokens and about 25
  minutes (five reviews, five rebuttals, one chair).
- **The council challenged the session's own decisions.** All five members asked for D9, D10 and
  D12 to change. The chair recorded this as owner questions and changed nothing.
- **The GitHub plan decides which gates exist.** On GitHub Free a private repository has no
  environments and no protected branches, and required reviewers need a public repository on Free,
  Pro and Team. The token could not read the account's plan, so the owner must confirm it.
- **The council had a gap in its own expertise.** No member owned agents, retrieval, evaluation and
  guardrails; runtime prompt-injection defence surfaced only in a rebuttal. Fixed by D15.

## 5. What was produced

- [CLAUDE.md](../../CLAUDE.md): process rules only.
- [.claude/agents/](../../.claude/agents/): six council members and the chair.
- [.claude/commands/council.md](../../.claude/commands/council.md): the `/council` command.
- [.claude/skills/cleanup/SKILL.md](../../.claude/skills/cleanup/SKILL.md): the `/cleanup` skill.
- [.mcp.json](../../.mcp.json): Microsoft Learn MCP (3 tools), Context7 (2 tools), Azure MCP
  pinned to 3.0.0-beta.49 (70 tools). All three verified to list their tools.
- [.gitignore](../../.gitignore) for Python, Node and Azure.
- [docs/intent.md](../intent.md) revision 8 with D7 to D17.
- [docs/council/02-intent-review.md](../council/02-intent-review.md): verdict Accept with changes,
  twelve owner questions.
- This journal and [TEMPLATE.md](TEMPLATE.md).

## 6. Reusable lessons

- Put the rules in the opening prompt, but expect the interview to change them. Budget for a
  second interview round whenever an answer contradicts the brief.
- Research before the interview, so each question comes with options, a recommendation and a source.
- Offer a recommendation with every question, and record the answer even when it goes against it.
- Check accounts and identities (git author, gh login, cloud subscription) in the first minutes.
- Pin tool versions in `.mcp.json`. `@latest` broke an existing setup on this machine.
- Create agents and commands early. They can take a while to register in the session that creates
  them, so plan to use them properly in the next one.
- Commit the artifact before reviewing it, so the review can cite a commit.
- Run the council before decisions harden. Here it found two facts that change the plan: GitHub
  required reviewers are not available on private repositories on Free, Pro and Team plans, and
  APIM v2 tiers cannot currently be created in UK South.
- For the prompt next time: name the GitHub account and the repository name, say whether the
  council should use a cheaper model, and state the commit plan (one commit or one per artifact).

## 7. Next session

Owner, before the next session:
- This repository pushes through the active gh account. If a push is refused, run
  `gh auth switch --user ranaidrees`.
- Make the repository public yourself (D18), in GitHub under Settings, General, Danger Zone. A
  session is not permitted to do this for you.
- Answer questions 4 to 12 in [02-intent-review.md](../council/02-intent-review.md), or leave them
  for the spec's clarifications section. Questions 1 and 2 are answered by D13 and D14.
- Decide whether to install Docker (missing), and whether to upgrade npm (9.2.0) and gh (2.76.2).
- Start a new session and approve the three project MCP servers when prompted. Check `/mcp` shows
  `azure` connected, and that `/council`, `/cleanup` and the seven `council-*` agents are listed.
- The user-level `azure` MCP entry was removed, so other projects on this machine no longer have
  Azure MCP. To restore it there: `claude mcp add azure -s user -- cmd /c npx -y @azure/mcp@latest
  server start`.

Next session (Stage 2, Design):
- If the owner's answers change the intent, record them as revision 11 first and commit.
- Confirm the repository is public before the spec relies on environments or required reviewers.
- Write `docs/spec-phase-0-1.md`. Its clarifications section must cover D11 (data store and the
  intent's questions 5 to 7) and the council's unopposed corrections (debate 7 in review 02).
- Run `/council docs/spec-phase-0-1.md`, this time with the registered agents.
