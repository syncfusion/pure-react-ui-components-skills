---
name: modules-and-imports
description: Complete module map, named exports, and enumerations for `@syncfusion/react-grid`. Load before authoring or modifying any React Data Grid code so the right feature modules, component exports, settings types, and enum values are imported.
---

# Modules, imports, exports & enums

Everything the Syncfusion React Data Grid exposes lives in `@syncfusion/react-grid` (peer packages carry the supporting controls: `@syncfusion/react-data`, `@syncfusion/react-base`, `@syncfusion/react-inputs`, `@syncfusion/react-buttons`, `@syncfusion/react-dropdowns`, `@syncfusion/react-calendars`, `@syncfusion/react-notifications`, `@syncfusion/react-charts`, `@syncfusion/excel-export`, `@syncfusion/react-cldr-data`, `@syncfusion/react-locale`, `@syncfusion/react-material-theme`, `@syncfusion/react-bootstrap-theme`, `@syncfusion/react-tailwind-theme`).

## Package install

```bash
npm install @syncfusion/react-grid --save
npm install @syncfusion/react-material-theme --save   # or Bootstrap/Tailwind
# Optional peers (load what you actually use):
npm install @syncfusion/react-inputs @syncfusion/react-buttons \
            @syncfusion/react-dropdowns @syncfusion/react-calendars \
            @syncfusion/react-data @syncfusion/react-base \
            @syncfusion/react-notifications @syncfusion/react-charts \
            @syncfusion/excel-export --save
```

CSS theme registration (one place in `src/App.css`, once, in import order):

```tsx
@import "../node_modules/@syncfusion/react-material-theme/styles/grid/index.css";
```

## Feature → required module

Every grid you produce declares its `modules` map. Missing modules produce silent runtime errors (`PagerModule` for paging, `FilterModule` for filter UI, etc.). `GridAllModule` is acceptable for prototypes; production should pick single modules for tree-shaking.

| Feature | Required module(s) |
|---|---|
| Always-on | (built-in: data bind, sort, columns, selection row default, lifecycle events) |
| **Paging** | `PagerModule` |
| **Filtering** (all modes: FilterBar, Menu, Excel, CheckBox, custom) | `FilterModule` |
| **Sorting** | built into `Grid` |
| **Search toolbar** | `SearchModule` |
| **Toolbar** (Add/Edit/Delete/Update/Cancel/Print/PdfExport/ExcelExport/ColumnChooser) | `ToolbarModule` (uses Toolbar registry; not strictly required, see code samples — most docs include only `EditModule` etc.) |
| **Editing** (any mode: Normal/Cell/Inline/Popup/PopupTemplate/Dialog/Batch) | `EditModule` |
| **Command Column** (built-in edit/delete/update/cancel buttons inside a cell) | `CommandColumnModule` |
| **Grouping** | `GroupModule` |
| **Aggregates** (footer + group caption) | `AggregateModule` |
| **Column Resize** | `ResizeModule` |
| **Column Reorder** | `ReorderModule` |
| **Column Menu** (per-header dropdown) | (built-in; ensure `FilterModule`/`ColumnChooserModule`/`GroupModule`/`AggregateModule` present if their entries should appear in the menu) |
| **Column Chooser** | `ColumnChooserModule` |
| **Autofill** | `AutoFillModule` |
| **Clipboard** | `ClipboardModule` |
| **Context Menu** | `ContextMenuModule` |
| **Row Reorder (drag-and-drop)** | `RowReorderModule` |
| **Master/Detail (nested grids)** | `DetailGridModule` |
| All-in-one | `GridAllModule` |

```tsx
import { Grid, Columns, Column,
         FilterModule, PagerModule, SearchModule, EditModule,
         SortModule, GroupModule, AggregateModule, ResizeModule } from '@syncfusion/react-grid';

const modules = { FilterModule, PagerModule, SearchModule, EditModule,
                  SortModule, GroupModule, AggregateModule, ResizeModule };

<Grid modules={modules} enableDevMode={false} ... />
```

## Top-level components

| Component | Purpose |
|---|---|
| `<Grid>` | Root grid container. Accepts `dataSource`, settings, modules, ref, event handlers. |
| `<Columns>` | Wrapper for `<Column>` children. Auto-generates columns when absent. |
| `<Column>` | Single column definition. |
| `<Aggregates>` | Container for `<AggregateRow>`. |
| `<AggregateRow>` | One aggregate row block. |
| `<AggregateColumn>` | Single aggregate cell spec. |
| `<CommandItem>` | Child of `<Column>` with `getCommandItems={...}`; renders inline action buttons. |

