---
name: pivot
description: Pivot view inside the Syncfusion React Data Grid — `pivotSettings` with rows / columns / values, `memberSorts`, `showSubTotals`, `showGrandTotals`, `defaultExpanded`, `customizePivotColumnGroup` / `customizePivotColumn` callbacks, and the column-menu integration for interactive sorting/expansion. Load when building pivot-style summaries, drill-down reporting tables, multi-level aggregation dashboards, or `customize*` overrides on pivot headers and result columns.
---

# Pivot view

Pivot view turns the grid into an interactive summary surface. Configure `rows`, `columns`, and `values` to shape the layout — the rest of the grid (sorting, resizing, column menu, export) continues to work on the generated result set.

`PivotModule` is required.

## Enabling pivot view

```tsx
import { Grid, Columns, Column, PivotSettings, PivotModule } from '@syncfusion/react-grid';

const pivotSettings: PivotSettings<T> = {
  enabled: true,
  rows:    ['Year', 'HalfYear', 'Quarter'],
  columns: ['EnergyType', 'EnergySource'],
  values: [
    { field: 'PowUnits', type: 'Sum' },
    { field: 'ProCost',  type: 'Sum' },
  ],
  showSubTotals:    true,
  showGrandTotals:  true,
  defaultExpanded:  true,
};

<Grid<T>
  dataSource={data}
  modules={{ PivotModule }}
  pivotSettings={pivotSettings}
  sortSettings={{ enabled: true }}
  columnMenuSettings={{ enabled: true }}
/>
```

`PivotSettings<T>` shape:

```ts
{
  enabled: boolean;
  rows?: string[];                                                            // field names for row groups (top to bottom)
  columns?: string[];                                                         // field names for column groups (left to right)
  values?: Array<{ field: string; type: 'Sum' | 'Average' | 'Min' | 'Max' | 'Count' | 'Product' | string }>;
  showSubTotals?: boolean;
  showGrandTotals?: boolean;
  defaultExpanded?: boolean;                                                  // expand all groups on first render
  memberSorts?: Array<{ field: string; direction: 'Ascending' | 'Descending' }>;
}
```

## Pivot result columns

Pivot result columns are generated from `pivotSettings.values` combined with `pivotSettings.columns`. Each (column × value) combination produces a result column automatically. Single-level hierarchies use one field in `columns`; multi-level hierarchies use more:

```tsx
const pivotSettings: PivotSettings<T> = {
  enabled: true,
  rows:    ['Department'],
  columns: ['Quarter', 'Region'],                                   // two-level column hierarchy
  values: [
    { field: 'Budget',          type: 'Sum' },
    { field: 'ActualCost',      type: 'Sum' },
    { field: 'Variance',        type: 'Sum' },
    { field: 'TransactionCount', type: 'Sum' },
  ],
  showSubTotals: true,
  showGrandTotals: true,
};
```

## Customizing result columns

`customizePivotColumn(context)` runs once per generated result column. The `context.measure` exposes the source `field` and `type` so per-measure overrides (width, format, header text, template) are decoded cleanly:

```tsx
const styleResultColumns = (
  context: PivotColumnContext<T>
): PivotColumnOverrides => ({
  width:
    context.measure?.field === 'PowUnits' ? 180 :
    context.measure?.field === 'ProCost'  ? 230 : undefined,
  headerText:
    context.measure?.field === 'PowUnits' ? '⚡ Energy Units' :
    context.measure?.field === 'ProCost'  ? '💰 Production Cost' : undefined,
  template:
    context.measure?.field === 'PowUnits'
      ? ({ data, column }: ColumnTemplateProps<PivotResultRow>) => (
          <strong>{Number((data as PivotResultRow).cells[column.field!.slice(6)] ?? 0).toLocaleString()}</strong>
        )
      : context.measure?.field === 'ProCost'
      ? ({ data, column }: ColumnTemplateProps<PivotResultRow>) => (
          <strong>{Number((data as PivotResultRow).cells[column.field!.slice(6)] ?? 0).toLocaleString('en-US', { style: 'currency', currency: 'USD' })}</strong>
        )
      : undefined,
});

<Grid<T>
  dataSource={data}
  modules={{ PivotModule, FilterModule }}
  pivotSettings={pivotSettings}
  customizePivotColumn={styleResultColumns}
  columnMenuSettings={{ enabled: true }}
  filterSettings={{ enabled: true, type: 'CheckBox' }}
/>
```

