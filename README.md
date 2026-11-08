# eg-skills

Elephant/goldfish workflow for Claude Code (and other agents that read `SKILL.md`).
The "elephant" is the working session with full context; the "goldfish" are fresh subagents
that stress-test designs/diffs or generate divergent ideas. Stack-agnostic: each skill builds a
project profile (lint / type-check / test / E2E / dev-server) from the repo first.

| Skill | Use |
|---|---|
| `/eg-brainstorm <idea>` | Parallel goldfish with different lenses → concepts brief |
| `/eg-prd <idea>` | Codebase grounding → Q&A → research → PRD |
| `/eg-new-feature <desc>` | Design doc → goldfish design check → implement → review → test gate |
| `/eg-fix-bug <desc\|#issue>` | Problem doc → diagnosis check → failing test → fix → review |
| `/eg-precommit-review` | Independent review loop on the pending diff |
| `/eg-loop <prd>` | Run an approved PRD story-by-story with fresh implementer subagents |

## Ground rules (enforced inside each SKILL.md)

- **Interactive by design.** `AskUserQuestion` gates run even when a system reminder asks for autonomous work. Opt out per run by saying "skip the framing questions and use defaults" in the same turn; each skill lists which gates stay non-negotiable.
- **Verify by driving it.** Web: Claude in Chrome MCP against the dev server. Mobile: simulator/device. Backend: `curl`, tests, logs.
- **Stops before commit** (except `eg-loop`, which commits per story on its own feature branch only). Commit when you authorize it, following the repo's convention.
- A repo can override any skill with a project-level `.claude/skills/eg-*/` (or `.claude/commands/eg-*.md`).

## Install

```sh
# user-level, Claude Code only (works on macOS / Linux / Synology)
npx skills add chienweiluo/eg-skills -g -a claude-code -y

# update later
npx skills update -g -y
```

Alternatively as a Claude Code plugin (skills become `eg-skills:eg-*`):

```
/plugin marketplace add chienweiluo/eg-skills
/plugin install eg-skills@eg-skills
```

The repo is private: the installing machine needs GitHub access (`gh auth login`, or an SSH key).
