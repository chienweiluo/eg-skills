# Design critic prompt (fresh goldfish — send exactly this, slots filled)

Do not add the implementer's reasoning, the conversation, or why the story exists beyond what is below. The asymmetry is the point.

```
Critique a design note for one story of a feature, in the repo at <repo root>. You may read any file in the repo.

Story <id>: <title>
<description>
Acceptance criteria:
<numbered list>

Source PRD section (<prdPath>, "<prdSection>"):
<paste the section verbatim>

Design note: <state dir>/designs/<id>.md

Check, with file:line evidence from the repo where relevant:
1. Does the design satisfy EVERY acceptance criterion? Name any criterion it does not cover.
2. Does it contradict the PRD, the repo's CLAUDE.md / AGENTS.md, or existing patterns?
3. Missing edge cases, error paths, auth / tenant scoping, migration or backwards-compatibility concerns.
4. Scope: does it do work outside this story, or rely on work no earlier story delivered?
5. Assumptions it relies on that the PRD does not state.

Classify each gap as TECHNICAL (the implementer can resolve it from the code) or PRODUCT (needs a human decision about behaviour or scope).

Output a numbered list: `[TECHNICAL|PRODUCT] gap — why it matters — suggested resolution`.
End with the literal string `design ready` if (and only if) there are no gaps that would cause a criterion to fail.
```
