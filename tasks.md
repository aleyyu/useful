# Tasks

Give the agent **one task at a time**. First read `AGENTS.md` and `docs/spec.md`. When a task is done, run the build and summarize the result. Do not move to the next task before the current one is finished.

## Phase 0 — Hello spike

### T0.1 — Project skeleton
- The repo must already be created from JetBrains' "IntelliJ Platform Plugin Template" (a human does this; the agent gets a ready repo).
- Agent:
  - Remove all sample Kotlin code from the template (`src/main/kotlin`, `src/test/kotlin`) and every reference to it in `plugin.xml` (sample tool window, services, listeners, etc.) and in resources.
  - Create the Java source roots (`src/main/java`, `src/test/java`) and use a package name for this project (not the template's sample package).
  - Keep the build working. Do not remove build plugins or change Gradle structure unless the build requires it; if you think a Gradle change is needed, explain why first.
  - Set the plugin id/name in `plugin.xml`, set the target platform to IDEA 2025.3, verify the JDK 21 toolchain.
- Acceptance: `./gradlew build` and `./gradlew runIde` work, and the sandbox IDE opens.

### T0.2 — "Merhaba" on commit
- Agent: first find the current/recommended pre-commit check mechanism in the official IntelliJ docs and explain in 3 sentences why you chose it. Then implement it in Java. When the commit button is pressed, show "Merhaba" and do not block the commit.
- Acceptance: the Phase 0 acceptance criteria in the spec.

### T0.3 — Manual test and summary
- Human: try a real commit in a test project via `runIde`. Add observations to the decision log in `docs/architecture.md`.

## Phase 1 — Copilot access spike (after Phase 0)
### T1.1 — Feasibility
- Agent: read the official Copilot Java SDK documentation. Write a minimal example that sends one simple request from inside the plugin (on a background thread). Do not invent APIs.
- Acceptance: written answers to "Open questions" 1-5 in the spec. The answers are added to section 6 of `docs/spec.md`.

## Phase 2 — MVP (to be detailed after Phase 1)
- T2.1 Change collector and filter
- T2.2 Standards md reader and rule selector
- T2.3 Prompt builder and output schema
- T2.4 `ReviewClient` interface + Copilot implementation + fake
- T2.5 Output parser
- T2.6 Warning UI (CRITICAL / SUGGESTION)
- T2.7 Timeout, cancellation, and fail-open

## Phase 3 — Improvement
- Cache, daily request limit, settings screen, local rules, measurement (tokens/time).

## Example prompt to give the agent (template)
> Read `AGENTS.md` and `docs/spec.md`. Do task **T0.1** in `docs/tasks.md`. Do not touch other tasks. When done, run `./gradlew build`; summarize changed files, the result, and open questions.
