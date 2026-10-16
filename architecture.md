# Architecture — AI Commit Review Plugin

Status: DRAFT v0.1. For Phase 0 only the "Commit hook" part is implemented.

## 1. Overall flow (target)
```
Commit button
   -> Commit hook (IntelliJ commit check)
   -> [background thread]
        1. Change collector: only the diff to be committed
        2. Filter: drop generated/lock/binary/very large files
        3. Local rules: deterministic checks without AI
        4. Cache: was this diff hash already asked?
        5. Rule selector: relevant rules from the standards md
        6. Prompt builder: diff + relevant rules + output schema
        7. AI client (interface) -> Copilot (single-shot, no tools)
        8. Output parser: JSON -> list of findings
   -> UI: show findings (CRITICAL / SUGGESTION)
   -> Commit proceeds (fail-open)
```

## 2. Components (suggested packages)
- `hook` — IntelliJ commit integration (plugin.xml registration, handler)
- `collect` — change collection and filtering
- `local` — local rule engine
- `standards` — md reading, splitting, rule selection
- `prompt` — prompt building
- `ai` — `ReviewClient` interface + Copilot implementation + fake for tests
- `model` — `Finding`, `Severity`
- `ui` — warning display
- `settings` — user settings (budget, on/off, model)

The `ai` package must sit behind an interface: the Copilot access path may change in Phase 1, and the rest of the code must not be affected.

## 3. Threading rules
- The commit hook is called on the EDT. Only very fast work is done there.
- Diff collection, the AI call, and parsing run in the background (using IntelliJ's cancellable background task/progress mechanism).
- If the user cancels or a timeout occurs, the work stops and the commit is released.
- UI updates happen on the EDT.

## 4. Token-saving layers (in priority order)
1. Local rules run before the AI.
2. Filter: unnecessary files are never sent.
3. Only changed hunks + a few context lines are sent.
4. Only relevant rules from the standards are sent.
5. Cache: no repeat request for the same diff.
6. Single-shot, no tools, short structured output, max output limit.
7. Daily/per-user request limit, ability to disable in settings.
8. Cheaper model option.

## 5. Error and security principles
- Fail-open: any AI-side error releases the commit.
- Source code, tokens, and prompt contents are not written to logs.
- The plugin does not store credentials itself; Copilot's own session is used (to be confirmed in Phase 1).

## 6. Decision log
| Date | Decision | Reason |
|---|---|---|
| — | Language: Java | Team preference |
| — | AI call is single-shot, no tools | Token control |
| — | Fail-open | Never block the commit flow |
