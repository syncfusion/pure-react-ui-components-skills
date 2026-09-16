---
name: rows
description: Row-level configuration for the Syncfusion React Data Grid — row class, alt rows, fixed row height vs dynamic, row template, row numbers, row drag-and-drop, and row spanning. Load when the user asks for an entire row to render as a custom structure, requests drag-and-drop reordering across rows or grids, asks for visible row indices, or wants dynamic row heights.
---

# Rows

The grid treats rows as first-class: row-level styling, fixed/dynamic heights, embedded row templates, visible row numbering, drag-and-drop reordering across grids, and row spanning (vertical cell merging).

## Row-level settings on `<Grid>`

| Prop | Type | Default | Purpose |
|---|---|---|---|
| `rowClass` | `(props?: RowClassProps) => string` | — | Conditional CSS class per row. |
| `enableAltRow` | `boolean` | `true` | Adds `.sf-altrow` on alternating rows. Default styling is none — provide your CSS. |
| `rowHeight` | `number \| string` | — | Uniform static height for every row. Best for performance with large datasets. |
| `getRowHeight` | `(rowInfo: RowInfo) => number` | — | Per-row dynamic height. Called on initial render. |
| `autoHeight` (column) | `boolean` | `false` | Per-column auto measure. Disables column virtualization. |
| `enableHover` | `boolean` | `true` | Hover highlight. Override CSS: `.sf-grid .sf-grid-content-row:hover .sf-cell`. |
| `selectionSettings` | — | — | Per-row selection. |
| `rowNumberSettings` | `RowNumberSettings` | `{ enabled: false }` | Show leading row-number column. |
| `dragAndDropSettings` | `DragAndDropSettings` | — | Cross-grid or within-grid drag. |
| `isMasterDetail` | `boolean` | `false` | Enable expandable detail rows (see `references/master-detail.md`). |
| `detailRowTemplate` `(params) => ReactElement` | — | — | Detail row content (see master-detail). |
| `defaultExpandedRows` | `number[]` | `[]` | Indexes of master rows pre-expanded. |
| `rowTemplate` | `(data: T) => React.ReactElement` | — | Replace the entire row's content. |

## Row class (`rowClass`)

```tsx
const rowClass = (props?: RowClassProps) =>
  (props?.data as Record)?.priority === 'Critical' ? 'critical-row' : '';
<Grid rowClass={rowClass} />
```

`RowClassProps`: `{ data: T }`. Combine with `enableAltRow={false}` to avoid class collisions.

## Alt rows (`enableAltRow`)

```tsx
<Grid enableAltRow />
```

```css
.sf-grid .sf-altrow { background: rgba(0, 0, 0, 0.04); }
```

## Fixed vs dynamic row height

```tsx
// Fixed — best perf, required for stable virtualization & streaming
<Grid rowHeight={36} />

// Dynamic — per-row measurement via callback
const getRowHeight = useCallback((info: RowInfo) => {
  const row = info.data as IssueData;
  return row.priority === 'Critical' ? 60
       : row.priority === 'High'     ? 50
       : 36;
}, []);
<Grid getRowHeight={getRowHeight} />

// Per-column auto (still disables column virtualization)
<Column field="description" autoHeight template={descriptionTemplate} />
```

**Trade-offs:**
- Fixed `rowHeight` is the best practice for large data + virtualization + streaming.
- `getRowHeight` causes the scrollbar size to change as heights change; rows may shift on revisit.
- `autoHeight: true` on a column disables column virtualization and degrades scroll perf.

## Row template (`rowTemplate`)

`rowTemplate` replaces the body row's content entirely. Each `Column` must still define `field` (sort/filter/edit still target the cell model).

```tsx
const rowTemplate = (props: GadgetsPurchase) => (
  <tr className="template-row">
    <td>{props.id}</td>
    <td>{props.product}</td>
    <td><Chart id={`chart-${props.id}`} height='80px'><ChartSeries .../></Chart></td>
  </tr>
);

<Grid rowTemplate={rowTemplate} width="auto" allowKeyboard={false}
       selectionSettings={{ enabled: false }} enableHover={false} />
```

