---
name: editing-modes
description: Editing mode walkthroughs for the Syncfusion React Data Grid — Normal (default), Cell, Popup, PopupTemplate, and external `setCellValue`/`setRowData` updates. Load when you need to choose between row-level / cell-level / dialog editing, build a custom form inside a popup, or wire external state into the grid without going through full edit lifecycle.
---

# Editing modes

`editSettings.mode` controls the editing UX form. This file is the mode-by-mode guide; the shared configuration (modules, primary key, commands, validation) lives in `references/editing.md`.

## Normal (default)

Inline row editing. Double-click a row (or press `F2`) to start editing; toolbar `Add` or `Insert` (`⌘+⌥+Enter` on macOS) adds a new row; `Delete` removes selected; `Enter` saves; `Esc` cancels.

Column-level props used in this mode:
- `defaultValue` — pre-fill for a freshly added row.
- `validationRules` — enforced on save.
- `edit={{ type: EditType.* }}` — editor binding.
- `filter={{ filterBarType: FilterBarType.* }}` — paired filter UI.
- `isPrimaryKey` — required flag.
- `allowEdit={false}` — opt-out per column.

Events unique to Normal:
- `onRowAddStart`, `onRowEditStart` — set `args.cancel = true` to block.
- `onRowSelect` — selection inside form rows.

Read edit state via `gridRef.current.isEdit: boolean` (true while form open). `selectRow(i)` + `editRecord()` is the canonical "click-to-edit" path; first call `saveDataChanges()` if you're already mid-edit.

```tsx
// Custom click-to-edit
const handleClick = (e: React.MouseEvent<HTMLElement>) => {
  if (!e.target.classList.contains('sf-cell')) return;
  if (gridRef.current?.isEdit) gridRef.current?.saveDataChanges();
  setTimeout(() => {
    const tr = (e.target as HTMLElement).closest('tr');
    const idx = tr?.sectionRowIndex ?? -1;
    gridRef.current?.selectRow(idx);
    gridRef.current?.editRecord();
  });
};

<div onMouseUp={handleClick}>
  <Grid ref={gridRef} ... />
</div>
```

## Cell

```tsx
const [edit] = useState<EditSettings>({ allowEdit: true, mode: 'Cell' });
```

Each cell editable independently. `<Column allowEdit={false} />` opts out.

Events: `onCellEditStart` (cancel via `args.cancel = true`), `onDataChangeStart` (commit gate).

Methods unique to Cell mode:

```tsx
gridRef.current?.editCell(primaryKey, columnField);     // enter edit on a cell
gridRef.current?.saveCellChanges();                     // commit the focused cell
gridRef.current?.cancelCellChanges();                   // restore original value
```

Read the cell back via `cellChange` / `onDataChangeStart`.

## Popup (Dialog)

```tsx
const [edit] = useState<EditSettings>({
  mode: 'Popup',
  allowEdit: true, allowAdd: true, allowDelete: true,
  popupSettings: {
    animation: { effect: 'FadeZoom', duration: 300 },
    draggable: true,
    resizable: true,
    position: 'Center',
    header: 'Edit Order',
    width: 600, height: 480,
    resizeHandles: ['All'],
  },
});

<Grid editSettings={edit} ... />
```

`popupSettings` mirrors the Dialog component: `header` (string|JSX), `width`/`height`, `position`, `animation`, `draggable`, `resizable`, `resizeHandles`. `header` may be wrapped in a state for dynamic titles (changing based on `args.data` in `onRowAddStart` / `onRowEditStart`).

## PopupTemplate (custom form)

Build a fully custom React form to render inside a modal. The grid provides `{ data, isAdd, requestType, action }`; your template collects values, validates, and pushes them back through `onDataChangeStart`.

```tsx
const [edit] = useState<EditSettings>({
  mode: 'PopupTemplate',
  allowEdit: true, allowAdd: true, allowDelete: true,
  popupTemplate: (props: PopupTemplateProps) => {
    const [values, setValues] = useState<FormData>(props.data ?? {});
    return (
      <Form initialValues={values} rules={formRules}>
        <FormField name='OrderID' label='Order ID' component={NumericTextBox} />
        <FormField name='Freight' label='Freight' component={NumericTextBox} format='N2' />
        <Button onClick={() => formRef.current?.submit?.()}>Save</Button>
      </Form>
    );
  },
});
```

`PopupTemplateProps`: `{ data, isAdd, requestType, action }`. Replace `args.data` inside `onDataChangeStart` with the form's collected values before save:

