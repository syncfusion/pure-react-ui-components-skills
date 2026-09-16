---
name: columns
description: Column-level configuration for the Syncfusion React Data Grid — defining columns, auto-generation, types, format, templates, headers, resizing, reorder, menu, chooser, spanning, and dynamic state via React. Load before generating any column-rich grid or when the user asks about headers, header templates, column resize/reorder, the column menu, the column chooser dialog, or merging cells.
---

# Columns

`Column` props are the most-touched surface in `@syncfusion/react-grid`. This reference covers every documented column prop, the interactions between column props and grid-level settings, and the patterns that scale.

## Defining columns

Two modes:

```tsx
// Manual
<Columns>
  <Column field="EmployeeID" headerText="Employee ID" width="110" isPrimaryKey />
  <Column field="Name" headerText="Name" />
  <Column field="Department" headerText="Department" />
</Columns>

// Auto — grid reads dataSource records, creates a column per field
<Grid dataSource={data} />   // no <Columns> child
```

When auto is in use, customize runtime by reading columns from the ref:

```tsx
const onDataLoad = () => {
  const cols = gridRef.current?.getColumns() ?? [];
  cols[0].headerText = 'ID';
  cols[0].width = 80;
  cols[0].textAlign = TextAlign.Right;
  setColumns([...cols]);
};
```

`headerText` falls back to `field` when omitted.

## `ColumnProps` (every column-level prop)

All from `@syncfusion/react-grid` (no deep imports except for `ColumnProps` typing).

| Prop | Type | Default | Purpose |
|---|---|---|---|
| `field` | `string` | — | Source data field. Dotted paths supported (`Name.FirstName`). |
| `headerText` | `string` | = `field` | Header caption. (HTML allowed when `disableHtmlEncode={false}`.) |
| `headerTextAlign` | `TextAlign` | `Left` | Header-only alignment (independent from cells). |
| `textAlign` | `TextAlign` | `Left` | Cell alignment. |
| `width` | `string \| number` | auto | `'120'`, `'120px'`, `'25%'`, or `number`. |
| `minWidth` | `number \| string` | — | Lower bound for resize & render. |
| `maxWidth` | `number \| string` | — | Upper bound. |
| `autoFit` | `AutoFitMode` | — | `'All' \| 'Header' \| 'Content'`. |
| `type` | `ColumnType` (or `'checkbox'`) | inferred | Controls default editor + filter + display. Set explicitly if first value can be null. |
| `displayAsCheckBox` | `boolean` | `false` | Boolean columns render as checkbox when paired with `type={ColumnType.Boolean}`. |
| `format` | `string \| FormatOptions` | — | Display format. See `format` reference in `references/modules-and-imports.md`. |
| `visible` | `boolean` | `true` | `false` hides the column. Still participates in filter/export. |
| `clipMode` | `ClipMode` | `Ellipsis` | `'Clip' \| 'Ellipsis' \| 'EllipsisWithTooltip'`. |
| `template` | `(props?: ColumnTemplateProps) => string \| React.ReactElement` | — | Cell renderer. Memoize. |
| `headerTemplate` | `(props?: ColumnHeaderTemplateProps) => string \| React.ReactElement` | — | Header renderer. |
| `filterTemplate` | `(props?: { column: ColumnProps }) => React.ReactElement` | — | Filter cell custom control. |
| `cellClass` | `string \| ((props?: CellClassProps) => string)` | — | Per-cell CSS class. |
| `allowFilter` | `boolean` | `true` | `false` disables filter on this column. |
| `allowEdit` | `boolean` | mode-dep | `false` makes this column read-only. |
| `allowSort` | `boolean` | `true` | `false` disables sort. |
| `allowReorder` | `boolean` | `true` | `false` locks position. |
| `allowResize` | `boolean` | `true` | `false` disables resize. |
| `isPrimaryKey` | `boolean` | `false` | Marks the unique id column. Required for editing/selection persistence/setCellValue/autofill/checkbox selection. |
| `autoHeight` | `boolean` | `false` | Row auto-grows to fit template content. Disables column virtualization. |
| `edit` | `{ type: EditType; params?: ... }` | `TextBox` editor | Editor binding. |
| `editTemplate` | `(props?: EditTemplateProps) => React.ReactElement` | — | Custom editor. |
| `filter` | `{ filterBarType?, operator?, filterOperators?, type?, params?, filterTemplate? }` | — | Per-column filter UI. |
| `validationRules` | `ValidationRules` | — | Built-in/custom validators. |
| `valueAccessor` | `(props?: ValueAccessorProps) => ValueType` | — | Display-only computed value. |
| `disableHtmlEncode` | `boolean` | `true` (encoded) | `false` renders raw HTML in cells/header. |
| `disableColumnChooser` | `boolean` | `false` | Hides column from the Chooser list. |
| `showInColumnChooser` | `boolean` | `true` | Hidden from chooser when `false`. |
| `colSpan` | `boolean \| ((args) => number)` | — | Auto-merge identical adjacent cells horizontally. Function returns span width. |
| `rowSpan` | `boolean \| ((args) => number)` | — | Auto-merge identical vertically stacked cells. |
| `getCommandItems` | `() => React.ReactElement[]` | — | Render inline action icons inside cell. Use with `CommandColumnModule`. |
| `contextMenuItems` | `ContextMenuItemProps[]` | — | Column-scoped menu items. |
| `disableAutofill` | `boolean` | `false` | `true` blocks autofill on this column. Auto-true for primary-key columns. |
| `allowGroup` | `boolean` | `true` | `false` prevents drag-to-group. |
| `groupCaptionAggregateType` | `AggregateType` | — | Show aggregate inline in group caption. |
| `formatFn` | `(value: number) => string` | — | Foreground formatter passed to your caption template. |
| `defaultValue` | `unknown` | — | Pre-fill for Add row when column is editable. |
| `validationRules` | (also listed above) | | |

