# Spec — AI Commit Review Plugin

Status: DRAFT v0.5. Items marked `[DECISION]` will be finalized by a human. Numeric values are starting proposals, to be updated with measurements.

User-facing plugin name: **"AI Reviewer"**.

## 1. Goal
Automatically review changed code against team standards and error risk before commit, giving the developer fast feedback. Aim: reduce repetitive fixes raised in post-commit code review.

## 2. Phases
| Phase | Content | Output |
|---|---|---|
| 0 | Hello spike: show "Merhaba" when the commit button is pressed | Working empty plugin, learned IntelliJ structure. **DONE** |
| 0.5 | Findings UX: balloon with warning/suggestion counts + "Review code analysis" link, details in a bottom tool window (fake data) | Readable findings UX, no AI yet. **DONE** |
| 1 | Copilot access spike: make one AI request from inside the plugin | Feasibility decision (answers to open questions below) |
| 2 | MVP: collect diff, ask AI together with standards, show findings | Version usable by a pilot group |
| 3 | Improvement: cache, budget limits, settings, local rules | Rollout to the whole team |

Only **Phase 1** is active now.

## 3. Phase 0 — Requirements (DONE)
- R0.1: The plugin loads in the sandbox IDE (`./gradlew runIde`) without problems.
- R0.2: When the commit button is pressed in the commit dialog/panel, the user sees a "Merhaba" message (keep this exact Turkish text). (Superseded by R0.5.11.)
- R0.3: After the message, the commit proceeds normally (the commit is not blocked).
- R0.4: The message is shown in a way that does not block the UI. Prefer IntelliJ's current recommended commit-check mechanism.
- R0.5: `./gradlew build` and `./gradlew verifyPlugin` pass without errors.

### Phase 0 acceptance criteria
- [x] In the `runIde` IDE, pressing commit in a Git project shows the message
- [x] The commit can complete
- [x] Build + verifyPlugin are green
- [x] The IDE does not freeze
- [x] Building with JDK 21 is verified

## 3b. Phase 0.5 — Findings notification and tool window (DONE)

### Goal
Make findings work like the IDE's own Problems notifications: after a commit attempt, a balloon in the bottom-right corner shows the warning and suggestion counts, with a clickable "Review code analysis" line under it. Clicking that line opens a dedicated "AI Reviewer" tool window at the bottom of the IDE (like the Build or Problems tabs) that lists the finding details. Use hardcoded fake findings for now; the AI is not involved yet.

### Notification layout
```
AI Reviewer
2 warnings, 3 suggestions
Review code analysis        <- clickable link
```
- Line 1 (title): "AI Reviewer".
- Line 2: the number of warnings and the number of suggestions, in exactly this format: "<N> warnings, <M> suggestions". Warnings are findings with severity `CRITICAL`; suggestions are findings with severity `SUGGESTION`.
- Line 3: the text "Review code analysis" as a clickable link (or notification action). Use exactly this text.

### Requirements
- R0.5.1: A tool window named "AI Reviewer" is registered, anchored at the bottom of the IDE, with its own icon.
- R0.5.2: The tool window lists findings in two groups with these user-facing labels: "Warnings" (`CRITICAL`) and "Suggestions" (`SUGGESTION`). Each group header shows its count, for example "Warnings (2)". Warnings are visually more prominent (icon and color) than suggestions.
- R0.5.3: Each finding row shows: severity icon, message, file name, and line number.
- R0.5.4: Double-clicking a finding opens the file in the editor at that line.
- R0.5.5: When the user presses commit, the fake findings (see R0.5.12) are loaded and a balloon notification with the layout above appears in the bottom-right corner. The tool window is NOT opened automatically. The commit is not blocked (fail-open).
- R0.5.6: If there are no findings, no balloon is shown. If the user opens the tool window manually, it shows a short empty-state text ("No issues found"). Modal dialogs are never used for findings.
- R0.5.7: The UI follows the IDE theme (light/dark) and uses standard IntelliJ UI components.
- R0.5.8: The findings data model is independent of the UI, so it can later be filled by the real AI result without UI changes.
- R0.5.9: Clicking "Review code analysis" in the balloon opens and activates the "AI Reviewer" tool window, which lists the finding details.
- R0.5.10: The balloon uses a dedicated notification group named "AI Reviewer" registered in `plugin.xml`, with warning-level styling.
- R0.5.11: The old "Merhaba" message must be completely removed (code, resources, and any notification or dialog). Pressing commit must not show it anymore.
- R0.5.12: For now, the balloon and the tool window both use the same hardcoded fake findings, provided from one single place (for example a `FakeFindings` provider class) so it can be replaced by real results in Phase 2 without UI changes. The fake list contains exactly 2 warnings and 3 suggestions. Where possible, fake findings point to existing files of the open project, so double-click navigation can be tested. Fake line numbers may exceed a file's length; the editor then goes to the last line, which is acceptable for now.
- R0.5.13: The canonical name "AI Reviewer" is used consistently everywhere (tool window id/title, notification group id/display name, balloon title, code, resources, `plugin.xml`, docs).
- R0.5.14: The Gradle build targets IntelliJ IDEA 2025.3 (unified distribution, build 253) by default, and can also build against a locally installed IDE via `COMMIT_REVIEWER_IDE` when downloads are blocked.

