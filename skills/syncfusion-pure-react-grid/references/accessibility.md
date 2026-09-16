---
name: accessibility
description: Accessibility configuration and behavior for the Syncfusion React Data Grid — WCAG 2.2 / Section 508 partial compliance, WAI-ARIA roles and attributes (grid, row, rowgroup, columnheader, gridcell, aria-colindex, aria-rowindex, aria-selected, aria-sort, aria-busy), keyboard shortcuts across pager / editor / sorter / selection, allowKeyboard prop, and known accessibility gaps. Load when building an accessible grid, writing keyboard-only navigation, configuring screen-reader labelling, or planning accessibility remediation.
---

# Accessibility

The Syncfusion React Data Grid was built with WCAG 2.2 and Section 508 in mind. Some ARIA-in-HTML warnings appear in automated checkers because of the two-table header/body architecture — those are known gaps listed below.

## Compliance status

| Aspect | Status |
|---|---|
| WCAG 2.2 | 🕒 **Partial** |
| Section 508 | 🕒 **Partial** |
| Screen Reader | ✅ |
| Right-to-Left | ✅ |
| Color Contrast | ✅ |
| Mobile | ✅ |
| Keyboard Navigation | 🕒 **Partial** |
| Accessibility Validation | 🕒 **Partial** |
| Axe-core Validation | 🕒 **Partial** |

## WAI-ARIA roles in use

| Role | Purpose |
|---|---|
| `grid` | Grid container |
| `row` | Row element |
| `rowgroup` | Row group |
| `columnheader` | Column header cell |
| `gridcell` | Grid cell |
| `button` | Buttons in grid |
| `search` | Toolbar search region |
| `navigation` | Pager element |
| `presentation` | Hidden from assistive tech |

Override via `L10n.load` key targets — search for keys like `columnheader`, `gridcell`, `aria-sort` when localising.

## WAI-ARIA attributes in use

| Attribute | Purpose |
|---|---|
| `aria-colindex` | Logical column index in the grid |
| `aria-rowindex` | Logical row index in the grid |
| `aria-selected` | Selected state of rows/cells |
| `aria-sort` | `ascending`, `descending`, `none` — reflects current sort |
| `aria-busy` | Updating state during async loads |

## Keyboard shortcuts

### Pager
- `Tab` / `Shift+Tab` — across pager items
- `Enter` / `Space` — activate page
- `←` / `PageUp` — previous page
- `→` / `PageDown` — next page
- `Home` (`Ctrl+Alt+PageUp` / `Fn+←`) — first page
- `End` (`Ctrl+Alt+PageDown` / `Fn+→`) — last page

### Cell focus
- `Home` (`Fn+←`) — first cell in current row
- `End` (`Fn+→`) — last cell in current row
- `Ctrl+Home` (`⌘+Fn+←`) — first cell of first row
- `Ctrl+End` (`⌘+Fn+→`) — last cell of last row
- Arrow keys — navigate cell focus
- `Alt+J` (`⌥+J`) — focus the entire grid
- `Alt+W` (`⌥+W`) — focus the grid content

### Selection
- Arrows — move focus
- `Shift+↑` / `Shift+↓` — extend selection
- `Enter` — move focus down
- `Shift+Enter` — move focus up
- `Esc` — clear row selection

### Editing
- `F2` — start editing the selected row
- `Enter` — save form
- `Insert` (`⌘+⌥+Enter` on macOS) — create new form
- `Esc` — cancel edit
- `Delete` — delete the focused row
- `Tab` / `Shift+Tab` — next / previous editable cell
- `Shift+Enter` — save (alternate)

### Sorting
- `Enter` — toggle sort on focused column (asc → desc → none)
- `Ctrl+Enter` (`⌘+Enter`) — multi-column sort
- `Shift+Enter` — clear sort on focused column

## Disabling keyboard interaction

The `<Grid>` accepts `allowKeyboard={false}` to disable all built-in keyboard interactions:

```tsx
<Grid allowKeyboard={false} ... />
```

Use only when the grid is purely informational and you have a parallel control panel (e.g., a streaming ticker with a side config panel).

## Localisation labels

`L10n.load` keys that map directly to accessibility labels: `columnHeader`, `gridCell`, `ariaSortAscending`, `ariaSortDescending`, `ariaSortNone`, `ariaSelected`, `ariaBusy`, `pagerStatusMessage`, `firstPageTooltip`, `lastPageTooltip`, `nextPageTooltip`, `previousPageTooltip`, `nextPageGroupTooltip`, `previousPageGroupTooltip`, `currentPageLabel`, `totalItemsLabel`, `pagerOfLabel`.

See `references/globalization.md` for the full locale-key catalog.

## Known accessibility gaps

The grid ships partial WCAG 2.2 compliance. Known gaps to plan around:

- **Two-table architecture** (header + content split) may trigger automated checker warnings.
- **`aria-required-children`** warnings when the grid renders without all features/toolbar (the `navigation` role for the pager expects a `search` region; missing roles raise warnings).
- **`role="grid"` with `<tr>`** triggers ARIA-in-HTML spec warnings — the grid resolves these with semantic HTML, but axe-core may still flag them.
- **Scrollable regions** may need explicit `tabindex` for keyboard access — the grid handles this on most browsers, but custom scroll wrappers should add `tabindex="0"`.
- **Checkbox inputs** may require explicit `<label>` association in your templates.
- **`role="rowgroup"`** validation against `grid`/`table` — present but typically benign.

## How to remediate

When the gaps surface in your specific context:

- Use `aria-label` / `aria-labelledby` on the grid root or container label.
- Add `tabindex="0"` on custom scroll wrappers.
- For tooltips on toolbar buttons (column chooser, print, etc.) override via `L10n.load`.
- Pair `altRow` and `cellClass` to ensure color is **not** the only signal (WCAG 1.4.1). Use icons + text for status, error, and selection states.

## Constraints & guardrails

- **`allowKeyboard={false}`** breaks keyboard navigation — avoid unless there's a parallel control path.
- **Streaming grids** (`enableHover=false`, `selectionSettings.enabled=false`, `allowKeyboard=false`) drop several ARIA affordances — apply only when the dataset is monitoring-oriented.
- **Color-only signals** violate WCAG 1.4.1. `cellClass` should also include text or icons, not just background colors.
- **High-risk motion**: avoid animating focus rings or sorting indicators beyond subtle transitions — vestibular-disorder users may be affected.
- **Multi-locale grids**: after switching `<Provider locale>`, re-test keyboard shortcuts — some platform keymaps differ (macOS `⌘` vs Windows `Ctrl`).

## Cross-references

- For locale-specific labels and the full L10n catalog, see `references/globalization.md`.
- For WCAG-compliant theming variables, see `references/theming.md`.
- For the `allowKeyboard` prop and other Grid-root props, see `references/modules-and-imports.md`.