```tsx
const onDataChangeStart = useCallback((args: DeleteEvent | SaveEvent) => {
  if (formRef.current?.validate()) {
    args.data = formStateRef.current?.values;        // push the form's data
  } else if (args.action !== 'Delete') {
    args.cancel = true;                              // block commit if invalid
  }
}, []);
```

Forms use the `Form`/`FormField` API from `@syncfusion/react-inputs` (`Form`, `FormField`, `FormInitialValues`, `FormState`, `FormValueType`, `IFormValidator`, `ValidationRules`, `TextBox`, `TextBoxChangeEvent`, `NumericTextBox`, `NumericChangeEvent`).

Pair-validation rules with array-form messages, e.g.:
```ts
{
  OrderID:        { required: [true, 'Order ID is required'], number: [true, 'Must be a number'] },
  Freight:        { required: [true, 'Required'], number: [true, 'Must be a number'], range: [[1, 1000], 'Must be 1–1000'] },
  OrderDate:      { required: [true, 'Date required'], date: [true, 'Invalid date'] },
  CustomerName:   { required: [true, 'Required'], minLength: [3, 'At least 3 chars'] },
}
```

## Custom (external) updates without opening an editor

`setCellValue` and `setRowData` push values directly without entering edit mode.

```tsx
gridRef.current?.setCellValue('K-1001', 'Status', 'Approved');           // UI only
gridRef.current?.setCellValue('K-1001', 'Status', 'Approved', /*triggerEvent*/ true);  // commits to data source

gridRef.current?.setRowData('K-1001', { ...row, rating: 5 });            // UI only
gridRef.current?.setRowData('K-1001', { ...row, rating: 5 }, /*triggerEvent*/ true);
```

Required: a column marked `isPrimaryKey={true}` for the `key` parameter to resolve the row.

Both methods trigger **partial re-renders** of the affected cell/row only — fast enough for streaming scenarios.

## Batch editing

`editSettings.allowBatchSave: true` lets users edit multiple cells across rows before committing them in a single save. Pair with `mode: 'Cell'` (cell-level batch) or `mode: 'Normal'` (full-row batch — multiple cells in a row committed together).

```tsx
const [editSettings] = useState<EditSettings>({
  allowEdit: true,
  allowBatchSave: true,
  mode: 'Cell',
});

<Grid
  editSettings={editSettings}
  toolbar={['Edit', 'Update', 'Cancel']}
  modules={{ EditModule }}
  ...
/>
```

Batch-specific imperative methods:

| Method | Description |
|---|---|
| `saveBatchChanges()` | Commit all pending batch changes as a single batch. |
| `cancelBatchChanges()` | Discard all pending batch changes and restore original values. |
| `getBatchQueue()` | Return the current queue of pending batch changes. |
| `getConfirmBatch()` | Indicates whether pending batch changes require confirmation. |

Batch editing events:

| Event | Description |
|---|---|
| `onBatchEditStart` | Fires when batch editing starts. |
| `onBatchEditRender` | Fires when a batch edit is rendered. |
| `onBatchStateChange` | Fires when the pending batch state changes. |
| `onBatchCommitStart` | Fires before pending changes are committed. |
| `onBatchCommitSuccess` | Fires after pending changes are committed successfully. |
| `onBatchCommitError` | Fires when committing pending changes fails. |

## Undo/Redo editing

`editSettings.allowUndoRedo: true` plus `EditModule` enables per-edit undo/redo. Keyboard shortcuts: `Ctrl+Z` (undo) and `Ctrl+Y` (redo). The default history size is **5**; raise it with `editSettings.undoRedoLimit`.

```tsx
const [editSettings] = useState<EditSettings>({
  allowEdit: true,
  allowUndoRedo: true,
  undoRedoLimit: 10,
});

<Grid editSettings={editSettings} modules={{ EditModule }} ... />
```

Add `'Undo'` and `'Redo'` to the `toolbar` array to surface toolbar buttons:

```tsx
const toolbarOptions = ['Add', 'Edit', 'Delete', 'Update', 'Cancel', 'Undo', 'Redo'];
```

For batch editing, also set `allowBatchUndoRedo: true` so undo/redo work across virtual batch saves:

```tsx
const [editSettings] = useState<EditSettings>({
  allowEdit: true,
  allowBatchSave: true,
  allowUndoRedo: true,
  allowBatchUndoRedo: true,
  mode: 'Cell',
});
```

Imperative helpers:

| Method | Description |
|---|---|
| `undo()` | Revert the most recent editing action. |
| `redo()` | Reapply the most recent undone action. |
| `getUndoActionsCount()` | Number of actions available for undo. |
| `getRedoActionsCount()` | Number of actions available for redo. |
| `clearUndoRedoHistory()` | Clear the undo and redo history. |

Lifecycle events:

| Event | Description |
|---|---|
| `onUndoStart` | Fires before an undo action begins. |
| `onUndoComplete` | Fires after an undo action completes. |
| `onRedoStart` | Fires before a redo action begins. |
| `onRedoComplete` | Fires after a redo action completes. |

## Bulk save

`saveBulkChanges(changedData, rowData?, callback?)` applies the same field-level changes to multiple records in a single operation. If `rowData` is omitted, the change targets the **currently selected records**.

```tsx
gridRef.current?.saveBulkChanges(
  { Discount: 15, ShippingMethod: 'Express' },          // changedData
  [recordP002, recordP003, recordP004],                // rowData (optional)
);

// Or apply to currently-selected rows:
gridRef.current?.saveBulkChanges({ Discount: 15 });
gridRef.current?.refresh();                            // update the view
```

| Parameter | Type | Description |
|---|---|---|
| `changedData` | `object` | Field names and values to apply to the matching records. |
| `rowData` | `object[]` (optional) | The records to update. Defaults to current selection. |
| `callback` | `(result) => void` (optional) | Invoked after the changes are saved. |

Requires `isPrimaryKey` so the grid can identify rows. `saveBulkChanges` updates grid state only — call `refresh()` after if the UI should reflect new values immediately.

## Method table (consolidated)

| Method | Mode | Behavior |
|---|---|---|
| `addRecord(data?, index?)` | Normal / Inline / Batch / Dialog / Popup / PopupTemplate | Adds a new row; optional seed data. |
| `editRecord()` | Normal | Begins editing on selected row. |
| `deleteRecord()` | All modes | Removes selected row. |
| `saveDataChanges()` | Normal/etc. | Commits current edit form. |
| `cancelDataChanges()` | Normal/etc. | Discards current edit. |
| `updateRecord(index, data)` | Normal | Replaces row's values by index (unstable across sort/filter). |
| `editCell(primaryKey, columnField)` | Cell | Opens cell in edit mode. |
| `saveCellChanges()` | Cell | Commits the edited cell. |
| `cancelCellChanges()` | Cell | Restores the cell's original value. |
| `setCellValue(key, field, value, triggerEvent?)` | All | Pushes a value without edit form. |
| `setCellValueAsync(rowKey, columnName, value, isDataSourceChangeRequired?)` | All | Async variant — returns a Promise that resolves after any remote data request. |
| `setRowData(key, data, triggerEvent?)` | All | Replaces entire row without edit form. |
| `saveBatchChanges()` | Batch | Saves pending batch changes. |
| `cancelBatchChanges()` | Batch | Cancels pending batch changes. |
| `getBatchQueue()` | Batch | Returns current batch change queue. |
| `getConfirmBatch()` | Batch | Indicates if batch confirmation is required. |
| `saveBulkChanges(changedData, rowData?, callback?)` | All | Bulk-update records. |
| `undo()` / `redo()` | Undo/Redo enabled | Reverse or reapply edits. |
| `getUndoActionsCount()` / `getRedoActionsCount()` | Undo/Redo enabled | Read history depth. |
| `clearUndoRedoHistory()` | Undo/Redo enabled | Clear history. |

## Mode selection guide

| Need | Mode |
|---|---|
| Edit like a typical table row | `'Normal'` |
| Excel-like fast inline corrections | `'Cell'` |
| Surface one record's full edit form in a modal | `'Popup'` |
| Custom modal form (non-column fields, custom validation) | `'PopupTemplate'` |
| Drive updates from external state (streaming, external triggers) | `setCellValue`/`setRowData` |

## Constraints & guardrails

- **`isPrimaryKey`** is mandatory for all editing flows.
- **`onDataChangeStart`** can mutate `args.data` (replace the row) and/or set `args.cancel = true`. Use this as the single commit gate.
- **Click-to-edit** must `saveDataChanges()` first if the grid is already in edit state — otherwise you can corrupt internal state.
- **`updateRecord(index, …)`** uses the **current view index**, not the primary-key index. After sort/filter, prefer `setRowData` rather than relying on indices.
- **Integration**: when switching modes, also update the toolbar entries you support. A `'Cell'` mode grid doesn't benefit from `'Cancel'` / `'Update'` buttons (no row-level form); remove them to avoid non-functional buttons.