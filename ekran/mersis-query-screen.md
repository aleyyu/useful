# Task: Mersis Query Screen

## Goal
Build a new screen in this React project that exactly matches the draft screenshot at `docs/reference/mersis-screen-draft.png`. The screen lets the user query a company by its Mersis number and shows the backend result.

## Required Reading (in this order)
1. `docs/project-overview.md` — project structure and conventions (follow strictly)
2. `docs/reference/mersis-screen-draft.png` — draft UI to reproduce
3. `docs/reference/mersis-sample-response.md` — sample JSON returned by the backend

## Language Rules
- All code in English (identifiers, files, comments).
- Only on-screen texts are Turkish.

| Turkish UI text | Code name |
|---|---|
| Mersis Numarası | `mersisNo` |
| Sorgula | `search` |
| Temizle | `clear` |

Other labels in the result area: take the exact Turkish texts from the draft screenshot, and give them English code names derived from the JSON field names.

## Functional Requirements

### Input: Mersis Number (`mersisNo`)
- Accepts digits only. Letters, spaces, and special characters must be blocked while typing AND when pasting (strip non-digit characters on change/paste).
- Maximum length is 16 characters.
- A valid Mersis number is exactly 16 digits.
- Use numeric keyboard hints (`inputMode="numeric"`) but do NOT rely on `type="number"` (it drops leading zeros and allows `e`, `+`, `-`).
- If the user tries to search with fewer than 16 digits, do not call the backend; show a Turkish validation message under the input (for example: "Mersis numarası 16 haneli olmalıdır.").

### Button: Search (`search`, Turkish label "Sorgula")
- Calls the backend service with the entered `mersisNo`.
- Disabled while a request is in progress (show a loading state).
- On success: render the response data in the matching places of the result area, as shown in the draft screenshot.
- On error / empty result: show a clear Turkish error message and keep the input value.
- Avoid duplicate requests and ignore stale responses (if a new request is made or the screen is cleared while a request is in flight).

### Button: Clear (`clear`, Turkish label "Temizle")
- Clears the input value.
- Clears the result area if there is one (and any error / validation message).
- Returns the screen to its initial state.

## Backend Service
- Endpoint: `<<FILL IN: HTTP method and URL path>>`
- Request: `<<FILL IN: how mersisNo is sent (query param / path param / body field name)>>`
- Response: see `docs/reference/mersis-sample-response.md`
- Use the existing API/service layer and base URL/config approach described in `docs/project-overview.md`. If the endpoint above is still not filled in, create the service function with a clearly named constant for the path and mention it in your final summary so it can be completed.

## Mapping the Response to the Screen
- Compare the JSON structure in `docs/reference/mersis-sample-response.md` with the result area in the draft screenshot and map each JSON field to its place on the screen.
- Create TypeScript types/interfaces (or the project's equivalent) for the response, based on the sample JSON.
- Handle missing / null fields gracefully (show an empty value placeholder, never crash).
- Format values (dates, amounts, etc.) only if the screenshot shows a specific format.

## Deliverables
- Screen/page component, its sub-components if needed, service function, types, and Turkish label constants, all placed according to `docs/project-overview.md`.
- Route/menu registration only if the project overview says new screens must be registered.
- Unit/component tests if the project already has a testing setup.

## Acceptance Checklist
- [ ] UI matches the draft screenshot
- [ ] Letters and special characters cannot be typed or pasted into the input
- [ ] Input never exceeds 16 digits; search with fewer than 16 digits shows a validation message and sends no request
- [ ] Search sends the request and fills the result area from the response
- [ ] Loading, error, and empty states work
- [ ] Clear resets the input, the result area, and any messages
- [ ] All code is English; only on-screen labels are Turkish
- [ ] No new libraries or structures outside the project conventions

## Prompt for Copilot Agent
```
Read .github/copilot-instructions.md and docs/tasks/mersis-query-screen.md and follow them exactly. Implement the Mersis Query Screen using the project conventions in docs/project-overview.md, the screenshot docs/reference/mersis-screen-draft.png, and the sample response in docs/reference/mersis-sample-response.md. When finished, list the created/changed files and anything that still needs to be filled in (for example the endpoint).
```
