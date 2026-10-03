# CLAUDE.md

Process rules for this repository. Keep this file under one page.
Scope and owner decisions live in docs/intent.md, which is the source of truth.

## Artifact chain

Each stage commits one artifact that the next stage reads. The commit history is the audit trail.

1. `docs/intent.md`: accepted intent and owner decisions.
2. `docs/spec-phase-N.md`: design for the phase.
3. `docs/implementation-plan-phase-N.md`: build plan, accepted before any code.
4. Code and tests.
5. Pull request with review findings.

Phases 0 and 1 share `docs/spec-phase-0-1.md` and `docs/implementation-plan-phase-0-1.md`.
A stage starts only when the previous artifact is committed and accepted by the owner.
At each gate run `/council <artifact path>` on a new artifact; the review is written to
`docs/council/NN-<stage>-review.md`. A revised artifact gets one deep single-reviewer pass with a
currency audit instead of a council rerun (D70), written to `docs/council/NN-<stage>-principal-review.md`.

## Rules

- One stage per session. Stop when that stage's artifact is committed.
- Plan mode first: read, propose a plan, wait for approval, then change files.
- Ask, do not assume. If something is unclear, ask the owner before acting. Research before the
  interview (D55): check the facts against current documentation first, so that each question
  comes with options, a recommendation and a source.
- Evidence with URLs: every design claim cites an official document or sample.
- Reuse before build: check official docs and samples first (Microsoft Learn MCP, Context7, web search).
- Azure writes (intent D9, D13, D14, D20, D21, D40): before any command or MCP tool call that creates,
  changes or deletes Azure resources, state the estimated added monthly cost and the blast radius.
  Then:
  - Prod, through any tool: always wait for the owner's yes.
  - Azure MCP write, in any environment: always wait for the owner's yes.
  - az or azd in dev: proceed without asking only if all of these hold: up to £20 a month; the
    month's forecast for dev stays under £40; touches only this project's resource groups; deletes
    nothing; no role, policy or Entra change outside the bootstrap. Otherwise wait for the owner's
    yes. One exception (D40): the repository's gateway teardown may run without asking. A full
    `down` waits for the owner's yes, and nothing touches the persistent group.
  These are written rules with no hook behind them (D20). Releases still go through GitHub Actions.
- Approvals (D41): never approve or reject a GitHub deployment. The owner approves in the browser.
- No clutter: add a document only if it is a chain artifact, a council review, a journal entry,
  harness configuration, or the owner asks for it. Run `/cleanup` before the final commit of a stage.
- Session journal: start by reading section 7 of the latest entry in `docs/journal/`. Before the
  final commit, write `docs/journal/NN-<stage>.md` from `docs/journal/TEMPLATE.md`.
- Writing style: plain British English, no em dashes.

## Architecture

TODO: add after `docs/spec-phase-0-1.md` is accepted.

## Commands

TODO: add build, test, lint and run commands when code exists.
