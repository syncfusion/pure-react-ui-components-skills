---
name: imperative-api
description: Consolidates every method on the Grid imperative handle (`GridRef`) for the Syncfusion React Data Grid — sort, filter, search, paging, selection (row/cell/checkbox), editing (add/edit/delete/save/cancel/setCellValue/setRowData), columns (getColumns/getColumnByField/resize/reorder/openColumnChooser), autofill (applyFill), clipboard (copy/cut/paste), data (refresh), streaming (currentViewData). Load when wiring command buttons, custom toolbar actions, context-menu handlers, or external state into a grid.
---

# Imperative API (`GridRef`)

Every public method on the grid's imperative handle, grouped by feature, with full signatures. Use `useRef<GridRef>(null)` and always chain optional-call: `gridRef.current?.method(...)`.

```tsx
import { useRef } from 'react';
import { Grid, GridRef } from '@syncfusion/react-grid';

const gridRef = useRef<GridRef>(null);
<Grid ref={gridRef} ... />
```

## Sort

```ts
gridRef.current?.sortByColumn(
  field: string,
  direction: 'Ascending' | 'Descending',
  isMultiSort?: boolean,
): void;
gridRef.current?.removeSortColumn(field: string): void;
gridRef.current?.clearSort(): void;

// Asynchronous variants — each returns a Promise
gridRef.current?.sortColumnAsync(columnName: string, sortDirection: SortDirection, isMultiSort?: boolean): Promise<void>;
gridRef.current?.removeSortColumnAsync(columnName: string): Promise<void>;
gridRef.current?.clearSortingAsync(fields?: string[]): Promise<void>;
```

Use the `Async` variants when sorting triggers a server round-trip (DataManager / `onDataRequest`); wait for the resolved promise before downstream actions (`pdfExport`, paging, etc.).

## Filter

```ts
gridRef.current?.filterByColumn(
  field: string,
  operator: string,
  value: unknown,
  predicate?: 'and' | 'or',
  matchCase?: boolean,
  ignoreAccent?: boolean,
): void;
gridRef.current?.clearFilter(fields?: string | string[]): void;

// Asynchronous variants
gridRef.current?.filterByColumnAsync(
  field: string,
  operator: string,
  value: unknown,
  predicate?: 'and' | 'or',
  matchCase?: boolean,
  ignoreAccent?: boolean,
): Promise<void>;
gridRef.current?.clearFilteringAsync(fields?: string[]): Promise<void>;
```

`fields` may be a single field name (string) or an array; passing no arg clears all filters.

## Search and data fetch

```ts
gridRef.current?.search(value: string): void;
gridRef.current?.searchAsync(value: string): Promise<void>;   // resolves after any server round-trip
gridRef.current?.getData(paged?: boolean): T[] | undefined;
```

Pass `getData(true)` (or omit) for the current page; `getData(false)` for the full filtered dataset across pages.

## Paging

```ts
gridRef.current?.goToPage(pageIndex: number): void;
gridRef.current?.goToPageAsync(pageIndex: number): Promise<void>;   // resolves after server request
gridRef.current?.pagerRef?.totalRecordsCount: number;  // read-only quota helper
```

## Selection (rows)

```ts
gridRef.current?.selectRow(index: number): void;
gridRef.current?.selectRows(indices: number[]): void;
gridRef.current?.clearRowSelection(indices?: number[]): void;
gridRef.current?.clearSelection(): void;
gridRef.current?.getSelectedRecords(): T[] | { isSelectAll: boolean; primaryKeys: string[] };
gridRef.current?.getSelectedRowIndexes(): number[];
```

`getSelectedRecords` returns local-data row arrays (with `persistSelection: true` covering all pages), or the remote-delete metadata object.

## Selection (cells)

```ts
gridRef.current?.clearCellSelection(): void;
```

## Editing — row

```ts
gridRef.current?.addRecord(data?: object, index?: number): void;
// Async variant
gridRef.current?.addRecordAsync(data: object): Promise<void>;

gridRef.current?.editRecord(): void;
gridRef.current?.updateRecord(index: number, data: object): void;
// Async variant
gridRef.current?.updateRecordAsync(index: number, data: object): Promise<void>;
gridRef.current?.deleteRecord(): void;
// Async variant
gridRef.current?.deleteRecordAsync(): Promise<void>;
gridRef.current?.saveDataChanges(): void;
gridRef.current?.cancelDataChanges(): void;
gridRef.current?.isEdit: boolean;       // read-only state
gridRef.current?.getRowInfo(target: HTMLElement): { rowIndex?: number } | null;
```

Async CRUD variants resolve after any remote `DataManager` round-trip finishes — chain them with paging/sort/search async variants and downstream exports. Full guide: `references/updating-data.md`.

## Editing — cell