## Settings (top-level) shapes

All settings types are exported. Defaults shown where the documentation is explicit.

### `FilterSettings`
```ts
{
  enabled: boolean;                         // default false
  type?: 'FilterBar' | 'Menu' | 'Excel' | 'CheckBox';  // default 'FilterBar'
  mode?: 'Immediate' | 'OnEnter';          // default 'Immediate'
  immediateModeDelay?: number;              // default 1500 (ms)
  caseSensitive?: boolean;                  // default false
  ignoreAccent?: boolean;                   // default false
  columns?: FilterPredicates[];             // initial filter set
  operators?: Partial<Record<OperatorGroup, { value: string; text: string }[]>>;  // Menu only
  enableFilterBarOperator?: boolean;        // FilterBar only
}
```

### `SortSettings`
```ts
{
  enabled: boolean;                         // default false
  mode?: 'Multiple' | 'Single';             // default 'Multiple'
  allowUnsort?: boolean;                    // default true
  columns?: { field: string; direction: SortDirection }[];
}
```

### `PageSettings`
```ts
{
  enabled: boolean;
  pageSize?: number;                        // default 12
  currentPage?: number;
  pageCount?: number;                       // number of pager buttons
  template?: (props: { currentPage: number; totalPages: number; totalRecordsCount: number }) => JSX.Element;
  estimatedTotalRecordsCount?: number;      // used in Infinite scroll for scrollbar hint
}
```

### `SearchSettings`
```ts
{
  enabled: boolean;
  fields?: string[];                        // restrict search to specified field names
  operator?: 'startswith' | 'endswith' | 'contains' | 'wildcard' | 'like' | 'equal';  // default 'contains'
  value?: string;
  ignoreCase?: boolean;
  caseSensitive?: boolean;
  ignoreAccent?: boolean;
}
```

### `EditSettings`
```ts
{
  mode?: 'Normal' | 'Dialog' | 'Inline' | 'Batch' | 'Cell' | 'Popup' | 'PopupTemplate';  // default 'Normal'
  allowEdit?: boolean;                      // default false
  allowAdd?: boolean;                       // default false
  allowDelete?: boolean;                    // default false
  confirmOnDelete?: boolean;                // default false; shows a confirm dialog when true
  popupSettings?: DialogLikeSettings;       // for mode 'Popup'/'PopupTemplate'
  popupTemplate?: (props: PopupTemplateProps) => React.ReactElement;
}
```

### `SelectionSettings`
```ts
{
  enabled: boolean;                         // default true (true = enabled)
  mode: 'Single' | 'Multiple';              // default 'Single'
  type?: 'Row' | 'Cell';                    // default 'Row'
  enableToggle?: boolean;                   // default false
  checkboxOnly?: boolean;                   // default false
  persistSelection?: boolean;               // default false (true when checkbox column rendered)
  cellSelectionType?: 'Flow' | 'Box' | 'BoxWithBorder';  // default 'Flow'; only with type 'Cell'
  autoSelectMode?: 'Default' | 'Intermediate';  // default 'Default'; checkbox selection
}
```

### `SortDescriptor`
```ts
{ field: string; direction: 'Ascending' | 'Descending' }
```

### `GroupSettings`
```ts
{
  enabled: boolean;
  columns?: string[];
  showDropArea?: boolean;
  defaultExpanded?: boolean | number;
  type?: GroupType;                         // 'GroupRows' | 'SingleColumn' | 'MultipleColumns'
}
```

### `VirtualizationSettings`
```ts
{
  enabled?: boolean;                        // default true when height set
  type?: 'Rows' | 'Columns' | 'Both';       // auto-detected if omitted
  scrollMode?: 'Auto' | 'Virtual' | 'Infinite';  // default 'Auto'
  enableCache?: boolean;                    // default true
  preventMaxRenderedRows?: boolean;         // default false (turn off to remove internal 500-row cap)
  viewPortBuffer?: { rows: number; columns: number };  // default { rows: 5, columns: 5 }
}
```

### `ClipboardSettings`
```ts
{
  enabled: boolean;
  allowRowCopy?: boolean;
  copyWithHeaders?: boolean;
  allowCut?: boolean;                        // default true
  allowPaste?: boolean;                     // default true
}
```

### `ContextMenuSettings`
```ts
{
  enabled: boolean;
  items?: (string | ContextMenuItemProps)[];
}
```

