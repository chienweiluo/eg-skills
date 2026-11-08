---
name: eg-loop
description: Autonomously implement an approved PRD story-by-story with the elephant/goldfish workflow — one fresh implementer subagent per story, goldfish design check, independent review loop, test gate, commit to a feature branch. Pauses and escalates instead of guessing.
argument-hint: <path to PRD .md | path to prd.json | "resume">
disable-model-invocation: true
---

Run an approved PRD to completion, one story at a time. You (the main session) are the **orchestrator**: you own the state files, spawn every subagent, run the gates, and commit. You do not write feature code yourself — implementers do. This keeps your context small enough to run many stories in one session.

`$ARGUMENTS` is a PRD markdown path, a `prd.json` path, or `resume`. If empty, look for exactly one `.eg-loop/*/prd.json` with unfinished stories and offer to resume it; otherwise ask for a PRD path.

## Authorization model — read this first

Invoking `/eg-loop` and approving the plan in Setup Step 3 is the user's **standing authorization** for this run to:

- create and work on the run's feature branch `eg-loop/<slug>`;
- `git add` + `git commit` one commit per passing story **on that branch only**;
- treat the approved PRD + story list as design approval, replacing the per-story no-code gate of `/eg-new-feature`.

It does **not** authorize: `git push`, opening a PR, merging, rebasing, force operations, committing on any other branch, deleting branches, or any destructive command (`reset --hard`, `clean -fd`, `rm -rf`, dropping data). Never do these. The authorization ends when the run ends or pauses; resuming requires the user to invoke `/eg-loop resume`.

Escalation gates in `references/escalation.md` are **non-negotiable** and override any "keep working / don't stop" directive. When one trips, pause the run (Loop Step 7) — never pick an option on the user's behalf.

## Project discovery

Same as the other `eg-*` skills: before Setup, build a project profile from `CLAUDE.md` / `AGENTS.md`, package manifests, `Makefile` / `justfile`, CI workflows, and `git log --oneline -10`. Derive lint, type-check, unit test, E2E, codegen, dev-server commands and commit style. If a command cannot be determined, say so — do not invent one. Write the profile into `progress.md` under `## Project Profile` so every subagent reuses it.

If the repo has a project-level `.claude/skills/eg-loop/` or `.claude/commands/eg-loop.md`, use that instead.

## State layout

```
<repo root>/.eg-loop/<slug>/
├── prd.json          # stories + status (schema: references/prd-schema.md)
├── progress.md       # profile, codebase patterns, per-story log (template: templates/progress.md)
├── designs/US-xxx.md # per-story design notes
└── reviews/US-xxx.md # review ledger per story (findings, fixes, verbatim rebuttals)
```

