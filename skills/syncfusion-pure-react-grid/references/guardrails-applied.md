---
name: guardrails-applied
description: How the Syncfusion React Data Grid skill integrates guardrails — code-quality verification gates for grid generation: isPrimaryKey checks, module-registration integration tests, memoization, fixed rowHeight for large datasets, sensitive-operation gating (bulk delete/export), config-protection, and bug-fix retrospective template. Load when generating, modifying, debugging, or reviewing grid code.
---

# Guardrails applied

This skill follows the same enforcement model as the project's `guardrails` skill: it tells you **when** each check applies and **how** to apply it, without re-teaching lint/test/security concepts the agent already knows. Everything here is grid-specific guidance — the model's existing knowledge carries the rest.

When generating, editing, reviewing, or debugging a React Data Grid surface, apply the gates below.

## SessionStart — discovery

When the agent launches into a grid task, quickly confirm:

- **Module list in `package.json`** — confirm `@syncfusion/react-grid`, the matching theme package, and the peer packages actually used (`react-inputs`, `react-buttons`, `react-data`, etc.).
- **Existing grid configuration** — read at least one `<Grid>` instance to confirm the current `modules` map and `enableDevMode` state.
- **`LESSONS_LEARNED.md`** — if it exists, scan for prior grid gotchas (typings mismatches, silent module omissions, theme CSS imports).
- **Syncfusion license registration** — confirm `registerLicense` runs once at app boot.

If you find an inconsistency, propose change; do not silently weaken checks.

## Hard pre-implementation gates

These must be true before any meaningful grid work begins:

1. **At least one column is marked `isPrimaryKey`** — required for editing, persistent selection, setCellValue, autofill, clipboard paste, checkbox selection.
2. **`modules` map is consistent with `features` in use** — every non-built-in feature has its module registered. Missing modules are silent runtime errors.
3. **`enableDevMode={false}`** on every production grid.
4. **Templates and callbacks memoized** — `template`, `cellClass`, `valueAccessor`, `headerTemplate`, `filterTemplate` each use `useCallback`; template components wrap in `React.memo`.
5. **`<Grid>` JSX wrapped in `useMemo`** — keyed on data + handler identities.
6. **CSS theme imported** in `App.css` once, in the right dependency order.

If any of these fails **before** a feature starts, stop and remediate.

## Stop hook — verification gates

Every completion must verify:

1. **Modules registered** — `modules` matches the feature surface.
2. **`isPrimaryKey` column present** when edit/selection/autofill/persistence is on.
3. **Production files changed AND tests / types / lint pass** — no skipping coverage.
4. **Deployment integration**: new feature is reachable, not just compiled. A new module was registered, a setting bound, the toolbar entrypoint wired.

Return the gate decision with evidence: modules used, primary key column, tests/build status, integration path.

## Thrashing circuit breaker

If a grid-related fix has been attempted twice without converging:

1. **After attempt 1 fails**: try a direct fix. Normal.
2. **After attempt 2 fails**: stop. Build a diagnostic tool (see `references/diagnostics.md`-style patterns below) or use a notation (state transition table for module/edit mode interactions; decision matrix for mode selection).
3. **After attempt 3 fails**: stop and report to user. Two failed direct fixes means the model of the problem is wrong — re-read the same code produces the same wrong answer.

## Tool building and notation

When stuck, build a notation or tool before retrying:

- **Module/edit-mode decision matrix** — for "what modules do I need for this behaviour": decision per feature flag.
- **State table for editing modes** — modes (`Normal`/`Cell`/`Popup`/`PopupTemplate`) × state (`view`/`edit-add`/`edit-row`/`edit-cell`) → API surface.
- **Call site inventory** for any change that touches `<Grid>` props (most props affect many call sites).
- **Schema check** for the data source — verify first-row values against expected `ColumnType` to prevent wrong editor inference.

Diagnostic scripts (e.g., "what modules are loaded?") are welcome; promote them to project infra if reused.

## Buffer / virtualization decisions

For grid performance work:

- Default to fixed `rowHeight`, `viewPortBuffer: { rows: 5, columns: 5 }`, `enableCache: true`.
- Use `getRowHeight` only when content truly varies per row — and only after measuring.
- Streaming grids: `enableHover=false`, `selectionSettings.enabled=false`, `allowKeyboard=false`, fixed `rowHeight`.