### `AutoFillSettings`
```ts
{
  enabled: boolean;
  shouldClearOnReduction?: boolean;
  allowedDirection?: 'row' | 'column' | 'both';   // default 'both'
  excludeFromAutoFill?: (field: string) => boolean;
  fillOperation?: (args: FillOperationArgs) => unknown | false;
}
```

### `ColumnChooserSettings`
```ts
{
  sortDirection?: 'None' | 'Ascending' | 'Descending';
  selectedColumns?: string[];
  operator?: string;                          // default 'startsWith'
  ignoreAccent?: boolean;                     // default false
  enableSearch?: boolean;                     // default true
  template?: (props: ...) => React.ReactElement;
  headerTemplate?: React.ComponentType | (props) => React.ReactElement;
  footerTemplate?: React.ComponentType<ColumnChooserFooterProps>;
  mode?: 'immediate';
  immediateModeDelay?: number;
}
```

### `ColumnMenuSettings`
```ts
{
  enabled: boolean;
  showFilter?: boolean;                       // default false  — moves filter action into the menu
}
```

### `ResizeSettings`
```ts
{
  enabled: boolean;
  mode?: 'Normal' | 'Auto';                  // default 'Normal'
  resizeKeyboardStep?: number;                // pixels per Alt+Arrow; default ~8
}
```

### `ReorderSettings`
```ts
{ enabled: boolean }
```

### `TextWrapSettings`
```ts
{
  enabled: boolean;
  wrapMode: 'Header' | 'Content' | 'Both';    // default 'Both' (when enabled)
}
```

### `RowNumberSettings`
```ts
{ enabled: boolean }
```

### `DragAndDropSettings`
```ts
{
  enabled: boolean;
  targetID?: string;
}
```

## Enumerations (named imports)

| Enum | Members |
|---|---|
| `TextAlign` | `Left`, `Right`, `Center`, `Justify` |
| `ClipMode` | `Clip`, `Ellipsis`, `EllipsisWithTooltip` |
| `ColumnType` | `String`, `Number`, `Date`, `DateTime`, `Boolean`, `Checkbox`, `SingleGroup` |
| `FilterBarType` | `TextBox`, `NumericTextBox`, `DatePicker` |
| `EditType` | `TextBox`, `NumericTextBox`, `DropDownList`, `CheckBox`, `DatePicker`, `RichTextEditor`, `Slider`, `Switch`, `TextArea`, `MaskedTextBox`, `AutoComplete`, `MultiSelect`, `ComboBox`, `TimePicker`, `DateTimePicker` |
| `WrapMode` | `Both`, `Header`, `Content` |
| `GridLine` | `Default`, `None`, `Both`, `Horizontal`, `Vertical` |
| `AutoFitMode` | `All`, `Header`, `Content` |
| `ResizeMode` | `Normal`, `Auto` |
| `SelectionMode` | `Single`, `Multiple` |
| `SelectionType` | `Row`, `Cell` |
| `CellSelectionType` | `Flow`, `Box`, `BoxWithBorder` |
| `AutoSelectMode` | `Default`, `Intermediate` |
| `ActionType` | `Add`, `BeginEdit`, `Save`, `Delete`, `DeleteConfirm`, `Refresh`, `Sorting`, `ClearSorting`, `Filtering`, `ClearFiltering` |
| `SortDirection` | `Ascending`, `Descending` |
| `GroupType` | `GroupRows`, `SingleColumn`, `MultipleColumns` |
| `AggregateType` | `Sum`, `Average`, `Min`, `Max`, `Count`, `TrueCount`, `FalseCount`, `Custom` |
| `CellType` | `Header`, `Content`, `Summary`, `GroupCaption`, `Expand`, `Detail`, … |
| `SelectionMode` | `Single`, `Multiple` |
| `VirtualDomType` | `Row`, `Column`, `Both` |
| `ScrollMode` | `Auto`, `Virtual`, `Infinite` |

## Filter operators (string literals accepted by `FilterPredicates.operator` and `filterByColumn`)

| Operator | String types | Number/Boolean/Date |
|---|---|---|
| `startsWith`, `endsWith`, `contains`, `doesNotStartWith`, `doesNotEndWith`, `doesNotContain` | ✓ | |
| `equal`, `notEqual` | ✓ | ✓ |
| `greaterThan`, `greaterThanOrEqual`, `lessThan`, `lessThanOrEqual`, `between` | | ✓ |
| `in`, `notIn` | | ✓ |
| `isNull`, `isNotNull`, `isEmpty`, `isNotEmpty` (empty only for string) | ✓ | partial |