```ts
gridRef.current?.editCell(primaryKey: string | number, columnField: string): void;
gridRef.current?.saveCellChanges(): void;
gridRef.current?.cancelCellChanges(): void;
gridRef.current?.setCellValue(
  key: string | number,
  field: string,
  value: unknown,
  triggerEvent?: boolean,
): void;
gridRef.current?.setCellValueAsync(
  rowKey: string | number | RowIndex,
  columnName: string,
  value: unknown,
  isDataSourceChangeRequired?: boolean,
): Promise<void>;
gridRef.current?.setRowData(
  key: string | number,
  data: object,
  triggerEvent?: boolean,
): void;
// Async variant
gridRef.current?.setRowDataAsync(
  key: string | number,
  value: object,
  isDataSourceChangeRequired?: boolean,
): Promise<void>;
```

`triggerEvent=true` (or `isDataSourceChangeRequired=true` in the async form) commits to the underlying data source and raises `onDataChangeStart` / `onDataChangeComplete`.

## Editing — batch

```ts
gridRef.current?.saveBatchChanges(): void;
gridRef.current?.cancelBatchChanges(): void;
gridRef.current?.getBatchQueue(): unknown[];        // pending batch changes
gridRef.current?.getConfirmBatch(): boolean;        // whether confirmation is required
```

## Editing — bulk & undo/redo

```ts
// Bulk update multiple records with the same field-level change
gridRef.current?.saveBulkChanges(
  changedData: object,                  // { field: value, … }
  rowData?: object[],                   // optional; defaults to selected records
  callback?: (result: unknown) => void,
): void;

// Undo / Redo (requires editSettings.allowUndoRedo)
gridRef.current?.undo(): void;
gridRef.current?.redo(): void;
gridRef.current?.getUndoActionsCount(): number;
gridRef.current?.getRedoActionsCount(): number;
gridRef.current?.clearUndoRedoHistory(): void;
```

## Grouping

```ts
gridRef.current?.groupColumnAsync(fields: string[], isResetRequired?: boolean): Promise<void>;
gridRef.current?.ungroupColumnAsync(fields: string[]): Promise<void>;
gridRef.current?.clearGroupingAsync(): Promise<void>;
```

## Columns

```ts
gridRef.current?.getColumns(): ColumnProps[];
gridRef.current?.getColumnByField(field: string): ColumnProps | null;
gridRef.current?.resizeColumns(items: { field: string; width: number }[]): void;
gridRef.current?.reorderColumns(fieldOrFields: string | string[], targetIndex: number): void;
gridRef.current?.openColumnChooser(x?: number, y?: number): void;

// Pinning
gridRef.current?.pinColumn(field: string, direction: ColumnPinDirection): void;
gridRef.current?.unpinColumn(field: string): void;
```

## Autofill

```ts
gridRef.current?.applyFill({
  direction: 'down' | 'right' | 'up' | 'left',
  fillCount: number,
  sourceCells?: CellRef[],
  targetCells?: CellRef[],
  isAltFill: boolean,
}): void;
```

`CellRef`: `{ rowKey: string | number; fieldName: string; rowIndex: number; columnIndex: number; }`.

## Clipboard

```ts
gridRef.current?.copyToClipboard(): void;
gridRef.current?.cutToClipboard(): Promise<void>;
gridRef.current?.pasteFromClipboard(text?: string): Promise<void> | void;
```

Without `text`, the grid tries the system clipboard (browser support varies). With a string argument, applies directly.

## Layout & data

```ts
gridRef.current?.refresh(): void;
gridRef.current?.currentViewData: T[];   // property (read-only)

// Pinning (rows)
gridRef.current?.pinRows(records: T[], position: 'top' | 'bottom'): void;
gridRef.current?.unpinRows(records: T[]): void;
```

`currentViewData` is the rows currently rendered — useful for picking a sample of rows to mutate (streaming), and for looking up a row index by primary key when `getCurrentViewRecords` is needed.

## Resolve a row index by primary key (helper pattern)

```ts
const idx = gridRef.current?.getCurrentViewRecords?.()
  ?.findIndex((row: T) => row[primaryKeyField] === pk);
```

If your templates need a row index by primary key, this is the canonical pattern.

## Constraints & guardrails

- **Optional chaining is mandatory**: every call must use `gridRef.current?.method?.()`. The ref is `null` until mount.
- **`addRecord`/`updateRecord`** use the **current view index**, not the primary key. Prefer `setRowData` after sort/filter has been applied.
- **`setCellValue`/`setRowData`** default to UI-only updates. Pass `true` to commit to the underlying source and emit events.
- **`isPrimaryKey`** is required for `setCellValue`, `setRowData` (they identify rows by primary key), `persistSelection`, `autoFill`, and clipboard paste.
- **Streaming grids**: prefer `setCellValue` over `updateRecord` to keep re-renders minimal.
- **High-risk operations**: gating the calls in this file is part of high-risk action gating per the project's guardrail framework — bulk calls (`clearFilter([])`, `deleteRecord()` on a selection of N rows, `setCellValue` in a tight loop) deserve user confirmation.
- **Integration**: tooling that calls these methods (context-menu handlers, command items, the toolbar) should remain pure — wrap with `useCallback` and pass the imperative handle through props or context.