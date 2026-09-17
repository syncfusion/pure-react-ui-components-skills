---
name: editing
description: Editing configuration for the Syncfusion React Data Grid — EditSettings modes (Normal/Cell/Popup/PopupTemplate/Dialog/Inline/Batch), toolbar entries, validation rules (built-in + custom), command column with CommandItem, programmatic CRUD methods (addRecord, editRecord, deleteRecord, saveDataChanges, cancelDataChanges, setCellValue, setRowData), and lifecycle events. Load when enabling add/edit/delete, building inline forms, writing validation, or wiring command-column action buttons.
---

# Editing — configuration

The grid supports seven editing modes and three layers of customization: built-in editors, template editors, and column-level read-only behavior. This file covers the shared configuration; mode-specific guidance lives in `references/editing-modes.md`.

## Modules and toolbar

Editing requires `EditModule`. The toolbar accepts the names `'Add'`, `'Edit'`, `'Delete'`, `'Update'`, `'Cancel'` (each fires only when its action is allowed by `EditSettings`).

```tsx
import { Grid, Columns, Column,
         EditSettings, SortSettings, PageSettings, FilterModule, PagerModule, EditModule } from '@syncfusion/react-grid';

const modules = { EditModule, FilterModule, PagerModule };
const [sort] = useState<SortSettings>({ enabled: true });
const [page] = useState<PageSettings>({ enabled: true, pageSize: 8, pageCount: 4 });
const [edit] = useState<EditSettings>({
  mode: 'Normal',          // default
  allowEdit: true,
  allowAdd: true,
  allowDelete: true,
  confirmOnDelete: true,   // shows the confirm dialog when true
});

<Grid
  dataSource={data}
  editSettings={edit} sortSettings={sort} pageSettings={page}
  toolbar={['Add', 'Edit', 'Delete', 'Update', 'Cancel']}
  modules={modules}
>
  <Columns>
    <Column field="OrderID" headerText="ID" isPrimaryKey />
    {/* other columns */}
  </Columns>
</Grid>
```

## `EditSettings` shape

```ts
{
  mode?: 'Normal' | 'Dialog' | 'Inline' | 'Batch' | 'Cell' | 'Popup' | 'PopupTemplate';
  allowEdit?: boolean;             // default false
  allowAdd?: boolean;              // default false
  allowDelete?: boolean;           // default false
  confirmOnDelete?: boolean;       // default false
  popupSettings?: DialogLikeSettings;
  popupTemplate?: (props: PopupTemplateProps) => React.ReactElement;
}
```

Defaults are **false** for `allowEdit`/`allowAdd`/`allowDelete` and `'Normal'` for `mode`. `confirmOnDelete` opens the built-in confirmation dialog when true (localizable via `L10n.load`; see `references/globalization.md`).

## Primary key requirement

**Editing and selection features require a column marked `isPrimaryKey={true}`.** Without a primary key, edits/deletes silently target the wrong rows (often the first row only).

```tsx
<Column field="OrderID" isPrimaryKey validationRules={{ required: true, number: true }} />
```

## Per-column read-only

`allowEdit={false}` makes a column read-only during both add and edit, regardless of mode:

```tsx
<Column field="EmployeeID" isPrimaryKey allowEdit={false} />
```

## Validation rules

Rule shape (one entry per rule — short form):

```ts
{ required: true }
{ required: true, number: true }
{ required: true, min: 1, max: 1000 }
```

Pair-style (with explicit message strings, used in PopupTemplate):

```ts
{
  required: [true, 'Order ID is required'],
  number:   [true, 'Order ID must be a number'],
  range:    [[1, 1000], 'Freight must be between 1 and 1000'],
}
```

Available rules (with their pair value shape):

| Rule | Value shape |
|---|---|
| `required` | `[true, msg]` |
| `number` | `[true, msg]` |
| `digits` | `[true, msg]` (digits only) |
| `email` | `[true, msg]` |
| `url` | `[true, msg]` |
| `date` | `[true, msg]` |
| `dateIso` | `[true, msg]` |
| `creditCard` | `[true, msg]` |
| `tel` | `[true, msg]` |
| `equalTo` | `['fieldName', msg]` |
| `minLength` / `maxLength` | `[n, msg]` |
| `rangeLength` | `[[min, max], msg]` |
| `min` / `max` / `range` | numeric `[n, msg]` or `[[min, max], msg]` |
| `regex` | `[/pattern/, msg]` |