`FilterPredicates` shape (used in `filterSettings.columns` and `filterByColumn`):
```ts
{
  field: string;
  operator: string;        // any operator above
  value: string | number | Date | boolean | null;
  predicate?: 'and' | 'or';
  ignoreAccent?: boolean;
}
```

## Event argument types (named imports)

| Type | Field summary | Fires on |
|---|---|---|
| `SortEvent` | `action`, `field`, `direction` | `onSort` |
| `SearchEvent` | `value` | `onSearch` |
| `PageEvent` | `currentPage`, `previousPage` | `onPageChange` |
| `FilterEvent` | `action`, `currentFilterColumn`, `currentFilterPredicate` | `onFilter` |
| `RowSelectEvent` | `selectedRowIndex`, `selectedRowIndexes`, `selectedCurrentRowIndexes`, `data` | `onRowSelect` |
| `RowSelectEvent` | `deSelectedRowIndex`, `deSelectedRowIndexes`, `deSelectedCurrentRowIndexes` | `onRowDeselect` |
| `RowEditEvent` | `data`, `cancel`, `rowIndex?` | `onRowEditStart`, `onRowAddStart` |
| `DeleteEvent` | `action: 'Delete'`, `data: T[]` | `onDataChangeStart` (delete) |
| `SaveEvent` | `action: 'add' \| 'edit'`, `data: T \| T[]` | `onDataChangeStart` (add/edit) |
| `EditTemplateProps` | `data`, `onChange(value)` | column-level `editTemplate` callback |
| `ColumnTemplateProps<T>` | `data`, `column`, `rowIndex` | `template` and `headerTemplate` |
| `ColumnHeaderTemplateProps` | `column` | header templates |
| `CellClassProps` | `data`, `column`, `cellType`, `rowIndex` | `cellClass` callback |
| `RowClassProps` | `data` | `rowClass` callback |
| `RowInfo` | `data` | `getRowHeight` callback |
| `ValueAccessorProps` | `data`, `column` | `valueAccessor` callback |
| `RowDragEventArgs` | `fromIndex`, `dropIndex`, `data: T[]`, `cancel`, `targetGrid?` | drag events |
| `DataRequestEvent` | `skip`, `take`, `requiresCounts`, `sort`, `where`, `search`, `select`, `distinct`, `requestType`, `dataSource` | `onDataRequest` |
| `ColumnResizeStartEvent` | `column`, `cancel` | resize start |
| `ColumnResizeEvent` | `column` | during resize |
| `ColumnResizeEndEvent` | `column`, `width` | resize end |
| `ColumnReorderStartEvent` | `column`, `fromIndex` | reorder start |
| `ColumnReorderEvent` | `column`, `fromIndex`, `toIndex` | during reorder |
| `ColumnReorderEndEvent` | `column`, `fromIndex`, `toIndex` | reorder end |
| `ColumnMenuOpenEvent` | `column`, `menuPosition` | menu open |
| `ColumnMenuClickEvent` | `item`, `column` | menu item click |
| `ColumnMenuCloseEvent` | `column` | menu close |
| `FilterDialogBeforeOpenEvent` | `columnName`, `options` | Excel/CheckBox filter popup |
| `ContextMenuOpenEvent` | `column`, `items` | context-menu open |
| `ToolbarClickEvent` | `item` (one of `'Add'`, `'Edit'`, `'Delete'`, `'Update'`, `'Cancel'`, `'Print'`, `'PdfExport'`, `'ExcelExport'`, `'ColumnChooser'`, `'Search'`) | toolbar click |
| `PdfExportAfterEvent` | `success`, `error?`, `promise?` | `onAfterPdfExport` |
| `ExcelAfterExportEvent` | `success`, `error?`, `promise?` | `onAfterExcelExport` |
| `ErrorEvent` | `args: Error` | `onError` |

## Cell selection helper enums (`SelectionType`, `AutoSelectMode`, `SelectionMode`)

`SelectionType.Cell` requires a primary-key column for accurate tracking. `AutoSelectMode.Intermediate` limits "select all" semantics to currently loaded pages — useful for infinite scroll + checkbox selection.

## Default values (locks for new files)

