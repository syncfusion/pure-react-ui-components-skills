---
name: interactivity
description: Inline interactivity for the Syncfusion React Data Grid — autofill (drag-handle cell copy/series), clipboard (copy/cut/paste + headers), context menu (per-header/per-cell/custom items), and the toolbar (built-in items like Add/Edit/Delete/Search/Column Chooser and hook-based Print/PDF/Excel Export). Load when wiring autofill, building copy/paste, configuring the context menu, or extending the toolbar with custom items.
---

# Interactivity: autofill, clipboard, context menu, toolbar

Inline interactivity features — autofill, clipboard, context menu, and the toolbar — share the `EditModule` (or related) registration and the `GridRef` for programmatic calls. This file covers them in one place so you can choose quickly.

## Autofill

Drag the small autofill handle on a selected cell to copy or extend values across cells. Auto-number series on `Alt+drag`.

### Setup

```tsx
import { Grid, Columns, Column,
         AutoFillSettings, SelectionSettings, SelectionType, EditSettings,
         AutoFillModule, FilterModule, PagerModule, EditModule } from '@syncfusion/react-grid';

const modules = { AutoFillModule, FilterModule, PagerModule, EditModule };

const [autofill] = useState<AutoFillSettings>({
  enabled: true,
  shouldClearOnReduction: false,        // back-drag clears (true) vs blocks (false)
  allowedDirection: 'both',             // 'row' | 'column' | 'both'
  excludeFromAutoFill: (field: string) => field === 'PrimaryKey',
  fillOperation: (args) => {            // custom fill; return false for default
    if (args.fieldName === 'Country') {
      return ['US', 'CA', 'UK', 'DE'][args.cellPosition % 4];
    }
    return false;
  },
});

const [sel] = useState<SelectionSettings>({
  type: SelectionType.Cell,
  mode: 'Multiple',
  cellSelectionType: 'Box',             // required for autofill
});

const [edit] = useState<EditSettings>({ allowEdit: true, allowDelete: true });

<Grid
  dataSource={data}
  autofillSettings={autofill}
  selectionSettings={sel}
  editSettings={edit}
  modules={modules}
>
  <Columns>
    <Column field="id" headerText="ID" isPrimaryKey disableAutofill />
    <Column field="Value" headerText="Value" type={ColumnType.Number} />
    <Column field="Code" headerText="Code" />
  </Columns>
</Grid>
```

### Column-level disable

`Column.disableAutofill={true}` blocks autofill on that column (auto-true for primary-key columns).

### Programmatic (`GridRef.applyFill`)

```ts
gridRef.current?.applyFill({
  direction: 'down' | 'right' | 'up' | 'left',
  fillCount: number,
  sourceCells?: CellRef[],     // uses current selection if omitted
  targetCells?: CellRef[],     // auto-computed if omitted
  isAltFill: boolean,          // alt-drag mode → numeric series / repeat pattern
});
```

`CellRef` shape: `{ rowKey: string | number; fieldName: string; rowIndex: number; columnIndex: number; }`.

### User interactions

| Action | Effect |
|---|---|
| Drag single cell | Copy value. |
| `Alt` + drag single | Numeric series (down/right ascending, up/left descending). |
| Drag multi-cell | Strings repeat; numerics extend linearly. |
| `Alt` + drag multi-cell | Repeats source cells (no linear progression). |
| Double-click handle | Auto-fills toward `allowedDirection`. |

### Events

| Event | Args shape | Use |
|---|---|---|
| `onCellFillStart` | `{ source: { selectedRowCells: [{ rowKey, fieldNames }] }, fillRange: { direction, fillCount }, cancel }` | Cancel to block. |
| `onCellFillComplete` | `{ modifiedRecords, filledCellsDetail: { [cellKey]: { filledValue } } }` | Post-fill reconciliation. |

`fillOperation` args: `{ fieldName, currentValue, fillRange: { direction, fillCount }, cellPosition }`. Return any value or `false`.

### Constraints

- Autofill requires `isPrimaryKey` for accurate updates.
- Primary-key columns are auto-disabled for autofill.
- Validation rules on `Column.validationRules` apply to autofilled values.

## Clipboard

### Settings

```tsx
const [clipboard] = useState<ClipboardSettings>({
  enabled: true,
  allowRowCopy: false,                // true → copies full row(s)
  copyWithHeaders: false,
  allowCut: true,                     // default
  allowPaste: true,                   // default
});
```

```tsx
import { ClipboardModule, EditModule, FilterModule, PagerModule } from '@syncfusion/react-grid';

const modules = { ClipboardModule, EditModule, FilterModule, PagerModule };
<Grid clipboardSettings={clipboard} editSettings={edit} modules={modules} ... />
```

### Programmatic (`GridRef`)

```tsx
gridRef.current?.copyToClipboard();             // copies the current selection
gridRef.current?.pasteFromClipboard(text?);     // tab/newline text matrix; omit to read from system clipboard
gridRef.current?.cutToClipboard();              // programmatic cut (preserves PK)
```

`pasteFromClipboard` with no arg first tries to read system clipboard (browser support varies); with an arg, applies the provided text directly.

### Events