Custom validators:

```ts
validationRules={{
  required: true,
  customValidator: (value: FormValueType): string | null => {
    if (typeof value !== 'string' || !value.startsWith('ID-')) {
      return 'Customer ID must start with "ID-".';
    }
    return null;
  },
}}
```

Render into `.sf-error` (single input) or `.sf-form-error` (form/message container).

## Edit types (`Column.edit`)

```ts
<Column field="Price" type={ColumnType.Number} format="C2"
        edit={{ type: EditType.NumericTextBox, params: { min: 0, max: 1000, format: 'N1', decimals: 0, spinButton: false } }} />
<Column field="Status"
        edit={{ type: EditType.DropDownList, params: { popupSettings: { height: '150px' } } }} />
<Column field="Active" type={ColumnType.Boolean}
        edit={{ type: EditType.CheckBox, params: { label: 'Active', color: Color.Info } }} />
<Column field="StartDate" type={ColumnType.Date} format="yMd"
        edit={{ type: EditType.DatePicker, params: { minDate: new Date(2025, 0, 1), maxDate: new Date(2025, 1, 28), format: 'dd/MM/yyyy' } }} />
```

`EditType` enum values used in samples: `TextBox`, `NumericTextBox`, `DropDownList`, `CheckBox`, `DatePicker`, `RichTextEditor`, `Slider`, `Switch`, `TextArea`, `MaskedTextBox`, `AutoComplete`, `MultiSelect`, `ComboBox`, `TimePicker`, `DateTimePicker`.

`edit.params` accepts a partial intersection of the underlying input's Props plus `labelMode?: string`. Default `edit.type` when omitted: `EditType.TextBox`.

The EditType.NumericTextBox type uses the Syncfusion React NumericTextBox component for editing numeric data. The numeric textbox can be customized using the edit.params property, which supports attributes such as decimals, format, spinButton, and other numeric textbox specific properties.

```tsx
    const editParams = {
        type: EditType.NumericTextBox,
        params: {
            format: 'N1',
            min: 1, 
            max: 1000
        }
    };
    <Column field='UnitsSold' headerText='Units Sold' width='100' edit={editParams} validationRules={numberValidationRules} filter={{filterBarType:FilterBarType.NumericTextBox}} textAlign={TextAlign.Right} />
```

The EditType.DropDownList type uses the Syncfusion® React DropDownList component for editing string data with predefined values.

Popup customization
The following example demonstrates popup height customization through the edit.params property for the "Ship Country" column.

```tsx
    const dropDownEdit = {
    type: EditType.DropDownList,
    params: {
      popupSettings: { height: '150px' },
    },
  };

   <Column field='ShipCountry' headerText='Ship Country' edit={dropDownEdit} width='160'></Column>
```

The EditType.CheckBox type uses the Syncfusion® React CheckBox component for editing boolean data. The checkbox can be customized using the edit.params property, which supports attributes such as label, color and other checkbox specific properties.
```tsx
    const booleanEdit = {
        type: EditType.CheckBox,
        params: {
            color: Color.Info,  
        } as Partial<TextBoxProps & NumericTextBoxProps & CheckboxProps & DatePickerProps & DropDownListProps> & { color?: Color },
    };    

    <Column field="IsAvailable" headerText="Item Availability" width={120} displayAsCheckBox={true} type={ColumnType.Boolean} edit={booleanEdit} textAlign={TextAlign.Center}  />
```

Default editor is inferred from `Column.type` when both are set; `Column.edit.type` overrides.

## Custom edit template

```tsx
const countryTemplate = useCallback((props?: EditTemplateProps) => (
  <DropDownList dataSource={countries} fields={{ text: 'text', value: 'value' }}
    value={(props?.data as SalesRecord)?.Branch}
    onChange={(args?: ChangeEvent) => props?.onChange(args?.value as string)} />
), []);
```

