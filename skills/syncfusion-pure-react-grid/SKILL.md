---
name: syncfusion-pure-react-grid
description: Production-ready guidance for the Syncfusion® React Data Grid (`@syncfusion/react-grid`). Use when the user is building, modifying, generating, scaffolding, configuring, or troubleshooting a React Data Grid — including features like filtering, sorting, paging, searching, editing, selection, grouping, aggregates, virtualization, infinite scroll, master/detail, autofill, clipboard, context menu, column resize/reorder/menu/chooser/spanning, row drag-and-drop, row numbers, sorting/filtering modes (FilterBar, Menu, Excel, CheckBox), print, PDF export, Excel export, accessibility, RTL/locale, theming, performance tuning, security, or framework compatibility. Trigger even when the user only mentions a "grid", "data table", "tabular data UI", "admin panel", or "dashboard table" in a React project, when they paste Syncfusion React Grid code, or when they ask how to wire up the grid with a remote data source (OData, WebApi, REST, custom `DataManager`) or a custom API. Do not use for non-React frameworks (Angular, Vue, Blazor, ASP.NET MVC), other Syncfusion components (charts, scheduler, Gantt, Rich Text Editor, etc.), or generic Tailwind/Material UI tables.
metadata:
  author: "Syncfusion Inc"
  version: "34.1.29"
---

# Syncfusion® React Data Grid

A knowledge and decision skill for everything in `@syncfusion/react-grid` (the pure-React Syncfusion grid, React 17+). It bundles the full feature surface, the conventions the library expects, and the guardrails that keep generated code safe.

The skill is **source-of-truth aware**: every feature, prop, enum, event, and module listed here comes from the official Syncfusion React Data Grid documentation. Nothing here is invented. When a feature is in flux or has known limitations, this skill calls that out explicitly.

## How to use this skill

1. **Identify the user's intent** — are they scaffolding a new grid, configuring an existing one, debugging, or generating new UI on top of it?
2. **Pick the right reference file(s)** from the table below. Read each before authoring code in that area — references are the authoritative detail surface.
3. **Always pass the baseline settings** the grid expects (see "Mandatory default settings" below). Missing modules, missing primary key, or missing `enableDevMode={false}` silently break features.
4. **Apply guardrails** when generating or modifying code. The full enforcement rules live in `references/guardrails-applied.md`.

## Mandatory default settings (apply to every Grid)

These settings are mandatory in **every** grid you produce unless the user explicitly opts out:

```tsx
import {
  Grid, Columns, Column, /* all features you plan to use */
  FilterModule, PagerModule, /* etc. */
  /* all settings/enums/events relevant to the feature mix */
} from '@syncfusion/react-grid';

const modules = { FilterModule, PagerModule /* add as needed */ };

<Grid
  dataSource={data}
  modules={modules}
  enableDevMode={false}
  // ...other feature-level settings
>
```