| Event | Args | Use |
|---|---|---|
| `onClipboardCopy` | `clipboardText`, `selectedCells`, `copyWithHeaders`, `cancel` | Cancel to block. |
| `onClipboardPaste` | `clipboardText`, `pasteMatrix`, `startRowIndex`, `startColumnIndex`, `cancel` | Validate / transform. |
| `onClipboardCut` | `clipboardText` | Pre-cut hook. |

### Behaviour

- Rectangular selections → tab-separated values.
- Scattered selections → newline-separated values.
- Cut preserves primary key columns automatically.
- Paste appends new rows when content exceeds the current view (if `allowAdd: true`).
- Primary key cannot be modified through clipboard paste.

## Context menu

### Settings

```tsx
const [ctx] = useState<ContextMenuSettings>({
  enabled: true,
  items: ['Edit', 'Delete', 'Save', 'Cancel', 'SelectRow', 'ClearSelection', 'SortAscending', 'SortDescending', 'ClearSort'],
});
```

### Per-column items

```tsx
<Column field="Status"
        contextMenuItems={[
          { id: 'markApproved', text: 'Mark Approved' },
          { id: 'exportRows', text: 'Export Selected' },
        ]} />
```

### Built-in items

| Item | Context |
|---|---|
| `SortAscending`, `SortDescending`, `ClearSort` | Header cells |
| `Edit`, `Delete`, `Save`, `Cancel`, `SelectRow`, `ClearRowSelection`, `ClearSelection` | Content cells |
| `FirstPage`, `PrevPage`, `NextPage`, `LastPage` | Pager |
| `Sum`, `Min`, `Max`, `Average`, `Count`, `TrueCount`, `FalseCount`, `Custom` | Aggregate cells |

### Events

| Event | Args |
|---|---|
| `onContextMenuClick` | `(args.item.id, args.column?, args.data?)` |
| `onContextMenuOpen` | `{ column, items }`; set `args.items = []` to hide all defaults |

### Wiring custom actions

```tsx
const onContextMenuClick = useCallback((args: { item: { id: string } }) => {
  const ref = gridRef.current;
  switch (args.item.id) {
    case 'markApproved': ref?.updateRecord(/* index */ 0, { status: 'Approved' }); break;
    case 'goPage3': ref?.goToPage(3); break;
    case 'applyFilter': ref?.filterByColumn('Status', 'equal', 'Approved'); break;
    case 'clearFilters': ref?.clearFilter(); break;
    case 'sortAsc': ref?.sortByColumn('Name', 'Ascending'); break;
  }
}, []);
```

## Toolbar

`toolbar` is a `string[]` of built-in names: `'Add'`, `'Edit'`, `'Delete'`, `'Update'`, `'Cancel'`, `'Search'`, `'ColumnChooser'`, `'Print'`, `'PdfExport'`, `'ExcelExport'`. Items may be conditionally rendered by their `EditSettings` flags and registered modules.

```tsx
import { ToolbarModule } from '@syncfusion/react-grid';

const modules = { /* ..., */ ToolbarModule };

// Multiple sets across pages / scenarios
const [toolbar] = useState<string[]>(['Add', 'Edit', 'Delete', 'Update', 'Cancel', 'Print', 'ExcelExport', 'PdfExport', 'Search', 'ColumnChooser']);

<Grid toolbar={toolbar} modules={modules} ... />
```

`onClick` is wired through `ToolbarClickEvent.item`. The handler may inspect `args.item.id === 'Print'` etc. to branch.

> Items like `'Print'`, `'PdfExport'`, `'ExcelExport'` require the matching hook-based exporter (`useGridPrint`, `useGridPdfExport`, `useGridExcelExport`) to do the work — see `references/exports.md`. Without the hook, the toolbar button renders but clicking is a no-op.

## Common patterns

| Need | Configuration |
|---|---|
| Excel-like fill down | `autofillSettings.enabled + cellSelectionType: 'Box'` |
| Copy range to clipboard | `clipboardSettings.copyWithHeaders: true` |
| Lock PK from autofill | `<Column isPrimaryKey disableAutofill />` (auto) |
| Custom context-menu action | `contextMenuItems={[{ id:'…', text:'…' }]}` + `onContextMenuClick` |
| Custom tools alongside built-ins | `toolbar={['Add', 'Edit', '…', 'ColumnChooser']}` |
| Print / PDF / Excel | see `references/exports.md` |

## Constraints & guardrails

- **Autofill cell selection** requires `selectionSettings.cellSelectionType: 'Box' | 'BoxWithBorder'` — `Flow` mode is insufficient.
- **Clipboard paste** honours `allowAdd: true` — without it, paste that exceeds the current view errors silently.
- **Context menu custom items** appear alongside the built-ins; to hide built-ins set `args.items = []` from `onContextMenuOpen`.
- **Toolbar entries** that need a hook (Print/PDF/Excel) require the matching `useGridXxxExport` wiring. A grid with `'PdfExport'` in toolbar but no `useGridPdfExport` call will trigger a no-op click — wire the hook or remove the entry.
- **Custom context-menu actions**: target a cell/row by primary key (`args.data[pk]`), then call ref methods. Index-based targeting is unstable across sort/filter.
- **Integration**: when enabling autofill/clipboard/context-menu, ensure `EditModule`, `isPrimaryKey`, and the relevant module (AutoFillModule / ClipboardModule / ContextMenuModule) are all registered — missing any of these is the most common silent failure.