**Always call `props?.onChange(value)`** to push the value into the grid's editing state. For complex (nested) addresses, name the editor input `name='Path__SubPath'` (e.g., `name='Name__FirstName'`) — the grid maps underscores back to dots.

## Programmatic CRUD (`GridRef`)

```tsx
gridRef.current?.addRecord({ /* optional row data, default empty */ });
gridRef.current?.addRecord({ BookID: 9000, Title: 'Satire', Author: 'Joe' });   // prefill
gridRef.current?.editRecord();
gridRef.current?.deleteRecord();
gridRef.current?.saveDataChanges();
gridRef.current?.cancelDataChanges();
gridRef.current?.updateRecord(/* index */ 3, /* row */ { ...data[3], approved: true });
gridRef.current?.setCellValue(/* primaryKey */ 'K1', /* field */ 'Status', /* value */ 'Approved', /* triggerEvent? */ true);
gridRef.current?.setRowData(/* primaryKey */ 'K1', /* data */ newRow, /* triggerEvent? */ true);
```

- `setCellValue`/`setRowData` final arg `true` updates the underlying source and raises change events; default updates only the UI.
- Cell mode uses `editCell(primaryKey, columnField)` / `saveCellChanges()` / `cancelCellChanges()` instead of the row-level trio.

### Resolve row index from primary key (for templates)

```tsx
const rowIndex = gridRef.current?.getCurrentViewRecords?.()
  ?.findIndex((row: T) => row[primaryKeyField] === primaryKey);
```

## Events

| Event | Args type | Fires |
|---|---|---|
| `onRowAddStart` | `RowEditEvent` | Before new row is inserted. `args.cancel = true` aborts. |
| `onRowEditStart` | `RowEditEvent` | Before edit begins. `args.cancel = true` aborts. |
| `onCellEditStart` | event payload | Cell mode: before a cell enters edit (Normal mode does not fire this). |
| `onDataChangeStart` | `DeleteEvent \| SaveEvent` | Pre-commit hook for add/edit/delete. Modify `args.data` (replacement values) or set `args.cancel = true`. |
| `onDataChangeComplete` | `DeleteEvent \| SaveEvent` | Fires after commit. |
| `onRowSelect` | `RowSelectEvent` | When a row is selected or its current selection changes. |

`DeleteEvent`: `action: 'Delete'`, `data: T[]`. `SaveEvent`: `action: 'add' | 'edit'`, `data: T | T[]`. `ActionType` values used here: `Add`, `Save`, `Delete`, `DeleteConfirm`, `Refresh`.

```tsx
const onDataChangeStart = useCallback((args: DeleteEvent | SaveEvent) => {
  if (args.action === 'Delete') {
    const records = (args as DeleteEvent).data as Employee[];
    if (records[0]?.Role === 'Manager') { args.cancel = true; }
  } else if (args.action === 'add') {
    /* mutate args.data before commit */
  }
}, []);
```

## Command column

Inline action buttons (edit/delete/update/cancel/custom) inside a cell:

```tsx
import { Column, CommandColumnModule, CommandItem, CommandItemType } from '@syncfusion/react-grid';

const modules = { /* ..., */ CommandColumnModule };

<Column headerText="Actions" width="120" getCommandItems={() => [
  <CommandItem key="edit" type={CommandItemType.Edit} />,
  <CommandItem key="delete" type={CommandItemType.Delete} />,
  <CommandItem key="cancel" type={CommandItemType.Cancel} />,
  <CommandItem key="update" type={CommandItemType.Update} />,
  <CommandItem key="custom">
    <span onClick={(e) => {
      const tr = (e.currentTarget as HTMLElement).closest('tr');
      const idx = tr?.sectionRowIndex ?? -1;
      gridRef.current?.clearSelection();
      gridRef.current?.selectRow(idx);
      gridRef.current?.deleteRecord();
    }}>
      <DeleteIcon />
    </span>
  </CommandItem>,
]}>
```

- `CommandItem.key` is one of `'edit'`, `'delete'`, `'update'`, `'cancel'`, `'custom'` (default icon/action binds through `CommandItemType`).
- Custom command uses `key='custom'` and any nested React element; the click handler is yours.

## Toolbar integration edit modes