### Widths
- `'auto'` → expands to fit content; truncated with ellipsis when too narrow.
- `'120'` / `120` / `'120px'` → fixed pixels.
- `'25%'` → proportion of container width.
- Without `width`, columns split remaining space equally.

## `ColumnType`

| Value | Default editor | Default filter | Display |
|---|---|---|---|
| `ColumnType.String` (default) | `TextBox` | `TextBox` | raw |
| `ColumnType.Number` | `NumericTextBox` | `NumericTextBox` | numeric |
| `ColumnType.Date` | `DatePicker` | `DatePicker` | date (`'yMd'`) |
| `ColumnType.DateTime` | `DateTimePicker` | `DateTimePicker` | date+time |
| `ColumnType.Boolean` | `CheckBox` | `CheckBox` | bool |
| `ColumnType.Checkbox` | `CheckBox` | `CheckBox` | bool with checkbox column |
| `ColumnType.SingleGroup` | n/a | n/a | placeholder used by `<Columns>` when grouping with `SingleColumn` |
| `'checkbox'` (string) | n/a | n/a | renders row-selection checkbox column |

When `type` is set, the matching default `FilterBarType` and `EditType` are applied unless overridden via `Column.filter` / `Column.edit`.

> Important: the grid infers `ColumnType` from the **first cell's value**. If the first cell is `null`, `undefined`, empty string, the inferred type will be wrong. Set `type` explicitly whenever the first row could be missing values.

## Format options

Two accepted shapes:

```tsx
// String shorthand
<Column field="Price" format="C2" />
<Column field="OrderDate" format="yMd" type={ColumnType.Date} />

// Object (richer)
<Column field="OrderDate"
        format={{ type: ColumnType.Date, format: 'EEE, MMM d, yyyy \'at\' h:mm a' }}
        textAlign={TextAlign.Right} />
```

Numbers: `'N'`, `'N2'`, `'C'`, `'C2'`, `'P'`, `'P0'`. Dates: `'yMd'`, `'dd/MM/yyyy HH:mm a'`, skeletons `'short'`, `'medium'`, `'long'`, `'full'`. Custom number specifiers: `'0000'`, `'####'`, `'###0.##0#'`, `'0000 %'`, `'$ ###.00'`, `'###.##;(###.00);-0'`. Literal text in single quotes: `'####.## \'@\''`.

## Cell templates (`template`)

```tsx
const dateTemplate = useCallback((props?: ColumnTemplateProps): React.ReactElement => {
  const formatted = props?.data?.date
    ? new Date(props.data.date).toLocaleDateString('en-US',
        { month: '2-digit', day: '2-digit', year: 'numeric' })
    : '';
  return <span className="next-contact-date">{formatted}</span>;
}, []);
```

`ColumnTemplateProps<T>`: `{ data?: T; column?: ColumnProps; rowIndex?: number }`.

Always memoize the template function. Wrap template components in `React.memo` for large datasets:

```tsx
const StatusBadge = React.memo(({ data }: ColumnTemplateProps) => {
  const cls = data?.status === 'New' ? 'badge-new' : 'badge-default';
  return <span className={cls}>{data?.status}</span>;
});

const statusTemplate = useCallback((p?: ColumnTemplateProps) =>
  p?.data ? <StatusBadge {...p} /> : <div />, []);
```