Limitations:
- Each `Column` requires an accurate `field`.
- Selection, editing, paging, sorting, filtering may be limited if the template HTML doesn't mirror a normal row layout.
- Pairing with charts/images often pairs with disabling keyboard and selection to avoid focus traps.

## Row numbers (`rowNumberSettings`)

```tsx
const [rowNumberSettings] = useState<RowNumberSettings>({ enabled: true });

<Grid rowNumberSettings={rowNumberSettings} />
```

Indices reflect the current view (changes on sort/filter/group/page). Often combined with `showColumnChooser` and `toolbar={['ColumnChooser']}` + `ColumnChooserModule`.

## Row drag-and-drop (`dragAndDropSettings`)

```tsx
const drag = useState<DragAndDropSettings>({ enabled: true, targetID: 'destination-grid' })[0];

<Grid id="source-grid" dragAndDropSettings={drag} />
<Grid id="destination-grid" />
```

- Within a single grid, leave `targetID` empty.
- Cross-grid: set `targetID` to the destination grid's `id`.
- Combine with `selectionSettings.mode = SelectionMode.Multiple` to drag multiple selected rows.

Events:

| Event | Args | Cancellation |
|---|---|---|
| `onRowDragStart` | `RowDragEventArgs` (`fromIndex`, `data`, `cancel`, `targetGrid?`) | `args.cancel = true` |
| `onRowDrag` | `RowDragEventArgs` (`fromIndex`, `dropIndex`, `data`, `targetGrid?`) | n/a (informational) |
| `onRowDrop` | `RowDragEventArgs` | generally not cancelable after drop |

Programmatic helper: `gridRef.current?.getSelectedRecords()` to read the dragged rows server-side.

## Row spanning (vertical cell merging)

Like column spanning, but vertical:

```tsx
<Grid enableAutoSpan enableHover={false} selectionSettings={{ enabled: false }} gridLines={GridLine.Both} />

<Column field="Employee" rowSpan={({ data, rowIndex, field }) => {
  if (field !== 'Employee') return 1;
  let count = 1;
  for (let i = rowIndex + 1; i < data.length; i++) {
    if (data[i].Employee === data[rowIndex].Employee) count++;
    else break;
  }
  return count;
}} />
```

Combine `rowSpan` and `colSpan` in the same grid for both directions on identical cells.

## Methods affecting rows (`GridRef`)

| Method | Use |
|---|---|
| `getSelectedRecords()` | All selected row objects. |
| `getSelectedRowIndexes()` | Selected indexes. |
| `setCellValue(pk, field, value, triggerEvent?)` | Programmatic edit by primary key. |
| `setRowData(pk, data, triggerEvent?)` | Replace entire row data by primary key. |
| `updateRecord(rowIndex, data)` | Set row values by index (unstable across sort/filter). |
| `refresh()` | Force a re-render. |

`setCellValue` / `setRowData` accept a final boolean to commit to the underlying data source: `triggerEvent=true` also raises `onDataChangeStart` / `onDataChangeComplete`.

## Constraints specific to rows

- `getRowHeight` runs on every row during initial render — keep it cheap (no async work, no heavy computation).
- `autoHeight` on a column disables column virtualization; avoid it for grids over ~1k rows.
- `rowTemplate` requires each `<Column>` to declare a real `field` (sort/filter/edit still target the cell model).
- Row drag-and-drop pairs with `selectionSettings` and `getSelectedRecords` for multi-row drags; primary-key column is recommended for stable reordering.

## Cross-references

- For master/detail grids, see `references/master-detail.md`.
- For virtualization tuning that affects row rendering, see `references/performance-and-scrolling.md`.
- For row drag's interaction with selection, see `references/selection.md`.