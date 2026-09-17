---
name: master-detail
description: Master/Detail grids for the Syncfusion React Data Grid — detailRowTemplate (single-tier simple case), detailCellRendererParams (relational mapping + async getDetailRowData + maxNestingDepth + nested master/detail), defaultExpandedRows, and integration with DetailGridModule. Load when building expandable detail rows, hierarchical records (order → line items), or relational child datasets via mappingID.
---

# Master/Detail

The grid supports expandable detail rows: parent rows that, when clicked, reveal a child grid (or any custom content). Two patterns:

1. **`detailRowTemplate`** — any React content; simple to use; every detail row is identical structure.
2. **`detailCellRendererParams`** — relational binding with `mappingID`, async loading via `getDetailRowData`, nested master/detail support via `maxNestingDepth`.

`DetailGridModule` is **required** for both.

```tsx
import { Grid, DetailGridModule } from '@syncfusion/react-grid';

const modules = { DetailGridModule };

<Grid dataSource={parents} detailRowTemplate={...} modules={modules} ... />
```

## Pattern A — `detailRowTemplate`

```tsx
<Grid
  dataSource={orders}
  isMasterDetail
  detailRowHeight="300px"                  // required for stable scroll math
  defaultExpandedRows={[1]}
  detailRowTemplate={(params: any) => (
    <Grid dataSource={params.row.transactions}
          columns={transactionColumns}
          width="100%" height={250} enableDevMode={false} />
  )}
  modules={modules}
/>
```

Common UX:
- Render a nested grid bound to a parent row's children.
- Render images/charts/HTML for visual detail rows.
- Render multiple detail grids side-by-side.

Pattern specifics:
- Each detail row builds an **independent grid instance** — own `dataSource`, `columns`, `modules`, `height`.
- `detailRowHeight` defaults to `'300px'`. Without a defined height, the grid can't adjust scroll positions when rows expand.
- Nested cascade: the child grid can itself declare `isMasterDetail` with its own `detailRowTemplate`. There's no inherent depth limit but practical limits come from performance.

Limitations:
- Selection, editing, paging, sorting, filtering may be limited/unsupported depending on the detail content (if it's not grid-shaped).
- Keep the detail row template non-interactive at the parent level when the detail content needs its own focus (or set `selectionSettings={{ enabled: false }}` and `enableAltRow={false}` at the parent).

## Pattern B — relational binding with `detailCellRendererParams`

Use this when child data lives in an external dataset and parents reference children via a foreign key.

```tsx
import { GetDetailRowDataParams, type DetailCellRendererParams } from '@syncfusion/react-grid/src/grid/types/detail-cell-renderer.interfaces';

const detailParams: DetailCellRendererParams<Parent> = {
  mappingID: 'caseId',                                 // parent field matching child key
  childDataSource: mappedMilestones,                   // external dataset of children
  detailGridOptions: {
    columns: childColumns,
    isMasterDetail: true,
    modules,
    maxNestingDepth: 2,
    currentNestingDepth: 1,
    sortSettings: { enabled: true },
    filterSettings: { enabled: true, type: 'Menu' },
    defaultExpandedRows: [1],
    detailCellRendererParams: grandchildParams,        // optional: nest again
  },
  getDetailRowData: (params: GetDetailRowDataParams<Parent>) => {
    const parentId = (params.data as Parent).caseId;
    const childRows = (params.childDataSource as Child[]).filter(c => c.caseId === parentId);
    params.successCallback(childRows);
  },
};

<Grid dataSource={parents}
      isMasterDetail
      detailCellRendererParams={detailParams}
      modules={modules} />
```

### Shape

`DetailCellRendererParams<T>`:

| Field | Type | Purpose |
|---|---|---|
| `detailGridOptions` | `GridProps` | Configuration for the nested detail grid. Accepts the same shape as `<Grid>` props (`columns`, `modules`, `height`, `isMasterDetail`, `maxNestingDepth`, `currentNestingDepth`, `sortSettings`, `filterSettings`, `defaultExpandedRows`, `detailCellRendererParams`). |
| `getDetailRowData` | `(params: GetDetailRowDataParams<T>) => void` | Caller **must** invoke `params.successCallback(rows)` when child rows are resolved. |
| `mappingID` | `string` | Foreign-key field name in the parent that matches the child. |
| `childDataSource` | `object[]` | External child dataset. |
| `onDetailGridCreated` | `(gridRef: any) => void` | Fires when the nested grid is created. `gridRef.parentData`, `gridRef.id`. |
| `onDetailGridDestroyed` | `(gridRef: any) => void` | Fires when the nested grid is removed. |

`GetDetailRowDataParams<T>`:

| Field | Description |
|---|---|
| `data` | Parent row data. |
| `childDataSource` | External child dataset (when provided). |
| `mappingID` | The mapping key (when provided). |
| `successCallback(rows)` | Call with the resolved child rows to populate the detail. |

### Async support

`getDetailRowData` can perform a fetch and call `successCallback(rows)` when ready:

```ts
getDetailRowData: (params) => {
  fetch(`/api/cases/${params.data.caseId}/milestones`)
    .then(r => r.json())
    .then(rows => params.successCallback(rows));
}
```

### Nested cascades (`maxNestingDepth`)

`detailGridOptions.maxNestingDepth` caps how deeply nested grids can recurse. `currentNestingDepth` is informational on each child. Past the cap, the child grid ignores nested master/detail.

### Events

| Event | Fires when |
|---|---|
| `onDetailGridCreated` | Nested detail grid instance was created (returns `gridRef`). |
| `onDetailGridDestroyed` | Nested detail grid was destroyed (after collapse or grid unload). |

`onDataLoad` and `onRowSelect` work the same way on detail grids as on top-level grids.

### Common pattern: aggregates on detail rows

```ts
getDetailRowData: (params) => {
  const rows = ...
  if (rows.length) {
    /* attach a customAggregate to grid via getDetailRowData context if needed */
  }
  params.successCallback(rows);
}
```

`customAggregate(data, column)` is the path for `AggregateType.Custom` aggregates.

## Constraints & guardrails

- **`detailRowHeight`** is **required** for stable scroll math when expanding rows. Default `300px` works in most cases.
- **`getDetailRowData` must call `successCallback`** — failing to do so leaves the detail row empty.
- **Relational binding** does not require a primary key, but you still need `mappingID` or your own resolution logic in `getDetailRowData`.
- **Nested master detail** with `maxNestingDepth` keeps grid instances independent and isolated; performance scales with how many details are expanded simultaneously.
- **Integration**: when adding details, register `DetailGridModule`, declare `isMasterDetail`, define `detailRowHeight`, and `defaultExpandedRows` if some should be open by default. Forgetting the registered module is the most common silent failure.
- **Guardrail: cascading deletes** — if the parent grid has edits/deletes that should propagate, code the propagation in `onDataChangeStart`; the detail grid does not auto-cascade.