---
name: updating-data
description: Programmatic update workflows in the Syncfusion React Data Grid — sync vs async methods for `addRecord` / `updateRecord` / `deleteRecord` / `setCellValue` / `setRowData`, batch operations (`saveBatchChanges` / `saveBulkChanges`), and high-frequency / streaming update patterns that minimize re-renders. Load when wiring remote add/edit/delete, sequencing state transitions that must wait for the data source to commit, or building a real-time counter / monitoring view.
---

# Updating data (programmatic transactions)

The grid exposes synchronous and asynchronous APIs for adding, updating, deleting, and refreshing records. Sync variants return immediately; async variants return `Promise` and resolve after any remote `onDataRequest` / `DataManager` round-trip completes — pick the async form when the next UI step must observe the committed state.

## API overview

| Operation | Sync | Async |
|---|---|---|
| Add a record | `addRecord(data?)` | `addRecordAsync(data)` |
| Update by index | `updateRecord(index, data)` | `updateRecordAsync(index, data)` |
| Delete selected | `deleteRecord()` | `deleteRecordAsync()` |
| Set cell value | `setCellValue(pk, field, value, triggerEvent?)` | `setCellValueAsync(rowKey, columnName, value, isDataSourceChangeRequired?)` |
| Set row data | `setRowData(pk, data, triggerEvent?)` | `setRowDataAsync(pk, value, isDataSourceChangeRequired?)` |
| Bulk save (multiple records) | `saveBulkChanges(changedData, rowData?, callback?)` | — |
| Batch save (queued cell edits) | `saveBatchChanges()` / `cancelBatchChanges()` | — |
| Refresh view | `refresh()` | — |

`triggerEvent` / `isDataSourceChangeRequired` of `true` commit to the underlying data source and raise `onDataChangeStart` / `onDataChangeComplete`. Without it, only the UI updates.

## Sync vs async — when to reach for which

- **Sync** when the data source is local and the next line of code reads the new state from the in-memory array.
- **Async** when the grid is bound to `DataManager` or `onDataRequest` and downstream code (`pdfExport()`, `getData(false)`, the next mutation) must wait for the server commit.

```ts
// Async: chain paging, then filter, then export — each step waits for the previous to commit.
await gridRef.current?.goToPageAsync(2);
await gridRef.current?.filterByColumnAsync('Status', 'equal', 'On Hold');
await gridRef.current?.pdfExport();
```

`isDataSourceChangeRequired` defaults to `false`. With `DataManager` backends, omit it only when you intend the next authoritative state to come from a manual refresh / re-query.

## Async record CRUD

```ts
// Add one record
gridRef.current?.addRecord({ SaleID: 'S-NEW', ProductName: 'Smart Lamp' });
await gridRef.current?.addRecordAsync({ SaleID: 'S-1001', ProductName: 'Smart Watch' });

// Update by index (sync) and async
gridRef.current?.updateRecord(1, { ProductName: 'Phone Pro', UnitPrice: 749 });
await gridRef.current?.updateRecordAsync(1, { ProductName: 'Phone Pro', UnitPrice: 749 });

// Delete selected (sync) and async
gridRef.current?.deleteRecord();
await gridRef.current?.deleteRecordAsync();
```

`addRecord(data?)` opportunistically pre-fills values; without data, a blank form opens depending on the edit mode.

## Bulk save

Apply the same change to many records in a single call:

```ts
gridRef.current?.saveBulkChanges(
  { Discount: 15, ShippingMethod: 'Express' },        // changedData
  [recordP002, recordP003, recordP004],               // rowData (optional; defaults to selection)
);
gridRef.current?.refresh();                           // surface the change visually if needed
```

Omitting `rowData` targets the **currently selected** records — make sure that's the intended behaviour before triggering the call. `saveBulkChanges` updates grid state only; pair with `refresh()` if the source array needs UI re-rendering.

## Batch save

```ts
gridRef.current?.saveBatchChanges();      // commit pending batch changes
gridRef.current?.cancelBatchChanges();    // discard pending batch changes
gridRef.current?.getBatchQueue();         // inspect pending changes
gridRef.current?.getConfirmBatch();      // true when confirmation is required
```