| Setting / prop | Default |
|---|---|
| `enableDevMode` | `true` — production code must override to `false` |
| `pageSettings.pageSize` | `12` |
| `sortSettings.mode` | `'Multiple'` |
| `sortSettings.allowUnsort` | `true` |
| `filterSettings.type` | `'FilterBar'` |
| `filterSettings.mode` | `'Immediate'` |
| `filterSettings.immediateModeDelay` | `1500` ms |
| `filterSettings.caseSensitive` | `false` |
| `filterSettings.ignoreAccent` | `false` |
| `searchSettings.operator` | `'contains'` |
| `editSettings.mode` | `'Normal'` |
| `selectionSettings.mode` | `'Single'` |
| `selectionSettings.persistSelection` | `false` (auto `true` with checkbox column) |
| `selectionSettings.cellSelectionType` | `'Flow'` |
| `textWrapSettings.wrapMode` | `'Both'` (when enabled) |
| `ClipMode.default` | varies; examples use `EllipsisWithTooltip` |
| `TextAlign.default` | `Left` |
| `resizeSettings.resizeKeyboardStep` | `~8` pixels |
| `Column.textAlign` | `Left` |
| `Column.headerTextAlign` | `Left` |
| `Column.visible` | `true` |
| `Column.allowResize` | `true` |
| `Column.allowReorder` | `true` |
| `Column.allowSort` | `true` |
| `Column.allowFilter` | `true` |
| `Column.allowEdit` | `mode-dependent` |
| `Column.autoHeight` | `false` |
| `Column.autoFit` | (none) |
| `Column.isPrimaryKey` | `false` |
| `virtualizationSettings.enableCache` | `true` |
| `virtualizationSettings.viewPortBuffer` | `{ rows: 5, columns: 5 }` |
| `virtualizationSettings.preventMaxRenderedRows` | `false` (internal 500-row cap applies when virtualization is off) |
| `clipboardSettings.allowCut` | `true` |
| `clipboardSettings.allowPaste` | `true` |
| `isPrimaryKey` on a primary-key column is **required** for: editing/add/delete, persistent selection, setCellValue/setRowData, checkbox selection, autofill, clipboard paste |  |

## Toolbar items accepted by the `toolbar` prop (string array)

`'Add'`, `'Edit'`, `'Delete'`, `'Update'`, `'Cancel'`, `'Print'`, `'PdfExport'`, `'ExcelExport'`, `'ColumnChooser'`, `'Search'`. The full toolbar list is registered through the `ToolbarClickEvent.item` value.

## Format options reference (`format` prop on `Column`)

Either a string shorthand or a `FormatOptions` object:
```ts
type FormatOptions = {
  type?: 'date' | 'time' | 'dateTime' | 'number' | 'currency' | 'percent';
  format?: string;            // e.g., 'yMd', 'C2', 'P0', 'dd/MM/yyyy', '$#,##0.00'
  skeleton?: 'short' | 'medium' | 'long' | 'full';
  calendar?: 'gregorian';
};
```
Common forms used in samples: `'yMd'`, `'C2'`, `'N0'`, `'N2'`, `'P0'`, `'dd/MM/yyyy HH:mm a'`, `'EEE, MMM d, yyyy \'at\' h:mm a'`. Numeric format specifier examples: `'0000'`, `'####.##'`, `'0000 %'`, `'$ ###.00'`, `'###.##;(###.00);-0'`. (See `references/cells.md` for full symbol table.)

## CSS hooks (for theming/styling overrides)

| Selector | What it styles |
|---|---|
| `.sf-grid` | Grid root |
| `.sf-grid-header-row .sf-cell` | Header cells (per-cell height override) |
| `.sf-grid-content-row` | Body rows |
| `.sf-grid-content-row:hover .sf-cell` | Hover row |
| `.sf-active` | Selected row (Row mode) |
| `.sf-cell.sf-cell-selected` | Selected cell (Cell mode) |
| `.sf-altrow` | Alternate rows when `enableAltRow` |
| `.sf-empty-record-template` | Empty record shell |
| `.sf-error` / `.sf-form-error` | Inline validation error wrapper |
| `.sf-column-chooser-dialog` | Column Chooser dialog |

## Constraints (do not violate silently)

- The grid's first render infers each column's `ColumnType` from the first cell. If the first cell is `null`/`undefined`/empty string, the inferred type will be wrong and the default editor + filter will mismatch. Always set `Column.type` when the first row could be missing values.
- Hidden (`visible: false`) columns still participate in `clearFilter([field])` and serialization for export. Use this deliberately.
- Dotted field paths (`field='Name.FirstName'`) need `expand('Name')` on the `Query` for `DataManager`-backed remote data.
- `valueAccessor` is **display-only** — sort/filter/edit use the raw data.
- `cellClass`/`template`/`headerTemplate`/`valueAccessor` callbacks execute on every render. Memoize them.
- `autoHeight: true` on any column disables column virtualization.