When template content varies in height, set `autoHeight` on that column. **Note: `autoHeight` disables column virtualization; avoid for large datasets.**

## Header templates (`headerTemplate`, `headerTextAlign`, `headerText`, `headerTextAlign`)

```tsx
const headerTemplate = useCallback((props?: ColumnHeaderTemplateProps) => (
  <span>📅 {props?.column?.headerText}</span>
), []);

<Column field="orderDate" headerText="Order Date" headerTemplate={headerTemplate} />
```

`headerTextAlign` accepts `TextAlign`; alignments are independent between header and body.

## Text wrap (`textWrapSettings`)

```tsx
const [textWrap] = useState<TextWrapSettings>({ enabled: true, wrapMode: WrapMode.Both });
<Grid textWrapSettings={textWrap} />
```

`WrapMode.Header` / `WrapMode.Content` / `WrapMode.Both`. Wrap depends on `Column.width` — auto-width columns can't wrap predictably.

## Cell styling (`cellClass`)

```tsx
const getRatingCellClass = (props?: CellClassProps): string => {
  if (props?.cellType !== CellType.Content) return '';
  const data = props?.data as { PerformanceRating?: string };
  return data?.PerformanceRating === 'Excellent' ? 'rating-excellent' : '';
};
<Column field="PerformanceRating" cellClass={getRatingCellClass} />
```

Same callback can apply to header (`CellType.Header`) and content cells. **Performance:** `cellClass` as a function runs on every render — for hot paths, prefer `headerTemplate` or templates.

## Grid lines (`gridLines` on `<Grid>`)

`GridLine.Default | None | Both | Horizontal | Vertical` (string or enum). State pattern:
```tsx
const [lines] = useState<GridLine | string>(GridLine.Default);
<Grid gridLines={lines} />
```

## Clip mode (`clipMode` on `<Grid>` or `<Column>`)

`ClipMode.Clip | Ellipsis | EllipsisWithTooltip`. Column-level overrides the grid-level. Header cells respect the grid setting.

## Column resizing

```tsx
<Grid
  resizeSettings={{ enabled: true, mode: ResizeMode.Auto, resizeKeyboardStep: 8 }}
  modules={{ ResizeModule }}
/>
```

- `ResizeMode.Normal` — only the resized column changes width.
- `ResizeMode.Auto` — width delta redistributes across subsequent columns (respects `minWidth` / `maxWidth`).
- Keyboard: `Alt + ←/→` adjusts width by `resizeKeyboardStep`.
- Per-column: `allowResize={false}`, `minWidth`, `maxWidth`, `autoFit: AutoFitMode.Header`.
- Methods: `gridRef.current?.getColumnByField(field)`; `gridRef.current?.resizeColumns([{ field, width }])`.
- Events: `onColumnResizeStart`, `onColumnResize` (no cancel), `onColumnResizeEnd`. `args.cancel = true` on start aborts.

## Column reorder

```tsx
<Grid reorderSettings={{ enabled: true }} modules={{ ReorderModule, PagerModule }} />
```

- Per-column: `allowReorder={false}`.
- Programmatic: `gridRef.current?.reorderColumns(fieldName, targetIndex)` or `reorderColumns([field1, field2], targetIndex)`.
- Events: `onColumnReorderStart`, `onColumnDrag` (no cancel), `onColumnReorderEnd`. Drag start/end accept `args.cancel = true`.

## Column menu

```tsx
<Grid
  columnMenuSettings={{ enabled: true, showFilter: true }}
  showColumnChooser
  groupSettings={{ enabled: true }}
/>
```

- `showFilter: true` relocates the filter action into the menu (header filter icon disappears). Filtering remains active when `filterSettings.enabled` is `true`.
- Surface actions based on registered modules: `FilterModule`, `ColumnChooserModule`, `GroupModule`, `AggregateModule`.
- Events: `onColumnMenuOpen`, `onColumnMenuClick` (args `item`, `column`), `onColumnMenuClose`.

## Column chooser

```tsx
<Grid
  showColumnChooser
  toolbar={['ColumnChooser']}
  columnChooserSettings={{
    sortDirection: 'Ascending',
    operator: 'startsWith',
    ignoreAccent: false,
    enableSearch: true,
    mode: 'immediate',
    immediateModeDelay: 300,
  }}
  modules={{ ColumnChooserModule, ToolbarModule }}
/>
```

- `showInColumnChooser={false}` on a column hides it from the chooser.
- Custom templates: `template`, `headerTemplate`, `footerTemplate` (signature includes `ColumnChooserFooterProps` with `onApply`/`onClose`).
- Programmatic open: `gridRef.current?.openColumnChooser(x, y)`.
- CSS hook: `.sf-column-chooser-dialog`.