- **Modules map**: every non-built-in feature requires its module — see `references/modules-and-imports.md`. `GridAllModule` may be used for prototypes; production should pick single modules.
- **`enableDevMode={false}`**: suppresses diagnostic console output. Required on every example in the official docs.
- **Primary key**: define exactly one column with `isPrimaryKey={true}` whenever the user enables editing, selection persistence, setCellValue/setRowData, autofill, clipboard paste, or checkbox selection. Without it, edits/deletes misbehave (first-row-only writes, selection doesn't persist across paging).
- **Module registration first**: a feature that needs `XModule` but lacks it produces a silent runtime error, often mistaken for a data bug.

## Reference file index

Load the relevant reference(s) before authoring code in that area. Files are scoped — load only what applies.

| Concern | Reference file |
|---|---|
| Install, modules map, all named exports, enums, settings types | `references/modules-and-imports.md` |
| Columns: configuration, types, format, templates, headers, resizing, reorder, menu, chooser, spanning, dynamic state | `references/columns.md` |
| Cells: clip mode, text wrap, styling, grid lines, expressions, HTML content | `references/cells.md` |
| Rows: rows, templates, row template, row numbers, row drag-and-drop, row spanning | `references/rows.md` |
| Data binding: local, remote (DataManager/adaptors), custom `onDataRequest`, empty-record template | `references/data-binding.md` |
| Paging | `references/paging.md` |
| Sorting | `references/sorting.md` |
| Filtering (all four modes), operators, programmatic | `references/filtering.md` |
| Searching | `references/searching.md` |
| Editing — Configuration, all modes, edit types, validation, command column | `references/editing.md` |
| Editing — Inline, Popup, PopupTemplate, Cell, custom-edit | `references/editing-modes.md` |
| Selection — Row, Cell, Checkbox, Conditional, Cell selection type | `references/selection.md` |
| Grouping + aggregates | `references/grouping-and-aggregates.md` |
| Virtualization, virtual scroll, infinite scroll — performance data strategies | `references/performance-and-scrolling.md` |
| Autofill, Clipboard, Context Menu, Toolbar | `references/interactivity.md` |
| Master/Detail — simple `detailRowTemplate` and relational `detailCellRendererParams` | `references/master-detail.md` |
| Real-time grid (interval-driven updates, setCellValue + currentViewData) | `references/real-time-grid.md` |
| Print, PDF Export, Excel Export — the three hook-based exports | `references/exports.md` |
| Accessibility (WCAG roles/attrs, keyboard map, known gaps) | `references/accessibility.md` |
| Globalization (L10n, loadCldr, Provider, `enableRtl`, locale prop) | `references/globalization.md` |
| Theming (Material/Bootstrap/Tailwind, light/dark, `--sf-*` variables) | `references/theming.md` |
| Security, compatibility, browser support | `references/security-compatibility.md` |
| Guardrails integrated into generated code | `references/guardrails-applied.md` |

## Imperative API surface (`GridRef`)

`gridRef.current?.<method>` is the public imperative handle. Key methods (full list with signatures in `references/imperative-api.md` — load that file when generating command-button code, custom toolbars, or programmatic mutations):

- Sort: `sortByColumn`, `removeSortColumn`, `clearSort`
- Filter: `filterByColumn`, `clearFilter`
- Search: `search`, `getData`
- Paging: `goToPage`, `pagerRef.totalRecordsCount`
- Selection: `selectRow`, `selectRows`, `clearRowSelection`, `clearSelection`, `clearCellSelection`, `getSelectedRecords`, `getSelectedRowIndexes`
- Editing: `addRecord`, `editRecord`, `updateRecord`, `setCellValue`, `setRowData`, `deleteRecord`, `saveDataChanges`, `cancelDataChanges`, `editCell`, `saveCellChanges`, `cancelCellChanges`
- Columns: `getColumnByField`, `resizeColumns`, `reorderColumns`, `openColumnChooser`, `getColumns`
- Autofill: `applyFill`
- Clipboard: `copyToClipboard`, `pasteFromClipboard`, `cutToClipboard`
- Layout: `refresh`
- Streaming: `currentViewData`

Use a `useRef<GridRef>(null)` and always chain optional-call: `gridRef.current?.method?.()`.

## Decision patterns (use these to pick features, don't memorize props)

When the user says... → reach for:

- "Filter my table" → start with `filterSettings.type: 'FilterBar'` (default). Move to `'Menu'` for AND/OR compounds, `'Excel'` for Excel-style checklist, `'CheckBox'` for a unified BA-only dropdown.
- "Sort by clicking headers" → `sortSettings.enabled = true` (default mode `Multiple`, ctrl-click to add, shift-click to remove).
- "Edit inline like Excel" → `editSettings = { mode: 'Cell', allowEdit: true }` + `editCell` / `saveCellChanges`.
- "Edit one row at a time" → `editSettings = { mode: 'Normal', allowEdit: true, allowAdd: true, allowDelete: true }` + toolbar.
- "Show a modal form for editing" → `mode: 'Popup'` + `popupSettings`. Fully custom form → `mode: 'PopupTemplate'` + `popupTemplate`.
- "Bulk delete with checkboxes" → `<Column type='checkbox' width='40' />` + `selectionSettings.persistSelection` (auto-enabled) + toolbar `Delete`.
- "Render thousands of rows fast" → `virtualizationSettings.enabled = true` + fixed `rowHeight` + `getRowHeight` only if needed.
- "Stream live data" → `setCellValue(pk, field, value)` with fixed `rowHeight` and `enableHover={false}`.
- "Master/detail rows" → `isMasterDetail` + `detailRowTemplate` for simple cases; `detailCellRendererParams` for relational binding.
- "Print / PDF / Excel" → `useGridPrint`, `useGridPdfExport`, `useGridExcelExport` hooks (these are the public API; pass `gridRef`).
- "Right-to-left" → `<Provider locale='ar'><Grid enableRtl /></Provider>`.

## Guardrails applied

This skill follows the same enforcement model as the project's `guardrails` skill: it tells you *when* each check applies and *how* to apply it, without re-teaching lint/test/security concepts the agent already knows.

When generating, editing, or modifying a React Data Grid, follow `references/guardrails-applied.md`. The summary rules:

- Confirm `isPrimaryKey` exists before turning on editing, selection persistence, setCellValue, autofill, clipboard, or checkbox selection.
- Never edit template/`template`/`cellClass`/`valueAccessor` callbacks that are not memoized (`useCallback`/`memo`) — these re-run on every render and degrade scroll perf.
- Always memoize the `<Grid>` JSX with `useMemo` keyed on data + handler identities for stable rendering.
- Use fixed `rowHeight` for any dataset over ~1k rows, any streaming grid, or any virtualized setup. Avoid `autoHeight` on columns that contain templates (it disables column virtualization).
- Migration/aggregate moments: integrate the change — register modules, import named exports, wire CSS theme at `App.css`, ensure the `detailRowTemplate` is reachable from the parent. Integration > correctness.
- Bug fix retrospectives for grid features: detection gap (silent feature failures come from missing modules / missing PK), prevention (add an explicit module registration test or PK assertion), pattern scan (sibling columns missing `isPrimaryKey`).
- Confirm sensitive operations (mass delete, `clearFilter([])` wiping all filters, persistence toggles that affect server payloads) before executing.
- Inject guardrail guidance into UI critical paths: never mass-delete from the server without a confirmation dialog coming from `editSettings.confirmOnDelete`.

## Output conventions for generated code

When generating React Data Grid code, always:

- Import everything explicitly from `@syncfusion/react-grid` (do not deep-import internal paths unless necessary for `DetailCellRendererParams`, `GetDetailRowDataParams`, `GroupSettings`).
- Use `enableDevMode={false}` on every `<Grid>`.
- Register modules in a `useMemo` or module-scope `const` — never inline.
- Wrap templates in `useCallback`, template components in `React.memo`.
- Memoize the `<Grid>` JSX with `useMemo`.
- Set `Column.type` explicitly when the first cell value could be `null`/`undefined`, otherwise the wrong default editor/filter is inferred.
- Mark exactly one column as `isPrimaryKey` whenever the grid mutates data.
- Pair `selectionSettings.type = SelectionType.Cell` with `cellSelectionType` setting whenever autofill is enabled.

## When NOT to use this skill

- Syncfusion Angular, Vue, Blazor, ASP.NET MVC, vanilla JS grids → wrong framework.
- Other Syncfusion React components (chart, scheduler, gantt, kanban, rich text editor, diagram, etc.) → not in scope. Use a different, dedicated skill.
- Non-React data tables (TanStack Table, AG Grid community edition, Mantine DataTable, MUI X DataGrid) → out of scope.

## Quick reference card

```tsx
// Minimum viable grid (built-in sorting + paging + filter-bar + search)
import {
  Grid, Columns, Column,
  FilterModule, PagerModule, SearchModule,
  SortSettings, FilterSettings, PageSettings, SearchSettings,
} from '@syncfusion/react-grid';

const modules = { FilterModule, PagerModule, SearchModule };

const [sort]    = useState<SortSettings>({ enabled: true });
const [page]    = useState<PageSettings>({ enabled: true, pageSize: 12 });
const [filter]  = useState<FilterSettings>({ enabled: true });
const [search]  = useState<SearchSettings>({ enabled: true });

<Grid
  dataSource={rows}
  modules={modules}
  enableDevMode={false}
  sortSettings={sort} filterSettings={filter} pageSettings={page} searchSettings={search}
  toolbar={['Search']}
>
  <Columns>
    <Column field="id" headerText="ID" isPrimaryKey />
    {/* ... other columns */}
  </Columns>
</Grid>
```