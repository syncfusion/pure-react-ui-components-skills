---
name: tree-data
description: Hierarchical Tree Data display in the Syncfusion React Data Grid — `isTreeMode`, nested vs self-referential data binding, tree-column expander UI, sorting/filtering that preserves parent-child relationships, and aggregates inside the tree grid. Load when displaying hierarchical data, building org-chart style tables, rendering file-folder structures, or enabling multi-level drill-down.
---

# Tree Data (hierarchical grid)

The Tree Grid enables parent-child record relationships inside the tabular surface, with expandable rows driven by a designated tree column. Render hierarchical data either nested (children inline on the parent row) or self-referential (flat dataset with parent-id references).

`TreeDataModule` is required. Without it, `isTreeMode: true` produces an empty grid.

## Enabling tree data

```tsx
import { Grid, Columns, Column, TreeDataModule } from '@syncfusion/react-grid';

const modules = { TreeDataModule };

<Grid
  dataSource={hierarchicalData}
  isTreeMode={true}
  modules={modules}
  treeColumnIndex={1}                  // column index whose row renders the expander UI
  treeDataChildrenField="subDepartments"
/>
```

| Property | Purpose |
|---|---|
| `isTreeMode` | Switches the grid to hierarchical rendering. |
| `treeColumnIndex` | Zero-based column index that hosts the expand / collapse caret. |
| `treeDataChildrenField` | Field name that contains the child array (nested mode). |
| `treeDataIdMapping` | Field name that uniquely identifies each record (self-referential mode). |
| `treeDataParentIdField` | Field name holding the parent record identifier (self-referential mode). |

> Add `isPrimaryKey={true}` to a single column whenever a tree grid also supports editing, selection persistence, or any imperative mutation API.

## Nested data binding

Use when the parent row already owns its child array.

```ts
type Unit = {
  unitId: string;
  unitName: string;
  departmentType: string;
  subDepartments?: Unit[];
};

const data: Unit[] = [
  {
    unitId: 'HSP-1000',
    unitName: 'Metropolitan Medical Center',
    departmentType: 'Hospital',
    subDepartments: [
      { unitId: 'DEP-1001', unitName: 'Emergency Department', departmentType: 'Department',
        subDepartments: [
          { unitId: 'UNT-1002', unitName: 'Critical Care Unit',  departmentType: 'Specialized Unit' },
          { unitId: 'UNT-1003', unitName: 'Diagnostic Center',    departmentType: 'Specialized Unit' },
        ],
      },
      /* … */
    ],
  },
];
```

`treeDataChildrenField="subDepartments"` recursively walks the structure. The `treeColumnIndex` controls which column displays the expander icon.

## Self-referential data binding

Use when records are stored flat with parent-id references.

```ts
const data = [
  { catalogId: 1, parentId: null, productName: 'Electronics',              /* … */ },
  { catalogId: 2, parentId: 1,    productName: 'Electronics Consumer Devices', /* … */ },
];

<Grid
  dataSource={data}
  isTreeMode={true}
  modules={modules}
  treeColumnIndex={1}
  treeDataIdMapping="catalogId"
  treeDataParentIdField="parentId"
/>
```

The grid auto-builds the hierarchy from the IDs. Sort-friendly record lookup uses `treeDataIdMapping` as the row identifier; resolve duplicates before sending data to the grid.

## Tree-grid filtering

Filter bar / menu / Excel / CheckBox modes all work with tree data. Sort by the filtered column, but parent-child expansion is preserved automatically.

```tsx
const [filterSettings] = useState<FilterSettings>({ enabled: true, type: 'CheckBox' });

<Grid
  filterSettings={filterSettings}
  isTreeMode={true}
  treeColumnIndex={2}
  treeDataChildrenField="subtasks"
  modules={{ TreeDataModule, FilterModule, PagerModule }}
/>
```

Per-column filter bar type is supported via `filter.filterBarType`:

