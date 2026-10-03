# Session 00: Stage 0, delivery harness

Date: 2026-10-03. Tool: Claude Code Desktop (Windows). Model: Claude Opus 5.5.
Commits: eb59d95 (harness and intent revision 7), plus the commit that adds this entry and council review 02.

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

## 4. Surprises and how they were handled

- **The prompt and the intent disagreed.** The prompt said `docs/spec.md` and
  `docs/implementation-plan.md`; the intent said per-phase `spec.md` and `plan.md`; the playbook
  says `plan.md`. Asked, then recorded D8.
- **The owner reversed a rule stated in the prompt** (no Azure writes). Asked a second round to
  remove ambiguity, recorded D9, and removed the non-goal it contradicted rather than leave the
  intent self-contradictory.
- **Three GitHub accounts.** gh was logged in as two accounts, neither the one wanted. A browser
  sign-in does not log the CLI in. The fix is the gh device login (`gh auth login --web`), which
  the owner completes in the browser with a one-time code. The first code expired unused, so the
  session ended with two local commits and no remote.
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

## 5. What was produced

- [CLAUDE.md](../../CLAUDE.md): process rules only.
- [.claude/agents/](../../.claude/agents/): five council members and the chair.
- [.claude/commands/council.md](../../.claude/commands/council.md): the `/council` command.
- [.mcp.json](../../.mcp.json): Microsoft Learn MCP (3 tools), Context7 (2 tools), Azure MCP
  pinned to 3.0.0-beta.49 (70 tools). All three verified to list their tools.
- [.gitignore](../../.gitignore) for Python, Node and Azure.
- [docs/intent.md](../intent.md) revision 7 with D7 to D12.
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
- Log the gh CLI in as `ranaidrees` (`gh auth login --hostname github.com --git-protocol https
  --web --scopes workflow`) so the private repository can be created and the commits pushed.
- Answer the twelve owner questions in [02-intent-review.md](../council/02-intent-review.md).
  Questions 1 to 3 ask you to reconsider D9, D10 and D12; the Contrarian's verdict becomes Reject
  if D9 and D12 stay as written.
- Decide whether to install `azd` and Docker (both missing), and whether to upgrade npm (9.2.0)
  and gh (2.76.2).
- Start a new session and approve the three project MCP servers when prompted. Check `/mcp` shows
  `azure` connected, and that `/council` and the `council-*` agents are listed.
- Optional: end the two stale `azmcp.exe` processes and fix or remove the user-level `azure` MCP
  entry, which duplicates the project one.

Next session (Stage 2, Design):
- If the owner's answers change the intent, record them as revision 8 first and commit.
- Write `docs/spec-phase-0-1.md`. Its clarifications section must cover D11 (data store and the
  intent's questions 5 to 7) and the council's unopposed corrections (debate 7 in review 02).
- Run `/council docs/spec-phase-0-1.md`, this time with the registered agents.
