---
name: tool-panel
description: Tool panel sidebar in the Syncfusion React Data Grid — `sideBar` configuration, built-in panels (`columns`, `filters`, `editing`) powered by `ColumnToolPanelModule` / `FilterToolPanelModule` / `EditToolPanelModule`, and custom tool-panel definitions (`{ id, label, content }`) with `content.render()` for dynamic content. Load when adding a side-bar of column / filter / editing controls to the grid, or when surfacing app-specific summaries or shortcuts inside a panel alongside the grid.
---

# Tool Panel (side bar)

The tool panel side bar colocates column, filter, editing, and custom panels on the side of the grid. Each built-in panel is backed by its own module; custom panels are first-class objects passed into `sideBar.toolPanels`.

## Built-in modules

| Panel | Module | Purpose |
|---|---|---|
| `columns` | `ColumnToolPanelModule` + `ColumnChooserModule` (visibility / reorder) + `GroupModule` + `AggregateModule` (drill-down inside the panel) | Show / hide / reorder / group / aggregate via the panel UI. |
| `filters` | `FilterToolPanelModule` + `FilterModule` | Centralised filter editor; one row per filterable column. |
| `editing` | `EditToolPanelModule` + `EditModule` | Bulk-edit selected rows via per-column editors. |

Custom panels don't require a dedicated module — they live alongside the built-ins.

## Enabling the side bar

```tsx
import {
  Grid,
  ColumnToolPanelModule, ColumnChooserModule, GroupModule, AggregateModule,
  FilterToolPanelModule, FilterModule,
  EditToolPanelModule,     EditModule,
  ReorderModule,
  SideBar,
} from '@syncfusion/react-grid';

const modules = {
  ColumnChooserModule, GroupModule, AggregateModule, ReorderModule,
  FilterModule, EditModule,
  ColumnToolPanelModule, FilterToolPanelModule, EditToolPanelModule,
};

const sidebar: SideBar = {
  enabled: true,
  toolPanels: ['columns', 'filters', 'editing'],
  defaultToolPanel: 'columns',
  openByDefault: true,
  position: 'left',         // 'left' | 'right'
};

<Grid sideBar={sidebar} modules={modules} /* … other settings … */ />
```

`SideBar` options:

| Property | Description |
|---|---|
| `enabled` | Show the side bar. |
| `toolPanels` | Array of built-in identifiers (`'columns' | 'filters' | 'editing'`) and/or custom panel objects. Panels render in the listed order. |
| `defaultToolPanel` | Identifier of the panel selected when the side bar opens. |
| `openByDefault` | Open the side bar on first render. |
| `position` | `'left'` or `'right'`. |
| `hideButtons` | Hide the panel-tab buttons; only active panel content remains. |

## Columns panel

`sideBar.toolPanels: ['columns']` enables column-visibility toggles, grouping drop-targets, aggregations, and reordering when paired with `reorderSettings.enabled` and `columnChooserSettings.enableReorder: true`:

```tsx
const reorderSettings: ReorderSettings        = { enabled: true };
const columnChooserSettings: ColumnChooserSettings = { enableReorder: true };
const sidebar: SideBar                       = { enabled: true, toolPanels: ['columns'] };

<Grid
  sideBar={sidebar}
  columnChooserSettings={columnChooserSettings}
  reorderSettings={reorderSettings}
  groupSettings={{ enabled: true }}
  filterSettings={{ enabled: true, type: 'CheckBox' }}
  modules={modules}
/>
```

## Filters panel

`sideBar.toolPanels: ['filters']` shows every filterable column in one stack — a single place to view, apply, and edit the entire filter set. Per-column `filter={{ type: 'Menu' | 'Excel' }}` overrides the global filter type for the panel row.

```tsx
const sidebar: SideBar = { enabled: true, toolPanels: ['filters'] };

<Column field="ProductName" headerText="Product Name" filter={{ type: 'Menu' }} />
<Column field="UnitPrice"   headerText="Unit Price"   filter={{ type: 'Excel' }} />
```

## Editing panel

`sideBar.toolPanels: ['editing']` exposes editable fields for the **currently selected rows** and applies bulk updates from a single panel. Pair with `selectionSettings.mode = 'Multiple'` for multi-row edits:

```tsx
const sidebar: SideBar = { enabled: true, toolPanels: ['editing'] };

const [editSettings] = useState<EditSettings>({
  allowEdit: true, allowAdd: true, allowDelete: true,
});

<Grid
  selectionSettings={{ enabled: true, mode: 'Multiple' }}
  sideBar={sidebar}
  editSettings={editSettings}
  modules={{ EditModule, EditToolPanelModule }}
/>
```

Editor types in the panel follow each `<Column edit={{ type: EditType.* }}>` config.

## Custom tool panels

Define a custom panel as an object with `id`, `label`, and `content`:

```tsx
const customPanel = {
  id: 'salesSummary',
  label: 'Sales Summary',
  content: {
    render: () => (
      <div style={{ padding: '15px', whiteSpace: 'nowrap' }}>
        <h2>Sales Summary</h2>
        <p>Total Units Sold: <b>{totals.units.toLocaleString()}</b></p>
        <p>Total Revenue: <b>{totals.revenue.toLocaleString()}</b></p>
      </div>
    ),
  },
};

const sidebar: SideBar = {
  enabled: true,
  toolPanels: [customPanel, 'columns', 'filters', 'editing'],
  defaultToolPanel: 'salesSummary',
};
```

You can also pass `content: { content: <span>Static text</span> }` (eagerly rendered) instead of `content.render()` for purely static content. Use `render` when the panel must reach `gridRef` (e.g., a "Clear Filters" button calling `gridRef.current?.clearFilter()`).

## Mixed built-in + custom

```tsx
const clearFiltersPanel = {
  id: 'clearFilters',
  label: 'Clear Filters',
  content: {
    render: () => (
      <div style={{ padding: '20px' }}>
        <Button onClick={() => gridRef.current?.clearFilter()}>
          Clear All Filters
        </Button>
      </div>
    ),
  },
};

const sidebar: SideBar = {
  enabled: true,
  toolPanels: ['columns', 'filters', 'editing', clearFiltersPanel],
  defaultToolPanel: 'filters',
  openByDefault: true,
};
```

## Constraints & guardrails

- **Missing panel module** is the silent failure — `toolPanels: ['filters']` without `FilterToolPanelModule` produces a side bar with an empty "filters" tab.
- **Editing panel needs selection**: without `selectionSettings` enabling row selection, the editing panel renders but has nothing to edit.
- **Custom panel side effects**: a `content.render` callback that fires imperative grid mutations (e.g., `clearFilter`, `refresh`) must be stable across renders; export the `gridRef` to the closure carefully so panel contents don't capture stale references.
- **Position change** (`left` vs `right`) requires re-laying out surrounding CSS — the panel occupies grid width just like a column.
- **Integration check**: enabling `'filters'` + per-column `filter.type: 'Menu'` overrides the global filter type **inside the panel** — verify the override matches the data type (`Menu` for strings, `Excel` for numeric lists, etc.).
