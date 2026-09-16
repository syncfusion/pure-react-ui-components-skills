# Grid (Data Grid) Migration

This section explains how to migrate the `Grid` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<Grid>`.


## Settings props (replace `<Inject services=[...]/>`)

| EJ2 prop | Pure React prop | Notes |
| --- | --- | --- |
| `aggregates` + `<AggregatesDirective>` | `aggregates` + `<Aggregates><AggregateRow>...` | Drop the directive pair. |
| `allowFiltering` + `<Inject services=[Filter]/>` | `filterSettings={{ enabled: true }}` | Config via object. |
| `allowPaging` + `<Inject services=[Page]/>` | `pageSettings={{ enabled: true }}` | Config via object. |
| `allowSelection` + `<Inject services=[Selection]/>` | `selectionSettings={{ enabled: true }}` | Config via object. |
| `allowSorting` + `<Inject services=[Sort]/>` | `sortSettings={{ enabled: true }}` | Config via object. |
| `allowTextWrap` | `textWrapSettings={{ enabled: true }}` | Config via object. |
| `enableVirtualization` + `<Inject services=[VirtualScroll]/>` | `virtualizationSettings={{ enabled: true, scrollMode: ScrollMode.Virtual }}` | Config via object. |
| `enableInfiniteScrolling` + `<Inject services=[InfiniteScroll]/>` | `virtualizationSettings={{ enabled: true, scrollMode: ScrollMode.Infinite }}` | Mode selects behavior. |
| `allowGrouping` + `<Inject services=[Group]/>` | `groupSettings={{ enabled: true }}` | Config via object. |
| `editSettings={{ allowEditing, allowAdding, allowDeleting }}` | `editSettings={{ allowEdit, allowAdd, allowDelete }}` | Keys become camelCase. |
| `toolbar` + `<Inject services=[Toolbar]/>` | `toolbar={['Add', 'Edit', 'Delete', ...]}` | Inject dropped. |
| `searchSettings={{ key, fields }}` | `searchSettings={{ value, fields, enabled }}` | `key` renamed to `value`; add `enabled`. |
| `loadingIndicator={{ indicatorType }}` | `loadingIndicatorSettings={{ indicatorType: LoadingIndicatorType.Spinner }}` | Renamed + enum. |
| `selectedDataIndexes` (selectionPattern selectionMode allowMultiSelection) | on `<ChartSelection mode, allowMultiSelection, selectedDataIndexes />` | See selection props. |
| `clipMode` string → `clipMode` enum | `ClipMode.EllipsisWithTooltip`, `ClipMode.Clip`, … | Enum. |
| `gridLines` string → `gridLines` enum | `GridLine.Both`, `GridLine.Default`, `GridLine.None`, `GridLine.Horizontal`, `GridLine.Vertical` | Enum. |
| `showInColumnChooser` (on Column) | `showInColumnChooser` | Same name; column directive → `<Column>`. |
| `columnChooserSettings={{ enableSearching, operator }}` (column chooser toolbar) | `columnChooserSettings={{ enableSearch, operator }}` | `enableSearching` → `enableSearch`; Inject dropped. |
| `selectionSettings={{ mode, cellSelectionMode, type }}` | `selectionSettings={{ type: SelectionType.Cell, mode, cellSelectionType, enabled }}` | `cellSelectionMode` → `cellSelectionType`. |
| `showContextMenu` + `contextMenuItems` + `<Inject services=[ContextMenu]/>` | `contextMenuSettings={{ enabled, items }}` | Inject dropped; combined prop. |
| `childGrid` + `<Inject services=[DetailRow]/>` | `isMasterDetail` + `detailRowTemplate` | New pattern — use template to render child Grid. |

`dataSource`, `query`, `rowTemplate`, `rowClass`, `emptyRecordTemplate`,
`rowHeight`, `width`, `height`, `id`, `enableAltRow`, `enableHover`,
`enableHtmlSanitizer`, `enableStickyHeader`, `allowKeyboard` carry over.

## Columns

`<ColumnsDirective>` / `<ColumnDirective>` → `<Columns>` / `<Column>`.
`Column.getColumnByField` returns `ColumnProps<T>` (was `Column`).

## Methods

Class refs use `useRef<GridRef<T>>(null)`. All `allow*`-typed grids must
remove `<Inject services=[…]/>` for the corresponding method to exist.

Renames on the ref:

| EJ2 method | Pure React method | Notes |
| --- | --- | --- |
| `closeEdit()` | `cancelDataChanges()` | Drop Inject Service. |
| `endEdit()` | `saveDataChanges()` (returns `Promise<boolean>`) | |
| `startEdit()` | `editRecord(rowElement?)` | Optional row. |
| `updateRow(index, data)` | `updateRecord(index, data)` | Use `T` for data. |
| `updateExternalMessage(msg)` | `setPagerMessage(msg?)` | Message optional. |
| `clearFiltering(fields?)` | `clearFilter(fields?)` | |
| `clearSorting(fields?)` | `clearSort(fields?)` | |
| `clearRowSelection()` | `clearRowSelection(indexes?: number[])` | Accepts indexes. |
| `sortColumn(name, dir)` | `sortByColumn(name, dir)` | |
| `selectCell({rowIndex,cellIndex})` | `selectCell(rowKey, fieldName)` | Use primary key + field. |
| `selectCells(rowCellIndexes)` | `selectCells(cells: RowCellInfo[])` | Keys + fieldNames. |
| `selectCellsByRange(start, end)` | `selectCellsByRange(start: CellIdentifier, end: CellIdentifier)` | Keys + fieldNames. |
| `selectRowsByRange(start, end)` | `selectRowByRange(start, end)` | |
| `filterByColumn(f, op, v, ..., matchCase)` | `filterByColumn(f, op, v, ..., caseSensitive)` | `matchCase` → `caseSensitive`; `actualFilterValue`/`actualOperator` removed. |
| `editCell(index, field)` | `editCell(primaryKey, field)` | Use primary key; mode `'Cell'`. |
| `groupColumn(columnName)` | `groupColumn(fields[], isResetRequired?)` | `columnName` (string) → `fields` (array). |
| `ungroupColumn(columnName)` | `ungroupColumn(fields[])` | Same. |
| `getColumnByField(field): Column` | `getColumnByField(field): ColumnProps<T>` | Return type. |
| `getColumns(): Column[]` | `getColumns(): ColumnProps<T>[]` | Return type. |
| `getHiddenColumns(): Column[]` | `getHiddenColumns(): ColumnProps<T>[]` | Return type. |
| `getVisibleColumns(): Column[]` | `getVisibleColumns(): ColumnProps<T>[]` | Return type. |
| `getSelectedRecords(): Object[]` | `getSelectedRecords(): T[] \| null` | Returns nullable list. |
| `getRowInfo(target)` | `getRowInfo(target: Element)` | Target must be `Element`. |
| `setCellValue(key, field, value)` | `setCellValue(key, field, value, isDataSourceChangeRequired?)` | Optional flag added. |
| `setRowData(key, rowData)` | `setRowData(key, data, isDataSourceChangeRequired?)` | Use `T`. |

Unchanged methods: `addRecord`, `clearSelection`, `clearCellSelection`,
`clearGrouping`, `deleteRecord`, `getData`, `getSelectedRowIndexes`,
`getPrimaryKeyFieldNames`, `goToPage`, `hideSpinner`, `showSpinner`,
`refresh`, `removeSortColumn`, `search`, `selectRow`, `selectRows`,
`openColumnChooser`.

`onExpandStateChange`, `validateEditForm`, `validateField` are new in Pure
React, exposed on the ref.

## Events

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `recordClick` | `onCellClick` (`CellFocusEvent<T>`) | |
| `onCellFocus` | `onCellFocus` | New event. |
| `onDataChangeCancel` | `onDataChangeCancel` (`FormCancelEvent<T>`) | New event. |
| `onDataChangeComplete` | `onDataChangeComplete` (`SaveEvent<T> \| DeleteEvent<T>`) | New event. |
| `dataSourceChanged` | `onDataChangeRequest` (`DataChangeRequestEvent<T>`) | Renamed. |
| `onDataChangeStart` | `onDataChangeStart` (`SaveEvent \| DeleteEvent`) | New event. |
| `dataBound` | `onDataLoad` | Renamed; no args. |
| `dataStateChange` | `onDataRequest` (`DataRequestEvent`) | Renamed. |
| `actionFailure` | `onError` (`Error`) | Renamed. |
| `onFilter` | `onFilter` (`FilterEvent`) | New event. |
| `onFormRender` | `onFormRender` (`FormRenderEvent<T>`) | New event. |
| `created` | `onGridRenderComplete` | Renamed; no args. |
| `load` | `onGridRenderStart` | Renamed; no args. |
| `onPageChange` | `onPageChange` (`PageEvent`) | New event. |
| `onRefresh` | `onRefresh` | New event. |
| `onRowAddStart` | `onRowAddStart` (`RowAddEvent<T>`) | New event. |
| `rowDeselected` | `onRowDeselect` (`RowSelectEvent<T>`) | Renamed. |
| `recordDoubleClick` | `onRowDoubleClick` (`RecordDoubleClickEvent<T>`) | Renamed. |
| `onRowEditStart` | `onRowEditStart` (`RowEditEvent<T>`) | New event. |
| `rowSelected` | `onRowSelect` (`RowSelectEvent<T>`) | Renamed. |
| `onSearch` | `onSearch` (`SearchEvent`) | New event. |
| `onSort` | `onSort` (`SortEvent`) | New event. |
| `toolbarClick` | `onToolbarItemClick` (`ToolbarClickEvent`) | Renamed. |
| `contextMenuOpen` | `onContextMenuOpen` (`ContextMenuOpenEvent`) | Renamed. |
| `contextMenuClick` | `onContextMenuClick` (`MenuSelectEvent`) | Renamed. |
| `contextMenuClose` | `onContextMenuClose` | Renamed. |
| `actionBegin` (grouping context) | `onGroup` (`OnGroupArgs`) | Renamed. |
| `detailExpand` | `onRowExpand` (`RowExpandEvent<T>`) | Renamed; remove DetailRow Inject. |
| `detailCollapse` | `onRowCollapse` (`RowCollapseEvent<T>`) | Renamed; remove DetailRow Inject. |
| `onColumnChooserBeforeOpen` | `onColumnChooserBeforeOpen` (`ColumnChooserBeforeOpenEvent`) | New event. |
| `onColumnChooserApply` | `onColumnChooserApply` (`ColumnChooserApplyEvent`) | New event. |