## Commit / PreToolUse

Before any commit on grid changes:

- Run the **full suite** (project's fast check + integration).
- **Secrets scan** — confirm the Syncfusion license key isn't committed anywhere in source or staged files.
- **Bundle inspection** — confirm `react-grid` is tree-shaken to the right modules in the production build (no `GridAllModule` in production shells unless deliberate).
- **CSS theme import** — confirm `App.css` still imports the theme CSS.
- **License key** still registered (config protection).

## Bug-fix retrospective (required on `fix:` commits)

When a commit fixes a bug, answer (per the project's `guardrails` rule):

1. **Detection gap** — why didn't existing tests / types / running-config catch this?
   - Common grid causes: missing `isPrimaryKey`, missing module, missing memoization, value-vs-display distinction on `valueAccessor`.
2. **Prevention** — add a concrete artifact: a unit test that the column has `isPrimaryKey`, a type narrowing on `filter operator`, a memoization lint rule, etc.
3. **Pattern scan** — search the codebase for the same error pattern. For grid bugs that means: any other `<Grid>` without `isPrimaryKey`, any cell template that allocates per render, any `cellClass` without memoization.

Block the commit if any question is unanswered.

## Config Protection

The agent does NOT modify the grid's own configuration to make a failing test pass. Specifically:

- Don't change `validationRules` to weaken input checks.
- Don't change `allowEdit={false}` to `true` to silence type errors.
- Don't disable modules (`modules={{}}`) to hide feature regressions.
- Don't edit `gridLines`/`clipMode` to mask CSS regressions.

If config needs changing, propose it to the user with a reason.

## High-Risk Action Gating

Some grid operations are irreversible or reach beyond this code change:

- **Bulk delete** through `deleteRecord()` for many rows, or sending `{ isSelectAll: true, primaryKeys: [] }` to delete all — confirm intent.
- **`clearFilter([])`** with no argument clears all filters; confirm before bulk filter reset.
- **`persistSelection: true`** without `isPrimaryKey` can produce stale selection across pages — verify PK first.
- **Mass export** (PDF/Excel/Print across `range: 'All'`) of sensitive data — gate before trigger.
- **Mass insert/update via `setCellValue`/`setRowData`** with `triggerEvent=true` — gate before bulk commit.
- **Filter, sort, or paging changes driven by user-supplied input** without server-side validation — treat as user input; sanitize.

Describe the operation and its blast radius, then wait for user confirmation.

## Lessons learned (`LESSONS_LEARNED.md`)

Append an entry when encountering:

- A guardrail failure requiring multiple attempts (e.g., silent missing module).
- A non-obvious project convention (custom validation theme, custom pager).
- Surprising tool behaviour (`enableDevMode` emits a per-render warning; `setCellValue` defaults to UI-only).
- A deployment gap tests didn't catch (export button rendered without hook).
- A bug-fix retrospective that revealed a detection gap.
- A useful diagnostic script.

Use the project's template:

```markdown
### YYYY-MM-DD — [short title]
**Context:** What task was being performed.
**What happened:** The surprising behavior or failure.
**Resolution:** How it was resolved.
**Rule:** [One-line directive for future sessions to follow.]
```

Commit to version control. If a lesson reveals a missing guardrail, **propose adding one** — don't just document the workaround.

## Agent responsibilities (not hookable)

**Planning:** define completion criteria — what "done" looks like. "Add sort by Age" is not done; "Add a `Column` with sort, register `SortSettings`, write a unit test that confirms click cycles asc → desc → unsort, and visually verify on a sample dataset" is done.

**Code writing:** format/type-check incrementally. For grid code, every `<Grid>` instance must be inspected for missing module + missing `isPrimaryKey` before considering the change complete.

**Handoff:** report verified items, skipped layers + reason, security findings, coverage delta. Don't present work as complete if any gate failed.

## Cross-reference

- The full project's guardrails are in `references/guardrails/SKILL.md` (the existing repository skill). This file specialises those rules for grid generation/modification.
- For specific grid feature decisions, see the relevant `references/<topic>.md` file.