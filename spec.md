# Spec — AI Commit Review Plugin

Status: DRAFT v0.1. Items marked `[DECISION]` will be finalized by a human. Numeric values are starting proposals, to be updated with measurements.

## 1. Goal
Automatically review changed code against team standards and error risk before commit, giving the developer fast feedback. Aim: reduce repetitive fixes raised in post-commit code review.

## 2. Phases
| Phase | Content | Output |
|---|---|---|
| 0 | Hello spike: show "Merhaba" when the commit button is pressed | Working empty plugin, learned IntelliJ structure |
| 1 | Copilot access spike: make one AI request from inside the plugin | Feasibility decision (answers to open questions below) |
| 2 | MVP: collect diff, ask AI together with standards, show findings | Version usable by a pilot group |
| 3 | Improvement: cache, budget limits, settings, local rules | Rollout to the whole team |

Only **Phase 0** is active now.

## 3. Phase 0 — Requirements
- R0.1: The plugin loads in the sandbox IDE (`./gradlew runIde`) without problems.
- R0.2: When the commit button is pressed in the commit dialog/panel, the user sees a "Merhaba" message (keep this exact Turkish text).
- R0.3: After the message, the commit proceeds normally (the commit is not blocked).
- R0.4: The message is shown in a way that does not block the UI. Prefer IntelliJ's current recommended commit-check mechanism. Verify from the official docs which one is current/recommended; this is a `[DECISION]` item.
- R0.5: `./gradlew build` and `./gradlew verifyPlugin` pass without errors.

### Phase 0 acceptance criteria
- [ ] In the `runIde` IDE, pressing commit in a Git project shows the message
- [ ] The commit can complete
- [ ] Build + verifyPlugin are green
- [ ] The IDE does not freeze
- [ ] Building with JDK 21 is verified (the assumption in AGENTS.md is confirmed)

## 4. Final product requirements (draft, Phases 2-3)
### Input
- Only the changes to be committed (staged/selected).
- Coding standards: an md file in the project. Location and format are `[DECISION]` (proposal: `.review/standards.md` in the project root).

### Output
Each finding: `severity` (`CRITICAL` = must fix, `SUGGESTION` = could improve), `file`, `line`, `message`, `rule` (the standards rule title, if any). The AI is asked to return JSON; unparseable output is ignored (fail-open).

### Display
- `CRITICAL` findings are shown as prominent (important) warnings; `SUGGESTION` findings as less prominent warnings.
- If there is a `CRITICAL` finding, the commit is not blocked; the user is offered "commit anyway / fix first". `[DECISION]`

### Performance budgets (starting proposal)
- IDE freeze: 0 (nothing runs on the EDT).
- Total AI analysis time limit: 30 s; on exceed, cancel + fail-open. `[DECISION]`
- Local (non-AI) pre-checks: under 1 s.

### Token budgets (starting proposal, to be updated with measurements)
- Max diff sent per request: ~400 changed lines; above that, tell the user "too large, skipped". `[DECISION]`
- From the standards md, only rules relevant to the changed file types are sent.
- A max output token limit is applied.
- A per-user daily request limit is configurable. `[DECISION]`
- The same diff (by hash) is not asked twice (cache).

## 5. Out of scope (for now)
- Automatic fixing/changing code (warnings only).
- Running outside the IDE (CI).
- Publishing to the Marketplace.
- Multiple AI providers.

## 6. Open questions (to be answered in Phase 1)
1. How will the plugin access Copilot? Java SDK or another way? Does the SDK require the Copilot CLI to be installed on the machine?
2. Does the company's Copilot policy allow SDK/CLI usage? Is admin approval needed?
3. Are credits per user or pooled? What is the approximate cost per review request?
4. Which models can be selected? Is quality sufficient with a cheaper model?
5. Authentication: will the user sign in with their own GitHub account?