Pair with `editSettings.allowBatchSave: true` and `mode: 'Cell'` (or `mode: 'Normal'` for full-row batch). See `references/editing-modes.md` for the full-batching config.

## Updating cell values (sync vs async)

```ts
gridRef.current?.setCellValue('P-001', 'Discount', 7.5, /*triggerEvent*/ true);
await gridRef.current?.setCellValueAsync('P-002', 'Discount', 7.5, /*isDataSourceChangeRequired*/ true);
```

Without the trailing `true`, the change is UI-only; the underlying source array / server isn't touched. **Streaming / monitoring grids** typically pass `true` once the connection has stabilised so downstream consumers see the new value — but for fast-tick simulations, leave it false to keep re-renders minimal.

## Updating row data

```ts
gridRef.current?.setRowData('ITEM-1002', updatedRow, /*triggerEvent*/ true);
await gridRef.current?.setRowDataAsync('ITEM-1001', updatedRow, /*triggerEvent*/ true);
```

`setRowData` replaces the row's data; `setCellValue` patches a single column. Both receive a primary key (any column marked `isPrimaryKey`). Without that key the imperative handle can't resolve the row.

## High-frequency updates (streaming / monitoring)

For dashboards that tick multiple cells per second:

```tsx
const updateOnce = () => {
  const rows = gridRef.current?.currentViewData ?? [];
  if (!rows.length) return;

  chargingStationData.forEach((row, index) => {
    setTimeout(() => {
      row.Revenue += 10;
      gridRef.current!.setCellValue(row.StationID, 'Revenue', row.Revenue, true);
    }, 0);
  });
};

const updateAsync = async () => {
  for (const row of chargingStationData) {
    row.Revenue += 20;
    await gridRef.current!.setCellValueAsync(row.StationID, 'Revenue', row.Revenue, true);
  }
};
```

Pair with `disable hover`, `disable keyboard`, fixed `rowHeight`, and `selectionSettings.enabled = false` for the cheapest re-render path. See `references/real-time-grid.md` for the full streaming pattern.

Track timing in `performance.now()` deltas to compare sync vs async throughput:

```tsx
const start = performance.now();
await gridRef.current?.setCellValueAsync(pk, 'Revenue', value, true);
console.log(`One update: ${(performance.now() - start).toFixed(2)} ms`);
```

## Refresh policy

`gridRef.current?.refresh()` forces a full layout recalculation. Use it as a last resort:

- After external mutations that bypassed the grid's APIs.
- After `saveBulkChanges` when the dataset is the same array reference and the cells would otherwise show stale values.
- After schema changes (columns added/removed at runtime).

For all other cases, prefer a targeted `setCellValue` / `setRowData` so only the affected cells re-render.

## Constraints & guardrails

- **`isPrimaryKey`** is required for every method that takes a row key (`setCellValue`, `setRowData`, async variants). Without it, the row resolution silently misses and edits silently fail.
- **`addRecordAsync` / `updateRecordAsync` / `deleteRecordAsync`** accept a separate `rowIndex` for updates; with sort/filter applied, an index-based update is unstable. Prefer selecting the row by primary key (use `gridRef.current?.selectRow(index)` then `updateRecord` for full edits, or use `setRowData`).
- **`saveBulkChanges`** bypasses `onDataChangeStart` (it doesn't edit through the form). For user-facing bulk changes that need validation, use the editing flow rather than imperative bulk save.
- **High-frequency tick patterns** must clean up `setInterval`s on unmount; lost timers continue updating rows after unmount and cause **high-risk data leaks** in production.
- **Trigger event semantics**: `setCellValue('P-1', 'Q', 5, false)` does not raise `onDataChangeStart`. If your server contract relies on the event to mirror UI to backend, pass `true`.
- **Integration**: when a remote `DataManager` is bound, async methods are usually required. Sync methods skip the network round-trip and may produce UI/server divergence.