| Toolbar name | Effect |
|---|---|
| `'Add'` | Triggers `addRecord()` |
| `'Edit'` | Triggers `editRecord()` on selected row |
| `'Delete'` | Triggers `deleteRecord()` (with confirm when `confirmOnDelete`) |
| `'Update'` | Triggers `saveDataChanges()` |
| `'Cancel'` | Triggers `cancelDataChanges()` |

### Toolbar click event

Wired through `<Grid onToolbarItemClick={handler}>` (note: **`onToolbarItemClick`, not `onToolbarClick`**). The handler receives a `ToolbarClickEvent`:

```ts
interface ToolbarClickEvent {
  item: ToolbarItemProps;  // { id: string; text?: string; ... } — match button via item.id
  event?: Event;
  cancel?: boolean;        // set to true to abort the button's default action
}

interface ToolbarItemProps {
  id: string;              // matches the string used in toolbar=[...] ('Add', 'Edit', ...)
  text?: string;
  icon?: React.ReactNode;
  disabled?: boolean;
  onClick?: () => void;
  title?: string;
}
```

```tsx
const onToolbarItemClick = useCallback((args: ToolbarClickEvent) => {
  switch (args.item.id) {
    case 'Add':    /* pre-fill, log, etc. */ break;
    case 'Edit':   /* pre-flight checks */   break;
    case 'Delete': /* bulk-delete gating */  break;
    case 'Update': /* commit-side hook */    break;
    case 'Cancel': /* rollback bookkeeping */ break;
  }
}, []);

<Grid onToolbarItemClick={onToolbarItemClick} toolbar={['Add','Edit','Delete','Update','Cancel']} ... />
```

> The skill's older documentation describes `args.item` as a plain string and the prop as `onToolbarClick`. Both are wrong in `@syncfusion/react-grid` v1.1 — use `args.item.id` and `onToolbarItemClick` respectively. Set `args.cancel = true` to veto the built-in button behavior.

## Per-mode quick reference

| Mode | Edit form shape | Methods |
|---|---|---|
| `'Normal'` (default) | Add a row below, edit in-place when double-clicked or via F2 | `addRecord`, `editRecord`, `updateRecord`, `saveDataChanges`, `cancelDataChanges`, `deleteRecord` |
| `'Cell'` | Each cell editable directly (F2/dbl-click/tab) | `editCell`, `saveCellChanges`, `cancelCellChanges`, `setCellValue` |
| `'Popup'` | Modal dialog with column editors and `popupSettings` | row-level methods |
| `'PopupTemplate'` | Modal with a custom React form via `popupTemplate` | row-level methods (inject edited form via `args.data`) |
| `'Dialog'` | Modal dialog with default form | row-level methods |
| `'Inline'` | Inline row of editors under the row | row-level methods |
| `'Batch'` | Cells stack changes into a single save | per-cell methods |

See `references/editing-modes.md` for the full Native / Cell / Popup / PopupTemplate / custom-edit walkthroughs.

## Constraints & guardrails

- **isPrimaryKey** is mandatory for edit/add/delete and any `setCellValue`/`setRowData` flow. Don't ship without one.
- **`onDataChangeStart` discrimination**: `args.action === 'Delete'` carries `data: T[]` (an array); `'add'` and `'edit'` carry `data: T | T[]`. Type-guard before mutating.
- **Boolean column single-click update**: pair a Checkbox `template` with `updateRecord(rowIndex, data)` after looking up the index via `getCurrentViewRecords()`.
- **Custom validators** return `null` for pass, `string` for fail. Their error renders into `.sf-error` / `.sf-form-error`.
- **Confirm-on-delete + `L10n.load`**: respect localization keys (`confirmDeleteMessage`, `okButtonLabel`, `cancelButtonLabel`) when the app ships in multiple languages.
- **High-risk action gating**: mass deletes via remote APIs should be confirmed in the application layer — guard with the guardrail check before bulk operations.
- **Integration**: register `EditModule`, declare `isPrimaryKey`, wire the toolbar entries you actually support. A grid that lists `'Delete'` in toolbar but lacks `allowDelete` will visually render a button that does nothing — that's a deployment gap to avoid.