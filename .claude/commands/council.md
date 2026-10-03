---
description: Run the five-member review council on a delivery artifact and write docs/council/NN-<stage>-review.md
argument-hint: <artifact path, for example docs/spec-phase-0-1.md>
---

Run the review council on the artifact at: $ARGUMENTS

Adapted from Karpathy's LLM Council (https://github.com/karpathy/llm-council: independent first
opinions, anonymised peer review, chairman synthesis) and the llm-council Claude Code skill
(https://github.com/tenfoldmarc/llm-council-skill: parallel subagents, neutral framing, reviews
relabelled A to E). Here the members are fixed roles rather than different models, and peer
ranking is replaced by one rebuttal round.

You are the orchestrator. You do not review the artifact, summarise it for the members or give an
opinion on it. Follow the steps in order.

## 1. Prepare

1. If the path is missing or the file does not exist, ask the owner for the path and stop.
2. Stage name: the artifact's file name without its extension (`docs/intent.md` gives `intent`).
3. Review number NN: one more than the highest two-digit prefix in `docs/council/`, starting at 01.
4. Output path: `docs/council/NN-<stage>-review.md`.
5. Revision: the revision stated in the artifact's header if it has one, plus the short hash of the
   last commit that touched it (`git log -1 --format=%h -- <path>`). Say "uncommitted changes" if
   `git status --porcelain -- <path>` shows any.
6. Upstream artifacts: `docs/intent.md` and every earlier artifact in the chain for the same phase
   (see CLAUDE.md), excluding the artifact under review.

## 2. Round one: independent reviews

Start all five members in parallel, in a single message, and wait for all five:
`council-architect`, `council-security`, `council-simplifier`, `council-hiring-manager`,
`council-contrarian`. Give each exactly this brief, with the placeholders filled in and nothing added:

> Review the artifact at `<path>` (<revision>). Upstream artifacts: <list>. Read them yourself.
> Work alone; other members are reviewing in parallel and you will not see their work yet.
> Return only this, in at most 450 words:
>
> **Verdict**: Accept, Accept with changes, or Reject. One sentence of reason.
>
> **Top five issues, ranked by impact** (fewer if you find fewer). For each:
> title; why it matters; where it is in the artifact; the change you propose; evidence (a URL you
> opened in this run, or a file and section).
>
> **Evidence links**: every URL you relied on, one per line.
>
> **Owner questions**: at most three, only where a decision depends on the owner's answer. Give
> your proposed default in square brackets.

If a member fails or returns nothing, run that member once more. If it fails again, continue
without it and tell the chair which member is missing.

## 3. Anonymise

Remove anything that names a role or lens from each review. Label the reviews Review A to Review E
in an order unrelated to the member list above. Keep the label-to-member mapping to yourself until
step 5.

## 4. Round two: one anonymised rebuttal

Start the five members again in parallel, in a single message, and wait for all five. Each member
receives its own round-one review and the other four anonymised reviews, with this brief:

> Rebuttal round for `<path>`. Below is your own review, then four reviews by other members,
> identities removed. Return only this, in at most 300 words:
>
> **Final verdict**: Accept, Accept with changes, or Reject. Say whether it changed and why.
>
> **Accepted**: points from other reviews that you now agree with, by review label.
>
> **Disputed**: points from other reviews that you reject, by review label, with your reason and
> evidence.
>
> **Top three issues overall**: across all five reviews, ranked by impact.
>
> **Missed by everyone**: at most one item, or "nothing".
>
> Your review: <the member's round-one review>
>
> Review <label>: <text> (repeated for the other four)

There is exactly one rebuttal round. Do not run another.

## 5. Chair

Start `council-chair` with: the artifact path and revision, the output path, today's date, the
label-to-member mapping, the five reviews and the five rebuttals in full and unedited, and the name
of any missing member. The chair writes the output file. The chair never adds scope.

## 6. Report

Check that the output file exists and contains the sections Verdict, Debates and resolutions,
Dissent and Owner questions. Then tell the owner: the verdict, the output path, each dissent in one
line, and the owner questions as a numbered list.

Do not edit the reviewed artifact. Changes to it are the owner's decision and belong to a later step.
