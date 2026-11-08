# prd.json schema and story rules

Compatible with the `ralph-skills:ralph` format, plus eg-loop run fields.

```json
{
  "project": "TaskApp",
  "slug": "task-status",
  "branchName": "eg-loop/task-status",
  "baseBranch": "main",
  "prdPath": "docs/prds/task-status.md",
  "description": "Track task progress with status indicators",
  "batchLimit": null,
  "userStories": [
    {
      "id": "US-001",
      "title": "Add status field to tasks table",
      "description": "As a developer, I need to store task status in the database.",
      "prdSection": "## Requirements > Persistence",
      "acceptanceCriteria": [
        "tasks.status column: 'pending' | 'in_progress' | 'done', default 'pending'",
        "Migration runs and rolls back cleanly",
        "Type-check passes",
        "Tests pass"
      ],
      "risk": ["schema"],
      "mode": "auto",
      "priority": 1,
      "passes": false,
      "status": "todo",
      "attempts": 0,
      "commit": null,
      "notes": ""
    }
  ]
}
```

Field notes:
- `risk`: any of `schema`, `api-breaking`, `auth`, `tenant`, `external`, `data-deletion`. Non-empty → needs explicit ack in Setup Step 3.
- `mode`: `auto` or `manual` (loop pauses on reaching it).
- `status`: `todo` | `in_progress` | `done` | `blocked`.
- `batchLimit`: story id to stop after, or `null`.

## Story rules

1. **One story = one implementer context.** If you cannot describe it in 2–3 sentences, split it.
2. **Dependency order:** schema/types → domain logic → auth rules → services → API → UI leaf → UI composite → routing → jobs → E2E. No story may depend on a later one.
3. **Verifiable criteria only.** "Filter dropdown has All | Active | Done" — yes. "Works well", "handles edge cases" — no; rewrite or ask.
4. Every story ends with the project's real gates as criteria (`Type-check passes` only if the project has one; `Tests pass`). UI stories add `Verified in browser (golden path + one edge case)`.
5. Each story cites the PRD section it came from (`prdSection`) so the critic can check it against source.
6. Anything in the PRD's out-of-scope / open-questions section is **not** turned into a story. Open questions that block a story → mark that story `mode: manual`.