| Column | `filter.filterBarType` |
|---|---|
| text | `FilterBarType.TextBox` (default) |
| number | `FilterBarType.NumericTextBox` |
| date | `FilterBarType.DatePicker` |

For `FilterBarType.TextBox`, expressions like `>10`, `=Pending`, `*Lead*` apply. `enableFilterBarOperator: true` adds an operator dropdown to each filter bar cell.

## Tree-grid sorting

```tsx
<Grid
  sortSettings={{ enabled: true }}
  isTreeMode={true}
  modules={{ TreeDataModule, PagerModule }}
/>
```

Multi-sort: hold <kbd>Ctrl</kbd> while clicking sortable headers; <kbd>Shift</kbd>+click removes sort from that column.

Programmatic sort:

```tsx
gridRef.current?.sortByColumn('taskName', 'Ascending');
gridRef.current?.clearSort();
gridRef.current?.removeSortColumn('taskName');
```

Custom comparer for priority ordering (e.g., Low → Medium → High → Critical):

```ts
const sortComparer = (ref: ValueType, cmp: ValueType): number => {
  const order: Record<string, number> = { Critical: 4, High: 3, Medium: 2, Low: 1 };
  return (order[String(ref)] ?? 0) - (order[String(cmp)] ?? 0);
};

<Column field="priority" headerText="Priority" sortComparer={sortComparer} />
```

## Tree-grid aggregates

The tree grid honours `<Aggregates>` blocks with `AggregateModule` enabled. Aggregate types and templates work the same way as on flat grids:

```tsx
<Aggregates>
  <AggregateRow>
    <AggregateColumn field='TotalCosts' type={AggregateType.Sum}
                     footerTemplate={footerSum} format='C2' />
    <AggregateColumn field='TotalCosts' type={AggregateType.Min}
                     footerTemplate={footerMin} format='C2' />
    <AggregateColumn field='TotalCosts' type={AggregateType.Max}
                     footerTemplate={footerMax} format='C2' />
  </AggregateRow>
</Aggregates>
```

Multiple `<AggregateRow>` allow several aggregate types per column. `AggregateType.Custom` lets you provide `customAggregate(data, column)` for non-standard calculations:

```ts
const customAverage = (data: object[] | { result: object[] }, column: AggregateColumnProps) => {
  const items = Array.isArray(data) ? data : (data as { result: object[] }).result ?? [];
  return items.filter((it) => it[column.field as keyof T] === 'Approved').length;
};

<AggregateColumn field="approved" type={AggregateType.Custom} customAggregate={customAverage} />
```

## Tree grid with paging

Combine with `PagerModule`:

```tsx
const [pageSettings] = useState<PageSettings>({ enabled: true, pageSize: 20, pageCount: 4 });

<Grid
  pageSettings={pageSettings}
  isTreeMode={true}
  modules={{ TreeDataModule, PagerModule }}
/>
```

## Constraints & guardrails

- **Missing tree module** is the most common silent failure when `isTreeMode={true}` is set but `TreeDataModule` is not in the `modules` map — the grid renders empty.
- **isPrimaryKey** still matters for tree grids when editing, persistent selection, or `setCellValue` / `setRowData` are in use.
- **Filtering and sorting** preserve parent-child grouping: collapsing a parent hides filtered children only if the parent doesn't match. Use `treeColumnIndex` to choose which column exposes the expand/collapse UI.
- **Aggregates on tree mode** are computed over the data source as a whole; they are *not* per-group summaries. For per-group totals use the regular grouping flow (`groupSettings` + `GroupType.GroupRows`) instead.
- **Self-referential orphan rows**: ensure every `parentId` (except for root rows where `parentId === null`) matches an existing `treeDataIdMapping` value. Orphans render at the root and warn in `onError`.
- **Integration** check: when adding `PinningModule` to a tree grid, pin the row inside the flat visible list — pinning by tree-level grouping is not supported.