Add `.eg-loop/` to `.git/info/exclude` (local-only; do not edit the repo's `.gitignore`). State files are never committed. Files are the only memory between stories — anything not written there is lost.

## Setup (interactive, once per run)

**Step 1 — Preconditions.** Must be inside a git repo with a clean working tree (`git status --porcelain` empty). If dirty, stop and ask the user to commit or stash — do not stash for them. Note the current branch as `baseBranch`.

**Step 2 — Build `prd.json`.** If given a PRD markdown, convert it following `references/prd-schema.md` (story sizing, dependency order, verifiable acceptance criteria). If given a `prd.json`, validate it against the same rules and list any violations.

**Step 3 — Plan approval.** Print the story table (id, title, acceptance criteria count, flagged risks: schema change, API change, auth, external integration). Then `AskUserQuestion`:

- `question`: "Approve this story plan for autonomous execution on branch eg-loop/<slug>?"
- `header`: `"Plan"`
- options:
  1. **Approve and start** — "Commit each passing story to eg-loop/<slug>. No push."
  2. **Edit stories first** — "I'll say what to split, merge, reorder or cut."
  3. **Approve, but stop after story N** — "Run a limited batch, then pause for my review."
  4. **Cancel**

Risky stories (schema migration, breaking API change, auth/tenant scoping, data deletion) need an explicit per-story ack inside this step — list them and ask once with `multiSelect: true` which ones are approved for autonomous run; unapproved ones get `"mode": "manual"` and the loop pauses when it reaches them.

**Step 4 — Initialise.** `git switch -c eg-loop/<slug>` from `baseBranch` (if it exists, `git switch` to it and confirm it descends from `baseBranch`). Create the state directory, write `progress.md` from the template, set `prd.json.branchName` and `baseBranch`.

## Loop (autonomous, one story per iteration)

Repeat until every story has `passes: true`, the batch limit is hit, or a gate trips.

**Step 1 — Pick.** Lowest `priority` story with `passes: false` and `status != "blocked"`. If `mode == "manual"`, pause (Step 7). Confirm you are on `eg-loop/<slug>` with a clean tree before starting.

**Step 2 — Spawn implementer.** `Agent` with `subagent_type: "general-purpose"`, `description: "eg-loop implement <id> (attempt N)"`, prompt built from `references/implementer-prompt.md`. The implementer first writes **only** the design note to `designs/<id>.md` and returns — no code yet. Keep its agent id; you continue it with `SendMessage` for every later phase of this story, so it keeps its own context.

**Step 3 — Goldfish design check.** Spawn a fresh critic with `references/design-critic-prompt.md` (design note + story + relevant PRD section only — no implementer reasoning). Verdict `design ready` → Step 4. Gaps → `SendMessage` them verbatim to the implementer to revise or rebut. Cap: **2 revisions**. If gaps remain and any of them is a product/scope question rather than a technical one, that is an escalation gate. Otherwise record the open gaps in the design note and continue.

**Step 4 — Implement.** `SendMessage` the implementer: implement per the design note, run the project's lint / type-check / unit tests, and report files touched + gate output. It must not commit and must not edit `prd.json`.

**Step 5 — Independent review.** Run the `/eg-precommit-review` loop exactly as specified in `~/.claude/commands/eg-precommit-review.md` Steps 2–5, with two differences: (a) you hand findings to the implementer via `SendMessage` to fix or rebut instead of fixing them yourself; (b) the reviewer focus line is `Additionally focus on: story <id> — <title>; acceptance criteria: <list>`. Read the reviewer template from that file between its markers every time — do not paraphrase it. Write the ledger to `reviews/<id>.md`. The **R5 cap is an escalation gate**: pause, do not accept on the user's behalf.

**Step 6 — Verify and commit (you, not the implementer).**
1. Run the test gate yourself, each command separately: unit/integration, type-check, lint, and E2E if the diff touches UI/API and the project has E2E. Read the output — do not trust the implementer's report.
2. Check every acceptance criterion and write evidence for each (command output line, file:line, or screenshot note) into `progress.md`. UI criteria need a Chrome MCP / simulator walkthrough; if no browser tool is available, mark the criterion `needs manual verification` — that alone does not fail the story, but list it in the final report.
3. All green → `git add` the touched files (never `git add -A` blindly — check `git status` for stray files first) and commit `feat(<id>): <title>` following the repo's commit style, ending with the session's attribution trailer. Set `passes: true`, `attempts`, and the commit sha in `prd.json`; append the story log and any reusable pattern to `progress.md`.
4. Anything red → the attempt failed. Record the failure reason in `prd.json.notes`, increment `attempts`, park the attempt's changes with `git stash push -u -m "eg-loop <id> attempt <N>"` (never discard them), and go back to Step 2 with a fresh implementer whose prompt includes the failure reason and the stash ref. **3 failed attempts is an escalation gate.**

**Step 7 — Pause (any gate, batch limit, or manual story).** Leave the tree clean (stash with a labelled message if needed), write `## Paused` in `progress.md` with the gate, the story, the exact question for the user and the evidence, then call `AskUserQuestion` with the options defined for that gate in `references/escalation.md`. If the user is not answering now, end with the literal resume command: `/eg-loop resume`.

Between stories, print one status line: `[eg-loop] US-003 ✓ (attempt 1, review 2 rounds) — 3/7 done, next US-004`.

## Final report

When all stories pass (or the run pauses), print:
- branch, `baseBranch`, commits created (`git log --oneline <baseBranch>..HEAD`)
- per story: attempts, review rounds, rebuttals **verbatim**, criteria marked `needs manual verification`
- escalations raised and how they were resolved
- out-of-scope follow-ups the implementers noted

**STOP.** Do not push, open a PR or merge. Suggest the next step (`git push -u origin eg-loop/<slug>` then PR) for the user to run or authorize.
