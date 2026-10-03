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
At each gate run `/council <artifact path>`; the review is written to `docs/council/NN-<stage>-review.md`.

## Rules

- One stage per session. Stop when that stage's artifact is committed.
- Plan mode first: read, propose a plan, wait for approval, then change files.
- Ask, do not assume. If something is unclear, ask the owner before acting.
- Evidence with URLs: every design claim cites an official document or sample.
- Reuse before build: check official docs and samples first (Microsoft Learn MCP, Context7, web search).
- Azure writes (intent D9): before any command or MCP tool call that creates, changes or deletes
  Azure resources, state the estimated added monthly cost and the blast radius. Proceed without
  asking only if all of these hold: up to £20 a month; touches only this project's resource groups;
  deletes nothing; no role, policy or Entra change outside the bootstrap. Otherwise stop and wait
  for the owner's yes. Releases still go through GitHub Actions with environment approval.
- Writing style: plain British English, no em dashes.

## Architecture

TODO: add after `docs/spec-phase-0-1.md` is accepted.

## Commands

TODO: add build, test, lint and run commands when code exists.
