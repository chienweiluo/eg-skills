# eg-skills

Claude Code skills for an **elephant/goldfish** engineering workflow: the session that knows everything does the work, and fresh subagents that know almost nothing check it.

- **Elephant**: your working session. It has the repo, CLAUDE.md and the whole conversation, so it is productive, but it is also anchored on its own first idea.
- **Goldfish**: a fresh subagent that gets one artifact (a design doc, a symptom, a diff) and no story about why. It can't be talked into agreeing, because nobody told it what to agree with.

The idea comes from [*Elephants, Goldfish, and the New Golden Age of Software Engineering*](https://drensin.medium.com/elephants-goldfish-and-the-new-golden-age-of-software-engineering-c33641a48874). The no-code gate and the three-pass design check in `eg-new-feature` follow that article. The rest is how I've adapted the model to day-to-day work. See [What this adds](#what-this-adds-to-the-original-model).

## The pipeline

```
 rough idea                                                     bug report
     │                                                              │
     ▼                                                              ▼
/eg-brainstorm ──► /eg-prd ──► /eg-new-feature ─┐            /eg-fix-bug ─┐
 (optional)        (optional)  one feature       │                         │
                       │                         ▼                         ▼
                       └──► /eg-loop ──► /eg-precommit-review ◄────────────┘
                            many stories      (inside every path)
                            (experimental)          │
                                                    ▼
                                         you say "commit"
```

Start wherever the work is. A clear, small feature goes straight to `/eg-new-feature`. A bug goes to `/eg-fix-bug`. `/eg-precommit-review` also works on its own for any hand-written diff.

| Skill | Use it for | Skip it when |
|---|---|---|
| `/eg-brainstorm <idea>` | A half-formed idea. 3–5 goldfish, each with one lens (technical, business, UX, contrarian, prior art), produce a concepts brief. | You already know what to build. |
| `/eg-prd <idea>` | A feature that needs requirements pinned down: codebase grounding → gap Q&A → parallel research goldfish → PRD in `docs/prds/`. | A one-component change. |
| `/eg-new-feature <desc \| #issue>` | One feature, end to end: scope → design doc → goldfish design check → implement → review → test gate. | Pure design discussion. |
| `/eg-fix-bug <desc \| #issue>` | Any code bug, including one-liners: problem doc → goldfish diagnosis → failing test → fix → review. | Typos, version bumps, formatter-only diffs (the skill skips its own ceremony for these). |
| `/eg-precommit-review [focus]` | An independent review loop on whatever is pending. | Docs-only changes. |
| `/eg-loop <prd>` *(experimental)* | A PRD too big for one session: one fresh implementer per story, one commit per passing story, on its own branch. | Anything `/eg-new-feature` can finish in one go. |

## What a goldfish sees

The asymmetry is deliberate, and every prompt template is passed **verbatim**. The elephant is not allowed to add "context" that would leak its intent.

| Stage | Goldfish gets | Goldfish does **not** get |
|---|---|---|
| Brainstorm / PRD research | The seed and one assigned lens | The other lenses, the elephant's preferred answer |
| Design check (comprehension, critic, readiness) | The design doc, read access to the repo | Chat history, implementation intent, the other passes' output |
| Bug diagnosis | Symptom and repro | The elephant's hypothesised root cause |
| Pre-commit review | `git diff` / `git status` / `git log` | The plan, the request, what the change was "supposed" to do |
| `eg-loop` design critic | One story, its PRD section, the design note | The implementer's reasoning |

## What this adds to the original model

1. **Goldfish in both directions.** The original uses a goldfish to *converge*, stress-testing one plan. Here goldfish also *diverge*: `eg-brainstorm` and `eg-prd` run several in parallel, each pushed hard down one lens, and the elephant only synthesizes.
2. **A goldfish at every stage, not just design.** Bugs get an independent diagnosis before the fix, so the first hypothesis can't anchor everything. Diffs get an independent reviewer before every commit.
3. **Rebuttals are first-class.** Every finding is either fixed (with file:line) or rebutted with a verifiable reason: visible in code, out of scope, documented trade-off, or the user's quoted words. A ledger carries findings across rounds, and the final report shows every rebuttal **verbatim**. Nothing is dismissed silently.
4. **Hard caps hand control back.** The design check stops after 3 revisions and the review stops after 5 rounds (or when a new finding lands on a line already fixed). At a cap the skill asks you what to do. It never accepts on your behalf.
5. **Interactive gates survive "be autonomous".** Questions are enumerated `AskUserQuestion` choices, and they still run when a system reminder says "don't stop to ask". Invoking the skill is the opt-in. Say "skip the framing questions and use defaults" in the same turn to skip the optional ones. Each skill lists which gates can't be skipped.
6. **Failing test before fix.** `eg-fix-bug` captures the bug as a test that fails first. Existing tests are never rewritten to pass without the reviewer signing off on the new behaviour.
7. **"Done" means driven.** Tests check the code, not the feature. UI work is walked through in a real browser (Claude in Chrome MCP) or simulator: the golden path, a plausible edge case, and a different user/tenant where that applies.
8. **Stack-agnostic.** Each run builds a project profile (lint, type-check, test, E2E, codegen, dev server, commit style) from CLAUDE.md / AGENTS.md, the manifests and CI. If it can't find a command, it says so instead of inventing one.
9. **Stops before commit.** Every skill ends with a report and waits for your "commit". `eg-loop` is the only exception: you approve the story plan once, and that allows commits on its own `eg-loop/<slug>` branch only. It never pushes, merges or rebases.

## Install

Requires [Claude Code](https://claude.com/claude-code). The skills depend on its `Agent`, `SendMessage` and `AskUserQuestion` tools, so they won't run as-is in other agents. Optional: the Claude in Chrome extension for UI verification and logged-in research, and `gh` for `#issue` arguments.

```sh
# as user-level skills: invoked as /eg-new-feature etc.
npx skills add chienweiluo/eg-skills -g -a claude-code -y
npx skills update -g -y        # update later
```

Or as a Claude Code plugin. Skills are then namespaced, e.g. `/eg-skills:eg-new-feature`:

```
/plugin marketplace add chienweiluo/eg-skills
/plugin install eg-skills@eg-skills
```

**Per-repo overrides:** a project-level `.claude/skills/eg-<name>/` (or `.claude/commands/eg-<name>.md`) takes precedence. Use one to pin a repo's exact test commands or add stack-specific review rules.

## Cost

Goldfish are full subagents. One `/eg-new-feature` run is typically 3 design passes on the first round, 2 on each later round, plus 2–3 review rounds. `/eg-brainstorm` and `/eg-prd` start 3–5 in parallel. `/eg-loop` does all of that per story. The extra tokens buy fewer bad designs and fewer PR review cycles. For a typo, skip the skills.

## Relation to other skill packs

[superpowers](https://github.com/obra/superpowers) covers similar ground (brainstorming, plans, subagent-driven development, code review) and is broader. eg-skills is narrower and more opinionated: strict context asymmetry, verbatim rebuttal ledgers, hard caps, and mandatory user gates. The `eg-loop` `prd.json` format is compatible with [ralph](https://github.com/snarktank/ralph).

## License

[MIT](LICENSE)
