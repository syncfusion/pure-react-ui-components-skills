---
name: selection
description: Selection configuration for the Syncfusion React Data Grid — row selection (single/multiple/toggle), cell selection (Flow/Box/BoxWithBorder), checkbox column with persistSelection, AutoSelectMode ('Default' / 'Intermediate'), conditional selection via isRowSelectable, and onRowSelect/onRowDeselect events. Load when enabling single/multiple row selection, building an Excel-style cell range, adding a select-all checkbox column, making some rows unselectable, or controlling how checkbox state interacts with remote delete payloads.
---

# Selection

Four overlapping patterns: **row**, **cell**, **checkbox**, **conditional**. The grid ships sensible defaults but the persistence and remote-payload semantics are subtle — wire them deliberately with a primary key.

## `SelectionSettings`

```ts
{
  enabled: boolean;                         // default true
  mode: 'Single' | 'Multiple';              // default 'Single'
  type?: 'Row' | 'Cell';                    // default 'Row'
  enableToggle?: boolean;                   // default false
  checkboxOnly?: boolean;                   // default false
  persistSelection?: boolean;               // default false (auto true when checkbox column rendered)
  cellSelectionType?: 'Flow' | 'Box' | 'BoxWithBorder';   // default 'Flow' (Cell only)
  autoSelectMode?: 'Default' | 'Intermediate';            // default 'Default' (checkbox)
}
```

```tsx
import { Grid, SelectionSettings, SelectionType } from '@syncfusion/react-grid';

const [sel] = useState<SelectionSettings>({
  enabled: true,
  mode: 'Multiple',
  type: SelectionType.Row,
  persistSelection: true,
  enableToggle: true,
});

<Grid selectionSettings={sel} ... />
```

## Row selection

### Single (default)

Click selects; clicking the same row keeps it selected unless `enableToggle: true`.

### Multiple

- Click selects the focused row.
- `Space` selects the focused row.
- `Ctrl` (or `⌘` on macOS) + click toggles a row in the selection.
- `Shift` + click selects the range from the last focus cell to the clicked row.

### Programmatic

```tsx
gridRef.current?.selectRow(index: number);
gridRef.current?.selectRows([1, 3, 7]);              // requires mode 'Multiple'
gridRef.current?.clearRowSelection([2, 5]);
gridRef.current?.clearSelection();
gridRef.current?.getSelectedRecords();
gridRef.current?.getSelectedRowIndexes();
```

`getSelectedRecords` returns:

- **Local data** + `persistSelection: true`: `T[]` (all records across pages).
- **Local data** + `persistSelection: false`: `T[]` (current page only).
- **Remote data**: metadata object:
  - `{ isSelectAll: true, primaryKeys: [] }` — delete all.
  - `{ isSelectAll: true, primaryKeys: [<excluded keys>] }` — delete all except excluded.
  - `{ isSelectAll: false, primaryKeys: [<keys>] }` — delete specific.
  - `{ isSelectAll: false, primaryKeys: [<loaded page keys only>] }` — intermediate mode.

### Events

| Event | Args |
|---|---|
| `onRowSelect` | `selectedRowIndex`, `selectedRowIndexes`, `selectedCurrentRowIndexes`, `data` |
| `onRowDeselect` | `deSelectedRowIndex`, `deSelectedRowIndexes`, `deSelectedCurrentRowIndexes` |

```tsx
const onRowSelect = useCallback((args: RowSelectEvent) => {
  console.log('Selected', args.data, 'indexes', args.selectedRowIndexes);
}, []);
```

### `enableToggle`

- `Multiple` + `enableToggle`: click a selected row → clears others; Ctrl/click adds/removes; Shift/click selects range.
- `Single` + `enableToggle`: same row click toggles selection.

### CSS hook

- `.sf-grid .sf-active` — currently selected row.

## Cell selection

```tsx
const [sel] = useState<SelectionSettings>({
  type: SelectionType.Cell,
  mode: 'Multiple',
  cellSelectionType: 'BoxWithBorder',
});
<Grid selectionSettings={sel} ... />
```

Interactions:
- Click → selects single cell.
- Click + drag → continuous range (`Flow`).
- `<Ctrl | ⌘>` + click / `Shift` + click → multi-cell.

`cellSelectionType`:

| Value | Effect |
|---|---|
| `'Flow'` (default) | Range selection across rows/cols; intermediate cells included. |
| `'Box'` | Rectangular range, block-bounded. |
| `'BoxWithBorder'` | Rectangular range with visible border. |

Programmatic: `gridRef.current?.clearCellSelection()`. CSS hook: `.sf-cell.sf-cell-selected`.

**Requires `isPrimaryKey`** for accurate tracking.

## Checkbox selection

Add a row checkbox column with header select-all:

```tsx
<Column type="checkbox" width="40" />
<Column field="ContractID" headerText="Contract ID" isPrimaryKey />
```

Default behavior:
- Selects `Multiple` mode.
- `persistSelection` flipped to `true` automatically.
- Header select-all checkbox unless `headerCheckbox={false}` on the column.

`<Column type='checkbox' headerCheckbox={false} />` for a checkbox column without a header checkbox.

`AutoSelectMode` enum:

| Value | Effect |
|---|---|
| `'Default'` | Standard select-all. Operates over the full dataset. |
| `'Intermediate'` | Select-all applies only to currently loaded pages. |

### Remote delete payload patterns (server contract)

| Operation | Payload |
|---|---|
| Delete all rows | `{ isSelectAll: true, primaryKeys: [] }` |
| Delete all except unselected | `{ isSelectAll: true, primaryKeys: [<excluded keys>] }` |
| Delete specific | `{ isSelectAll: false, primaryKeys: [<keys>] }` |
| Intermediate mode | `{ isSelectAll: false, primaryKeys: [<loaded keys only>] }` |

These are the documented payloads — coordinate with the server's delete handler so it interprets them consistently.

## Conditional selection (`isRowSelectable`)

```tsx
const canSelect = (row: T) => row.Status === 'Canceled' || row.Status === 'Delivered'
  ? false
  : true;

<Grid isRowSelectable={canSelect} ... />
```

Pair with `{ selectable: false, showDisabledCheckboxes: false }` to selectively hide checkboxes:

```ts
type RowSelectableParams = { selectable: boolean; showDisabledCheckboxes?: boolean };

isRowSelectable = (row) =>
  row.Status === 'Canceled' || row.Status === 'Delivered'
    ? { selectable: false, showDisabledCheckboxes: false }
    : true;

// Behavior matrix:
// selectable=false + showDisabledCheckboxes=true  → checkbox rendered, disabled
// selectable=false + showDisabledCheckboxes=false → checkbox hidden
// selectable=true (or return true)               → selectable
```

`isRowSelectable` runs on every row pre-render — keep it pure and cheap.

## Common patterns

| Need | Configuration |
|---|---|
| Single product selection | `selectionSettings={{ mode: 'Single' }}` |
| Excel-like multi-row range | `selectionSettings={{ mode: 'Multiple' }}` (default) |
| Cell range like Excel | `selectionSettings={{ type: SelectionType.Cell, cellSelectionType: 'BoxWithBorder' }}` |
| Toggling in/out the same row | `selectionSettings={{ enableToggle: true }}` |
| Bulk action header | `<Column type='checkbox' />` + `persistSelection: true` (auto) |
| Tiered delete (only loaded pages) | `selectionSettings={{ autoSelectMode: 'Intermediate' }}` |
| Some rows unselectable | `isRowSelectable={row => row.status !== 'Locked'}` |

## Accessibility

- Selection changes update `aria-selected` on rows/cells.
- Keyboard: arrows move focus; `Shift+↑/↓` extends selection; `Esc` clears selection.

## Constraints & guardrails

- **`isPrimaryKey` is required** for any persistent selection (`persistSelection: true` or checkbox column).
- **`type: SelectionType.Cell`** changes the model — methods like `selectRow` won't apply; use `clearCellSelection`.
- **`getSelectedRecords`** returns different shapes for local vs remote data; consumers must defensively check the type.
- **`checkboxOnly: true`** disables click-to-select; only checkbox clicks select. Combined with `enableToggle`, this can confuse users — pick one metaphor.
- **`persistSelection: true`** without `isPrimaryKey` will produce stale selection state across paged data — guard with a primary key assertion.
- **Integration**: when wiring selection to a server delete, ensure the documented payload contract is built exactly into the request body; mismatch is the most common silent failure.