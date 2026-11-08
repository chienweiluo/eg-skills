# Escalation gates

When any gate trips: finish no further code, pause per SKILL.md Loop Step 7, and ask with `AskUserQuestion`. These override any autonomous / no-stopping directive. Never answer them yourself.

| Gate | Trigger | `AskUserQuestion` options |
|---|---|---|
| **G1 Attempts** | A story failed 3 attempts (red gate or unmet criterion) | Retry with my guidance (chat) · Split the story · Mark blocked and continue with stories that don't depend on it · Stop the run |
| **G2 R5 cap** | `/eg-precommit-review` loop hit 5 rounds, or a new finding landed at a previously-fixed spot | Accept story as-is and commit · Keep iterating · Discard attempt (stash it) and retry fresh · Stop the run |
| **G3 Product gap** | Design critic raised a PRODUCT gap still open after 2 revisions | Answer in chat, then continue · Drop the requirement · Mark story manual · Stop |
| **G4 Scope** | Story cannot be done without work outside it, or implementer finds the PRD is wrong / contradictory | Add a new story (I'll describe it) · Amend this story · Mark blocked · Stop |
| **G5 Risk not acked** | Work turns out to need a schema change, breaking API change, auth/tenant change, data deletion, new dependency, or external-service call that was not approved in Setup | Approve for this story · Mark manual · Stop |
| **G6 Test rewrite** | An existing test must change expected behaviour and the reviewer did NOT sign off on the new behaviour | Approve the behaviour change · Treat as a bug, retry · Stop |
| **G7 Environment** | A gate can't run (missing service, credentials, flaky infra failing twice the same way) | I'll fix the environment, then resume · Skip that gate for this story (recorded) · Stop |
| **G8 Branch drift** | Not on `eg-loop/<slug>`, unexpected commits on the branch, or the tree is dirty with changes no implementer made | Always stop and show `git status` + `git log`; ask how to proceed |
| **Batch limit** | Reached `batchLimit` story | Continue to next batch (give new limit) · Stop here |

For every pause, `progress.md` `## Paused` must contain: gate id, story id, the one-sentence question, evidence (command output tail, review ledger path, design note path), and what the loop will do for each option.