`PivotColumnOverrides` accepted fields: `width`, `headerText`, `headerTemplate`, `textAlign`, `template`, `format`, `visible`.

## Customizing pivot column groups

`customizePivotColumnGroup(context)` runs while column *group* headers are rendered (i.e., the per-field-name hierarchical headers above the result columns). Return a partial overrides object to control header text, templates, alignment, attributes:

```tsx
type PivotColumnGroupContext<T> = {
  field: string;
  value: string | number;
  /* … */
};

type PivotColumnGroupOverrides = {
  headerTemplate?: React.ReactNode;
  /* … */
  [key: string]: unknown;
};

const customizeColumns = (
  context: PivotColumnGroupContext<T>
): PivotColumnGroupOverrides | undefined =>
  context.field === 'Quarter'
    ? { headerTemplate: (<div className="sf-quarter-header">Financial Quarter: {String(context.value)}</div>) }
    : context.field === 'Region'
    ? { headerTemplate: (<div className="sf-region-header">Reporting Region: {String(context.value)}</div>) }
    : undefined;

<Grid<T>
  dataSource={data}
  modules={{ PivotModule }}
  customizePivotColumnGroup={customizeColumns}
  pivotSettings={pivotSettings}
/>
```

## Sorting pivot groups

`memberSorts` configures initial ordering of pivot row / column groups:

```tsx
const pivotSettings: PivotSettings<T> = {
  enabled: true,
  rows: ['Department'],
  columns: ['Quarter', 'Region'],
  values: [/* … */],
  memberSorts: [{ field: 'Department', direction: 'Descending' }],
};
```

Change sort at runtime by mutating `memberSorts` and re-rendering. Interactive sorting through the column menu: enable `columnMenuSettings` and pick `Sort Ascending` / `Sort Descending` / `Clear Sort` from a pivot result column. Use `Expand All Row Groups` / `Collapse All Row Groups` for navigation.

## Subtotals and grand totals

| Flag | Effect |
|---|---|
| `showSubTotals: true` | Renders subtotal rows / columns at every intermediate pivot group. |
| `showGrandTotals: true` | Renders grand total rows / columns at the pivot root. |

Toggle both via state when wiring a property pane. The Hidden API is the same flag pair passed through `pivotSettings`.

## Default expansion

`pivotSettings.defaultExpanded: true` expands all row and column groups on first render; otherwise groups render collapsed and the user clicks them. Interactive: enable `columnMenuSettings.enabled` and use `Expand All Row Groups` / `Collapse All Row Groups`.

## Hide expand/collapse indicators

Hide the inline caret icons via CSS (useful when expansion is driven programmatically):

```css
.sf-pivot-view .sf-pivot-expand,
.sf-pivot-view .sf-pivot-collapse {
  display: none;
}
```

## Tree Data + Pivot integration

Render pivot-style metrics on top of a tree-data source by combining `TreeDataModule`, `AggregateModule`, and a hidden summary row. The pivot engine itself operates on rows/values/columns directly — when the project both pivots and shows hierarchy, prefer **pivot with row groups** or **tree grid with aggregate footer**, not both simultaneously. The two are complementary modes chosen by data shape:

- Use **Tree Data** when the relationship is naturally hierarchical (org chart, file system, BOM).
- Use **Pivot** when the relationship is relational and the summary is multi-dimensional (slice by Quarter × Region × ProductCategory).

## Constraints & guardrails

- **Pivot result columns are generated at runtime** — do not declare static `<Column field="result:...">` rows for them; let the engine render. Static `<Columns>` declarations continue to control source columns (rows, source-axis fields) that should appear in the grid.
- Pivot is **single-pane**: a grid either renders pivot or regular grid — you cannot pivot part of a `<Columns>` and leave the rest flat.
- Aggregation happens server-side when bound to `DataManager`. Local data computes aggregates in the browser using the provided `values.type`.
- Pivot groups do not survive a column reorder that moves source-axis columns across fixed row axis templates — design the pivot first and pin source columns with `allowReorder={false}` if stability matters.
- **Integration check**: `customizePivotColumn` runs once per result column at render. Memoize the callback or it returns a fresh `headerTemplate` reference per render and triggers full pivot rerenders.