### Acceptance criteria
- [x] Pressing commit no longer shows "Merhaba"
- [x] After pressing commit, a balloon appears in the bottom-right corner with "2 warnings, 3 suggestions" and the "Review code analysis" link; the tool window does not open by itself
- [x] Clicking "Review code analysis" opens the "AI Reviewer" tool window, which shows "Warnings (2)" and "Suggestions (3)" with the fake findings
- [x] The balloon and the tool window show consistent numbers (same fake list)
- [x] Icons are correct; works in light and dark themes
- [x] Double-click on a finding navigates to the correct file and line
- [x] The commit still completes and the IDE does not freeze
- [x] The name "AI Reviewer" is used everywhere; no leftover "AI Review" (without "er")
- [x] `./gradlew build` passes; `./gradlew verifyPlugin` passes when the network allows it

## 3c. Phase 1 — Copilot access spike

### Goal
Find out, with the least possible effort and cost, whether the plugin can call GitHub Copilot from inside the IDE using the company's Copilot account, and what a request costs. Output: a go/no-go decision and written answers to section 6.

### Scope and constraints
- Sources: the Copilot Java SDK documentation in `docs/reference/` (the human pasted it; do not browse the web).
- The spike code is dev-only: it is NOT connected to the commit flow and the findings UI.
- Only dummy/example code is used in prompts. No real company code, and no credentials or tokens in the repo, logs, prompts, or docs.
- Spike-only code is marked with a `// SPIKE` comment so it can be found and removed or replaced later.

### Requirements
- R1.1: Written answers to the section 6 questions 1-5 in `docs/phase1-findings.md`. Each answer is marked: `confirmed` (tested), `from docs` (not tested), or `unknown`.
- R1.2: A single request works from a background thread inside the plugin (sandbox IDE), triggered by a dev-only action, and the result is shown without blocking the UI.
- R1.3: The request is single-shot with a short prompt; tools/agent behavior are disabled and an output limit is set, where the SDK allows it. If the SDK cannot do this, the finding is documented.
- R1.4: Measurements for three prompt sizes (small, medium, large diff): latency and, if the SDK or the account exposes it, token usage and credit cost per request.
- R1.5: Failure modes are checked and each gives a clear message, never a freeze: CLI missing, not signed in, no credits/quota, timeout, user cancel. (These must be fail-open when connected to commits later.)
- R1.6: The spike code sits behind a small `ReviewClient`-style interface, or can easily be moved behind it in Phase 2.
- R1.7: Dependency check: adding the SDK does not break the build or the plugin packaging, and conflicts with libraries bundled in the IDE are checked and reported.
- R1.8: Token hygiene for every spike request: no tools available; the CLI must not load project or user custom instructions or other extra context (use an empty temporary working directory and/or the SDK's option to skip custom instructions); the system message is minimal (replace mode, if available); reasoning effort is set to the lowest useful level, if available. Document which of these the SDK supports.
- R1.9: Record in `docs/phase1-findings.md` whether the SDK is officially supported by GitHub or community-driven, and what the company policy says about using it.

### Go / no-go criteria
Go only if all are true:
1. A request works from inside the plugin on a background thread.
2. It works with the company account under the company's policy (confirmed with the Copilot admin).
3. Single-shot use without tools is possible.
4. The estimated credit cost per review is acceptable. `[DECISION]`
5. The setup needed on each teammate's machine (installs, sign-in) is acceptable. `[DECISION]`

If no-go: document why and propose alternatives; the human decides.

### Acceptance criteria
- [ ] `docs/phase1-findings.md` answers every question (or marks it unknown)
- [ ] A dev-only action sends one request from the sandbox IDE and shows the response
- [ ] The IDE never freezes during the request, and cancel/timeout work
- [ ] Failure modes (R1.5) each show a clear message
- [ ] Measurement table filled for three prompt sizes
- [ ] Dependency check done and reported
- [ ] Token hygiene (R1.8) applied and its SDK support documented
- [ ] SDK support status (R1.9) recorded
- [ ] Go/no-go decision recorded in `docs/architecture.md` decision log

## 4. Final product requirements (draft, Phases 2-3)
### Input
- Only the changes to be committed (staged/selected).
- Coding standards: an md file in the project. Location and format are `[DECISION]` (proposal: `.review/standards.md` in the project root).

### Output
Each finding: `severity` (`CRITICAL` = must fix, `SUGGESTION` = could improve), `file`, `line`, `message`, `rule` (the standards rule title, if any). The AI is asked to return JSON; unparseable output is ignored (fail-open).

### Display
- A balloon notification shows the warning and suggestion counts with a "Review code analysis" link. Clicking the link opens the "AI Reviewer" tool window, which lists the details. Warnings (`CRITICAL`) are shown as prominent (important) findings; suggestions as less prominent ones.
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
- Inline editor highlighting of findings.
- Running outside the IDE (CI).
- Publishing to the Marketplace.
- Multiple AI providers.

## 6. Open questions (to be answered in Phase 1)
1. How will the plugin access Copilot? Java SDK or another way? Does the SDK require the Copilot CLI to be installed on the machine?
2. Does the company's Copilot policy allow SDK/CLI usage? Is admin approval needed?
3. Are credits per user or pooled? What is the approximate cost per review request?
4. Which models can be selected? Is quality sufficient with a cheaper model?
5. Authentication: will the user sign in with their own GitHub account?
