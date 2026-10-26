# Phase 2 — MVP: real review in the commit flow

Status: DRAFT v0.1. Items marked `[DECISION]` are finalized by a human before the related task starts. Read this file only when working on a Phase 2 task (T2.x). General rules stay in `AGENTS.md`, general requirements in `docs/spec.md`, architecture in `docs/architecture.md`, Phase 1 results in `docs/phase1-findings.md` (especially the "Phase 2 handoff" section).

## 1. Goal
Replace the fake findings with real ones. When the developer commits, the plugin collects the staged changes, asks Copilot (through the SDK and the local CLI server) to review them against the team's standards md, and shows the result with the existing balloon and "AI Reviewer" tool window. Pilot-ready for 2-3 developers.

## 2. Decisions to finalize (proposals included)
| # | Topic | Proposal | State |
|---|---|---|---|
| D1 | Standards source | One standards md file per project. Path is a setting; default `.review/standards.md` in the project root; a different path (for example the team's existing main AI-instructions md) can be configured. MVP sends the whole file up to a size cap (proposal: 12,000 characters); if it is larger, the review is skipped with a clear notice. Selecting sections per file type is Phase 3. | `[DECISION]` |
| D2 | Review timing and blocking | The review runs as a pre-commit check with a cancellable progress indicator and a 30 s limit. If there are warnings (`CRITICAL`), the user chooses "Review" (opens the tool window) or "Commit anyway". If there are only suggestions, the commit continues and the balloon shows them. Any error or timeout: commit continues (fail-open). | `[DECISION]` |
| D3 | File scope | Only `.java` files for the MVP. Other file types are skipped. | `[DECISION]` |
| D4 | Language of findings | Short messages in Turkish (the team reads them); configurable later. | `[DECISION]` |
| D5 | Default model while building | A cheap model for development (`claude-haiku-5.5`); the real default is chosen in T2.8 by quality tests. | `[DECISION]` |
| D6 | CLI server | Still started by the developer (`copilot --headless --port 4321`); the plugin only connects. One long-lived SDK client per IDE session, created lazily, reconnecting when needed. | accepted for the pilot |

## 3. Requirements
- R2.1: Real findings replace the fake findings provider. The balloon and tool window behave as in Phase 0.5.
- R2.2: Only the changes to be committed are reviewed. Binary, generated, lock files and files outside D3 are skipped. A diff above the size limit (proposal: 400 changed lines) is skipped with a notice ("too large, skipped"). `[DECISION]`
- R2.3: The standards md is read as in D1; a missing or oversized file gives a clear notice and no AI call.
- R2.4: Prompt layout: stable parts first (system message, output rules, standards), the diff last. The system message is minimal and asks for JSON only: `{"findings":[{"severity":"CRITICAL|SUGGESTION","file":"...","line":N,"rule":"...","message":"..."}]}`, at most 10 findings, messages under about 160 characters, line numbers refer to the new version of the file, and nothing is reported that is not in the diff.
- R2.5: The model's reply is parsed tolerantly (code fences, text around the JSON). Malformed findings are dropped; invalid severities, files not in the diff, and out-of-range lines are corrected or dropped; the count is capped. Unparseable output means "no findings" plus a short notice (fail-open).
- R2.6: Nothing blocks the UI thread. The whole review has a 30 s limit and can be cancelled; after a timeout or cancel the SDK turn is aborted and the session deleted.
- R2.7: The same diff plus the same standards plus the same model (by hash) is not reviewed twice; the previous findings are reused.
- R2.8: Credits per request are read from the usage event (`nanoAiu / 1e9`) and logged as numbers only. A monthly plugin budget is enforced (T2.9).
- R2.9: Logs contain counts, durations, credits and error kinds, never source code, prompts or replies.
- R2.10: Settings: enable/disable, CLI URL, standards path, model, reasoning effort, monthly credit budget.
- R2.11: Quality acceptance for the pilot: on a set of test diffs with known planted defects, the chosen model finds the agreed share of them (proposal: at least 70%) and at most one in five reported `CRITICAL` findings is a false alarm. `[DECISION]`

## 4. Tasks
Give the agent one task at a time. Use fake or dummy data in tests; real company code only in the pilot, and only by the human.

### T2.0 — Cleanup after the spike
- Agent: Follow the "SPIKE item" table in the Phase 2 handoff section of `docs/phase1-findings.md`: remove, replace or keep each item as marked. Keep the `ReviewClient` interface and the connection settings. Remove the decoy action and measurement action unless the table says keep. Bump the patch version.
- Acceptance: the build passes, tests pass, no `// SPIKE` code is left except what the table says to keep.

### T2.1 — Change collector and filter
- Agent: Collect the changes to be committed (use the IntelliJ VCS API; check `docs/reference/` and `javap`, do not guess). Produce a diff text per file with new-file line numbers, apply the filters of R2.2, and return a `ReviewInput` object (files, diff text, counts, skipped reasons). Runs on a background thread.
- Acceptance: unit tests with synthetic changes; skipped files and oversize diffs are reported with reasons.

### T2.2 — Standards loader
- Agent: Read the standards md as in D1, enforce the size cap, compute a content hash, return text or a clear problem. No AI call here.
- Acceptance: tests for present, missing, empty and oversized files.

### T2.3 — Prompt builder and output schema
- Agent: Build the system message and user message as in R2.4 (stable prefix first, diff last). Keep the output schema in one place. Provide a golden-file test of the prompt.
- Acceptance: the prompt for a sample input matches the golden file; the prefix is identical across different diffs.

### T2.4 — Real `ReviewClient`
- Agent: Evolve `CopilotReviewClient` per the `ai` section of `docs/architecture.md`: one long-lived client, lazy start, reconnect, one session per review (no tools, rejecting permission handler, isolation settings from Phase 1), model and effort from settings, usage event to credits, abort on timeout or cancel, session deleted afterwards. Map errors to the existing exception kinds.
- Acceptance: unit tests with a fake SDK layer; manual test by the human with the CLI server running.

### T2.5 — Output parser
- Agent: Implement R2.5 with tests (clean JSON, fenced JSON, text around JSON, malformed entries, wrong severity, unknown file, out-of-range line, too many findings, empty output).
- Acceptance: all cases pass.

### T2.6 — Orchestrator and wiring
- Agent: A `ReviewService` that runs collector, standards loader, cache lookup, prompt builder, client and parser on a cancellable background task with the 30 s limit, then publishes findings to the existing balloon and tool window. Connect it to the commit hook according to D2. Remove the fake findings provider. Fail-open on every error, with a short notice.
- Acceptance: human test in the IDE with a dummy project: warnings, suggestions only, no findings, server down, cancel, timeout. Spec R2.1, R2.6, R2.7.

### T2.7 — Settings page
- Agent: A settings page for R2.10 with sensible defaults and validation.
- Acceptance: settings persist and take effect without restarting the IDE where possible.

### T2.8 — Model quality and cost comparison (human + agent)
- Agent: Prepare a set of synthetic diffs with known planted defects (for example null dereference, resource leak, missing validation, swallowed exception, naming against the standards) and a small harness (dev-only action, marked `// SPIKE`) that runs them on candidate models and records: defects found, false alarms, output size, credits, latency.
- Human: Run it on three candidates (a small model, a mid-tier model, a Sonnet-class model) and decide the default model with the numbers. Not an Opus-class or higher model.
- Acceptance: a comparison table in `docs/phase2-results.md`; the default model recorded in the architecture decision log. Spec R2.11.

### T2.9 — Credit budget tracker
- Agent: Sum credits from usage events per developer per month (persisted locally), enforce the configurable monthly plugin budget (proposal: 5% of the monthly credits, about 375), skip reviews with a notice when reached, show used/remaining in settings.
- Acceptance: unit tests for month rollover and the limit; manual check in settings.

### T2.10 — Pilot preparation (human + agent)
- Agent: Write a short setup guide for teammates (install the plugin zip, `copilot login`, start the server, restart it after a logout, where the standards file goes, known limits) in `docs/setup.md`.
- Human: Pilot with 2-3 developers; collect feedback on false alarms and speed.
- Acceptance: the guide works for a developer who has never seen the project.

## 5. Order
1. T2.0 cleanup. 2. T2.1, T2.2, T2.3, T2.5 (independent, can use fake client data). 3. T2.4 real client. 4. T2.6 wiring. 5. T2.7 settings. 6. T2.8 model comparison. 7. T2.9 budget. 8. T2.10 pilot.

## 6. Example prompt to give the agent
> Read `AGENTS.md`, `docs/spec.md` and `docs/phase2.md` (sections 1 to 3 and the task you are doing). Do task **T2.0** in `docs/phase2.md`. Do not touch other tasks. Do not try to fetch web documentation. When done, run the build (with `COMMIT_REVIEWER_IDE` and `--offline` if needed); summarize changed files, the result, and open questions.
