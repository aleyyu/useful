# Copilot Instructions

## Project Context
- This is a React frontend project. Our team builds all React projects with the same structure.
- Before writing any code, READ `docs/project-overview.md`. It describes the general project structure, conventions, folder layout, naming rules, API layer, and styling approach. Follow it strictly.
- Do not introduce new libraries, folder structures, or patterns that are not already used in the project overview. If something is missing, reuse the closest existing pattern.

## Language Rules (MANDATORY)
- ALL code must be in English: file names, folder names, component names, function names, variables, constants, types/interfaces, props, state, comments, commit messages, and test names.
- ONLY user-facing text visible on the screen (labels, button texts, placeholders, validation/error messages) is Turkish.
- Turkish UI text must be kept in a single central place (for example a `labels` / `constants` / i18n file, following the project overview) and referenced from components. Do not hardcode Turkish strings inside JSX when the project has a central place for them.
- Example mapping:
  - "Mersis Numarası" -> code name `mersisNo`
  - "Sorgula" -> code name `search`
  - "Temizle" -> code name `clear`
- Make sure Turkish characters (ç, ğ, ı, ö, ş, ü, İ) are saved correctly (UTF-8).

## Implementation Rules
- Reproduce screens exactly as shown in the provided draft screenshot (layout, order of fields, spacing, texts). Do not redesign.
- Keep components small and typed the same way the existing project does (TypeScript or JavaScript as per the project overview).
- Keep API calls in the project's existing service/API layer, not inside UI components.
- Handle loading, success, empty, and error states for every backend call.
- Do not leave unused code, console logs, or TODOs.

## Task Files
- Feature tasks live in `docs/tasks/`. Reference materials (screenshots, sample backend responses) live in `docs/reference/`.
- When a task file is given, read it fully together with the files it references before coding.
