# Tasks

Give the agent **one task at a time**. First read `AGENTS.md` and `docs/spec.md`. When a task is done, run the build and summarize the result. Do not move to the next task before the current one is finished.

## Phase 0 — Hello spike (DONE)

### [x] T0.1 — Project skeleton
- The repo must already be created from JetBrains' "IntelliJ Platform Plugin Template" (a human does this; the agent gets a ready repo).
- Agent:
  - Remove all sample Kotlin code from the template (`src/main/kotlin`, `src/test/kotlin`) and every reference to it in `plugin.xml` (sample tool window, services, listeners, etc.) and in resources.
  - Create the Java source roots (`src/main/java`, `src/test/java`) and use a package name for this project (not the template's sample package).
  - Keep the build working. Do not remove build plugins or change Gradle structure unless the build requires it; if you think a Gradle change is needed, explain why first.
  - Set the plugin id/name in `plugin.xml`, set the target platform to IDEA 2025.3, verify the JDK 21 toolchain.
- Acceptance: `./gradlew build` and `./gradlew runIde` work, and the sandbox IDE opens.

### [x] T0.2 — "Merhaba" on commit
- Agent: first find the current/recommended pre-commit check mechanism in the official IntelliJ docs and explain in 3 sentences why you chose it. Then implement it in Java. When the commit button is pressed, show "Merhaba" and do not block the commit.
- Acceptance: the Phase 0 acceptance criteria in the spec.

### [x] T0.3 — Manual test and summary
- Human: try a real commit in a test project via `runIde`. Add observations to the decision log in `docs/architecture.md`.

## Phase 0.5 — Findings notification and tool window (DONE)

### [x] T0.5.1 — Findings model and tool window skeleton
- Agent: Create a `Finding` model (severity, file path, line, message) and a `Severity` enum (`CRITICAL`, `SUGGESTION`). Register an "AI Reviewer" tool window anchored at the bottom in `plugin.xml`. Show a static empty state ("No issues found"). Use the API recommended in `docs/reference/tool-windows.md`. Do not invent APIs.
- Acceptance: the tool window appears at the bottom in `runIde` and shows the empty state.

### [x] T0.5.2 — Show grouped findings
- Agent: Render findings grouped as "Warnings" (`CRITICAL`) and "Suggestions" (`SUGGESTION`) with counts, severity icons, and message/file/line columns or labels. Use standard IntelliJ UI components that follow the IDE theme.
- Acceptance: with a hardcoded list of fake findings, the groups and counts are correct in light and dark themes.

### [x] T0.5.3 — Navigation
- Agent: Double-clicking a finding opens the file at the given line.
- Acceptance: navigation works for a real file in the test project.

### [x] T0.5.4 — Notification with counts and "Review code analysis" link
- Agent: Register a notification group named "AI Reviewer" in `plugin.xml`. Create a notifier class that takes a list of `Finding` objects and shows a warning-level balloon in the bottom-right corner with exactly this layout: title "AI Reviewer"; a line "<N> warnings, <M> suggestions" (warnings = `CRITICAL`, suggestions = `SUGGESTION`); below it a clickable link with the exact text "Review code analysis". Clicking the link opens and activates the "AI Reviewer" tool window. Use the API recommended in `docs/reference/notifications.md`. Do not invent APIs.
- Acceptance: with a hardcoded list of findings, the balloon shows the correct counts and the link, and clicking the link opens the tool window with those findings.

### [x] T0.5.5 — Connect to the commit hook and remove "Merhaba"
- Agent:
  - Create one single provider of fake findings (exactly 2 warnings and 3 suggestions) used by both the balloon and the tool window.
  - On commit, load the fake findings into the model, update the tool window content, and show the balloon via the notifier. Do not open the tool window automatically. Do not block the commit.
  - Completely remove the old "Merhaba" message (code, resources, any dialog or notification).
- Acceptance: spec section 3b acceptance criteria. (Implemented; human acceptance check pending.)

### [x] T0.5.6 — Housekeeping: naming, build target, docs
- Agent:
  - Naming: make "AI Reviewer" the only user-facing name: tool window id and title, notification group id and display name, balloon title, plugin name where it is a display name, all code, resources (message bundles), `plugin.xml`, and tests. Search the repo for leftovers of "AI Review" that are not followed by "er" and fix them.
  - Build target: change the Gradle default platform from the old Community 2024.2.5 to IntelliJ IDEA 2025.3 (unified distribution, build 253), following `docs/reference/gradle-plugin.md`. Keep the optional local-IDE build via the `COMMIT_REVIEWER_IDE` environment variable working. Make sure `sinceBuild` is consistent with 2025.3.
  - Document the local-IDE build option in `AGENTS.md` (Commands) if it is not already documented.
  - Do not try to fetch web documentation (the network is restricted).
- Acceptance: spec R0.5.13 and R0.5.14; `./gradlew build` passes (with the local IDE if downloads fail); a search for the stale name returns nothing; `verifyPlugin` is run only if the network allows it, otherwise say so in the summary.

## Phase 1 — Copilot access spike (ACTIVE)

Goal and go/no-go criteria: spec section 3c. Work in this order; do not skip ahead. Use only dummy code in prompts.

### T1.0 — Prerequisites (human)
- Ask the company's Copilot admin whether the Copilot CLI/SDK is allowed for your account (organization policy), and whether credits are per user or pooled.
- Install and sign in as described in the Copilot SDK file in `docs/reference/`. Run the quick-start example from that document outside the plugin, if it has one.
- Write the results (versions, sign-in method, policy answer) into `docs/phase1-findings.md`, section "Environment".
- Acceptance: the quick start works outside the plugin, or the blocking problem is written down.

### T1.1 — Read the SDK docs and write findings (agent, no code)
- Agent: Read only the Copilot Java SDK file in `docs/reference/`. Write `docs/phase1-findings.md` covering: dependency coordinates and version; minimum Java version; whether the SDK bundles or separately requires the Copilot CLI and how it finds it; authentication; how to send one prompt and read the response; how to disable tools/agent behavior; model selection; whether token/usage information is reported; streaming; timeouts and cancellation; error types; threading expectations; transitive dependencies (possible conflicts with libraries bundled in the IDE). Answer spec section 6 questions 1-5 as far as the docs allow, marking each `from docs` or `unknown`. End with a short list of risks for this plugin. Do not write code and do not browse the web.
- Acceptance: the file exists, every item is answered or marked unknown, and risks are listed.

### T1.1b — Update findings with the additional SDK pages (agent, no code)
- Human: Paste the SDK's Documentation and Advanced Usage pages, matching the SDK version in use, into `docs/reference/`.
- Agent: Update `docs/phase1-findings.md`. Re-check every `unknown` item against the new pages. In particular: how to make no tools available (`availableTools` / `excludedTools` semantics, what an empty list means), permission handling (deny-all), skipping custom instructions and the working directory (`cwd`), system message replace mode, reasoning effort, usage/billing events, timeouts and cancellation, error types, a simple send-and-wait style call, connecting to an already running CLI server versus starting one per request, authentication options (for example a token option), and whether the SDK is officially supported or community-driven. Mark each item `from docs`. State which go/no-go criteria are now answered. Do not write code and do not browse the web.
- Acceptance: no `unknown` remains that the pasted pages could answer; the highest-priority unknown (single-shot without tools) is answered or clearly still open.

### T1.2 — Dependency check
- Precondition: T1.1b is done and the T1.0 policy answer is not "no". Do not start this task otherwise.
- Agent: Add the SDK dependency to the build as described in the findings. Make sure `./gradlew build` and `runIde` still work and the plugin packages the SDK correctly. If the download is blocked by the network, stop and report; do not work around it without asking. Check for conflicts with libraries the IDE already bundles and report them in `docs/phase1-findings.md`.
- Acceptance: build passes; conflict report written (spec R1.7).

### T1.3 — Dev-only "Test Copilot request" action
- Agent: Add a Tools menu action "AI Reviewer: Test Copilot Request" (mark all of it `// SPIKE`). It sends one fixed, tiny, dummy prompt through a small `ReviewClient`-style interface implemented with the SDK, on a background thread, and shows the response or the error in a balloon. The UI must never block; support cancel and a 30 s timeout. Apply spec R1.3 and R1.8 (no tools, minimal context, no custom instructions loaded, lowest useful reasoning effort, output limit, wherever the SDK allows it). Do not connect this to the commit hook.
- Acceptance: spec R1.2, R1.3, R1.5, R1.6.

### T1.4 — Measurements
- Agent: Extend the spike to run three dummy prompts of different sizes (small about 20 changed lines, medium about 100, large about 400) and record latency and any usage information the SDK returns in `docs/phase1-findings.md`, section "Measurements".
- Human: Check the credit/usage numbers in your Copilot account before and after, and add them to the table.
- Acceptance: a table with three rows; R1.4.

### T1.5 — Decision (human + agent)
- Human: Apply the go/no-go criteria in spec section 3c and record the decision in the `docs/architecture.md` decision log.
- Agent: Update spec section 6 with the confirmed answers, update the `ai` package part of `docs/architecture.md` based on the findings, and list all `// SPIKE` code that should be removed or replaced in Phase 2.
- Acceptance: decision recorded; spec section 6 updated.

## Phase 2 — MVP (to be detailed after Phase 1)
- T2.1 Change collector and filter
- T2.2 Standards md reader and rule selector
- T2.3 Prompt builder and output schema
- T2.4 `ReviewClient` interface + Copilot implementation + fake
- T2.5 Output parser
- T2.6 Connect real findings to the notification and the "AI Reviewer" tool window
- T2.7 Timeout, cancellation, and fail-open

## Phase 3 — Improvement
- Cache, daily request limit, settings screen, local rules, measurement (tokens/time).

## Example prompt to give the agent (template)
> Read `AGENTS.md` and `docs/spec.md` (section 3c). Do task **T1.1b** in `docs/tasks.md`. Read only the Copilot SDK files in `docs/reference/` and `docs/phase1-findings.md`. Do not touch other tasks. Do not try to fetch web documentation. When done, summarize what changed in the findings, what is still unknown, and open questions.
