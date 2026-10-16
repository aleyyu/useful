# AGENTS.md — AI Commit Review Plugin (IntelliJ)

Persistent rules for every AI agent (Copilot agent mode, Copilot CLI, etc.) working in this repo. Follow them in every session.

## Project
An IntelliJ IDEA plugin that, before a developer commits, takes the changes about to be committed (the diff) and:
1. Checks them against the team's coding standards defined in an md file,
2. Finds error-prone, critical points,
3. Reports findings as "important warnings" (must fix) and "minor warnings" (could improve).
AI calls use the company's GitHub Copilot account credits.

Two core constraints: **low token consumption** and **never slowing down the IDE**.

## Read first
- `docs/spec.md` — what to build, acceptance criteria, budgets
- `docs/architecture.md` — architecture decisions
- `docs/tasks.md` — task list (your assigned task comes from here)

## Technical environment
- Target IDE: IntelliJ IDEA 2025.3.x (build 253). Since 2025.3 there is a single unified distribution; use the `intellijIdea()` dependency in Gradle.
- Build: IntelliJ Platform Gradle Plugin 2.x (2.10.4 or newer). Start with the version the template ships; do not upgrade it on your own.
- JDK: IntelliJ Platform 2025.3 is Java 21 level. This project **builds and runs on JDK 21**. (The team's general standard is Java 17; this project is an exception. This assumption is verified in the first task.)
- Language: **Java**. Do not write Kotlin application code. Note: Gradle build files (`*.gradle.kts`) may stay as Kotlin DSL; they are build scripts, not application code. Delete the sample Kotlin classes that come with the template.
- Commands:
  - Build and test: `./gradlew build`
  - Tests only: `./gradlew test`
  - Launch sandbox IDE for manual testing: `./gradlew runIde`
  - Compatibility check: `./gradlew verifyPlugin`

## Working rules
- Do **only the single task** you are given. Do not go beyond `docs/spec.md` or add extra features.
- **Do not invent APIs.** If you are unsure about an IntelliJ Platform or Copilot SDK API, check the official docs or source code. If still unsure, ask instead of assuming.
- Do not add unnecessary dependencies. If a new dependency is needed, explain why and ask for approval.
- When the task is done, run `./gradlew build`. Summarize in this format: changed files, what works / what does not, open questions.
- No broad refactors. Make small, reversible changes.
- Do not read the whole repo unless needed. Open only files relevant to the task (your own token usage also consumes credits).

## Non-negotiable architecture rules
1. **Never block the UI thread (EDT).** Network calls, AI calls, process execution, and heavy diff processing run on a background thread.
2. **Fail-open:** an AI error, timeout, or credit/limit problem must never block a commit. Show the user a short notice and allow the commit.
3. **The AI call is single-shot:** no tool use, no agent loop; one prompt and a structured (JSON) response. Details in `docs/architecture.md`.
4. Source code and tokens/credentials are **never written to logs**.
5. Local (non-AI) checks run first; the AI is called only when needed and with small input.

## Testing
- Write unit tests (JUnit) for every class with logic. For IDE-dependent behavior, use the IntelliJ Platform test infrastructure (`BasePlatformTestCase`, etc.).
- AI-call tests **never hit the real service**; use a fake implementation behind an interface.