## Cell spanning (auto-merge)

```tsx
<Grid
  enableAutoSpan
  enableHover={false}
  selectionSettings={{ enabled: false }}
  gridLines={GridLine.Both}
/>
<Column field="Slot1" colSpan />
<Column field="Employee" rowSpan={({ data, field, rowIndex }) => {
  let count = 1;
  for (let i = rowIndex + 1; i < data.length; i++) {
    if (data[i].Employee === data[rowIndex].Employee &&
        data[i].Project === data[rowIndex].Project) count++;
    else break;
  }
  return count;
}} />
```

- `colSpan: true` → auto-merge identical adjacent cells horizontally.
- `colSpan: (args) => number` → custom span width.
- `rowSpan: true` → auto-merge identical vertically stacked cells (combine with `colSpan`).
- Spanning is purely visual; underlying data is unchanged.

## Multi-level (stacked) headers

```tsx
<Columns>
  <Column headerText="Employee Information" textAlign={TextAlign.Center}>
    <Columns>
      <Column field="EmployeeCode" headerText="Employee ID" isPrimaryKey />
      <Column headerText="Personal Details">
        <Columns>
          <Column field="FirstName" />
          <Column field="LastName" />
        </Columns>
      </Column>
    </Columns>
  </Column>
</Columns>
```

Parent grouping `<Column>` elements must not have a `field` — they are pure visual headers.

## Dynamic columns via React state

```tsx
const [columns, setColumns] = useState<ColumnProps[]>([
  { field: 'EmployeeID', headerText: 'ID', isPrimaryKey: true, width: 80 },
  { field: 'Name', headerText: 'Name' },
]);

const addSalaryColumn = () => {
  setColumns(prev => prev.some(c => c.field === 'Salary')
    ? prev
    : [...prev, { field: 'Salary', headerText: 'Salary', format: 'C0', textAlign: TextAlign.Right }]);
};

<Columns>
  {columns.map((c, i) => <Column key={c.field ?? i} {...c} />)}
</Columns>
```

- Adding/removing columns triggers a **full grid refresh** — reinitialize layout. Always memoize the `<Grid>` and use stable keys (`c.field`, not index, when the dataset is editable).
- Toggling `visible` is a **lightweight UI refresh** (does not reset internal grid state). Pair with a `Checkbox` + state to build a runtime visibility toggle.

## Complex (nested) data binding

```tsx
<Column field="Name.FirstName" headerText="First Name" />
<Column field="Customer.CustomerID" />           // requires new Query().expand('Customer')
<Column field="Customer.Details.Address.City" /> // requires Query().expand('Customer.Details.Address')
```

Local: works automatically as long as your data has the nested shape. Remote (`DataManager`): add `expand('Path')` to the `Query`.

## Auto-fit modes (`autoFit`)

```tsx
<Column field="OrderID" autoFit={AutoFitMode.Header} />  // header-width
<Column field="OrderID" autoFit={AutoFitMode.Content} /> // content-width
<Column field="OrderID" autoFit={AutoFitMode.All} />     // max of header & content
```

Works with virtualization: only visible rows are measured. `minWidth`/`maxWidth` always bound the final size.

## Common column patterns

| Need | Configuration |
|---|---|
| ID column with strict no-edit | `field='id' isPrimaryKey allowEdit={false}` |
| Fixed-width status badge | `template={statusTemplate} clipMode={ClipMode.EllipsisWithTooltip}` |
| Action buttons column | `getCommandItems={() => [<CommandItem key='edit' />]}` (require `CommandColumnModule`) |
| Hidden helper field used by templating | `visible={false}` |
| Sortable but not searchable | `field='foo' headerText='Foo'` (search toolbar respects only `searchSettings.fields`) |
| Numeric range editing with min/max | `type={ColumnType.Number} edit={{ type: EditType.NumericTextBox, params: { min: 0, max: 1000, format: 'N1' }}}` |
| Date column with strict date filter UI | `type={ColumnType.Date} filter={{ filterBarType: FilterBarType.DatePicker }}` |

## Constraints specific to columns

- `Column.field` uniqueness: `isPrimaryKey` should be set on exactly one column per grid.
- `headerTemplate` and `template` must be memoized with `useCallback` (and components wrapped in `React.memo`) to avoid re-renders.
- `editTemplate` must call `props?.onChange(value)` to push back cell state.
- `cellClass`/`valueAccessor` run on every render — keep them pure and cheap, return strings, avoid allocations.
- `autoHeight` disables column virtualization → use sparingly.
- Parent header columns (multi-level headers) cannot have a `field` — text only.