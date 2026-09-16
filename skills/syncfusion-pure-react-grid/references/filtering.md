---
name: filtering
description: All filtering modes and operators for the Syncfusion React Data Grid — FilterBar (default), Menu, Excel, CheckBox, custom filters, programmatic filterByColumn/clearFilter, and per-column filter UI (TextBox/NumericTextBox/DatePicker, custom filterTemplate, expressions). Load when the user asks about filters, search-as-you-type, multi-criteria filters, Excel-style checkboxes, or dynamic filter UI.
---

# Filtering

The grid ships **four** distinct filter UI modes, all driven by `filterSettings`:

| Mode | UI |
|---|---|
| `'FilterBar'` (default) | One input per column header with built-in operators |
| `'Menu'` | Per-column popup with operator dropdowns + combined AND/OR predicates |
| `'Excel'` | Excel-style popup with searchable checkbox list, sort, operator list |
| `'CheckBox'` | Checkbox list with virtual scroll, search, "Add current selection" |

All four share the same `Column.filter` API and the same `onFilter` lifecycle event.

## Enable

```tsx
import { Grid, FilterModule, FilterSettings, SortSettings, PageSettings } from '@syncfusion/react-grid';

const modules = { FilterModule };
const [sort]   = useState<SortSettings>({ enabled: true });
const [filter] = useState<FilterSettings>({ enabled: true, type: 'CheckBox' });
const [page]   = useState<PageSettings>({ enabled: true, pageSize: 8, pageCount: 4 });

<Grid dataSource={data} sortSettings={sort} filterSettings={filter} pageSettings={page} modules={modules} />
```

## `FilterSettings` shape

```ts
{
  enabled: boolean;                          // default false
  type?: 'FilterBar' | 'Menu' | 'Excel' | 'CheckBox';   // default 'FilterBar'
  mode?: 'Immediate' | 'OnEnter';             // default 'Immediate'
  immediateModeDelay?: number;               // default 1500 ms
  caseSensitive?: boolean;                   // default false
  ignoreAccent?: boolean;                    // default false
  columns?: FilterPredicates[];              // initial/preset filter set
  operators?: Partial<Record<OperatorGroup, Array<{ value: string; text: string }>>>;
  enableFilterBarOperator?: boolean;         // only with type 'FilterBar'
}
```

`OperatorGroup` values for `operators` (Menu mode only): `stringOperator`, `numberOperator`, `dateOperator`, `datetimeOperator`, `dateonlyOperator`, `booleanOperator`.

## Filter modes

### FilterBar (default)

Per-column header input. Mode `Immediate` applies on a `1500ms` debounce; mode `OnEnter` waits for Enter (recommended for remote backends).

Filter-bar expressions (TextBox only — not NumericTextBox/DatePicker): symbols like `=`, `!=`, `>`, `<`, `>=`, `<=`, `*`, `%`. Custom `filterBarType` disables expression syntax.

```tsx
<Column field="EnergyConsumption" headerText="Energy"
        filter={{ filterBarType: FilterBarType.NumericTextBox }} />
<Column field="StartDate" type={ColumnType.Date} format="yMd"
        filter={{ filterBarType: FilterBarType.DatePicker }} />
<Column field="Amount" format="C2" textAlign={TextAlign.Right}
        filter={{ filterBarType: FilterBarType.NumericTextBox }} />
```

Operator dropdown — toggle via `enableFilterBarOperator: true` (FilterBar only). Combine with per-column `filter.operator` (default) and `filter.filterOperators` (whitelist).

### Menu

Popup with operator dropdowns and type-specific input controls. Supports AND/OR predicates inside the popup.

```tsx
const [filter] = useState<FilterSettings>({ enabled: true, type: 'Menu' });
<Grid filterSettings={filter} ... />
```

Restrict operator lists:

```tsx
filterSettings: {
  enabled: true, type: 'Menu',
  operators: {
    stringOperator: [{ value: 'contains', text: 'Includes' }, { value: 'startsWith', text: 'Begins with' }],
    numberOperator: [{ value: 'greaterThan', text: 'Above' }],
  },
}
```

Date range preset:

```tsx
filterSettings: {
  enabled: true, type: 'Menu',
  columns: [
    { field: 'OrderDate', operator: 'greaterThanOrEqual', value: new Date(2025, 1, 1) },
    { field: 'OrderDate', operator: 'lessThanOrEqual', value: new Date(2025, 2, 31), predicate: 'and' },
  ],
}
```

### Excel

```tsx
const [filter] = useState<FilterSettings>({ enabled: true, type: 'Excel' });
```

Excel-style popup with searchable checkbox list, sort options, value search, operator list. Combine with `mode: 'Immediate'` for instant filtering on selection.

Per-column override: `<Column filter={{ type: 'CheckBox' }} />` to flip a single column to checkbox dropdown.

`onFilterDialogBeforeOpen` event lets you mutate popup options: `args.options.disableSearchOption`, `args.options.disableSortOption`, `args.options.dataSource`. `args.columnName: string` is the column id.

### CheckBox

```tsx
const [filter] = useState<FilterSettings>({ enabled: true, type: 'CheckBox' });
```

Virtualized checklist with search. After a search, an "Add current selection" option enables cumulative selections.

Per-column `<Column filter={{ type: 'Excel' }} />` flips one column to Excel-style.

## `FilterPredicates` (initial/preset + custom paths)

```ts
{
  field: string;
  operator: string;                          // any operator below
  value: string | number | Date | boolean | null;
  predicate?: 'and' | 'or';                  // join with same-field siblings
  ignoreAccent?: boolean;
}
```

## Filter operators

| Operator | String | Number/Date/Boolean |
|---|---|---|
| `startsWith`, `endsWith`, `contains`, `doesNotStartWith`, `doesNotEndWith`, `doesNotContain` | ✓ | |
| `equal`, `notEqual` | ✓ | ✓ |
| `greaterThan`, `greaterThanOrEqual`, `lessThan`, `lessThanOrEqual`, `between` | | ✓ |
| `in`, `notIn` | | ✓ |
| `isNull`, `isNotNull` | ✓ | |
| `isEmpty`, `isNotEmpty` (string only) | ✓ | |
| `wildcard` (uses `*`) | ✓ | |
| `like` (uses `%`) | ✓ | |

Wildcard pattern table (`*` semantics):
- `Tom*Lee` — starts with Tom, ends with Lee.
- `Tom*` — starts with Tom.
- `*Lee` — ends with Lee.
- `*Tom*` — contains Tom.

LIKE pattern table (`%` semantics):
- `%Tom Lee%` — contains "Tom Lee".
- `Tom Lee%` — ends with "Tom Lee".
- `%Tom Lee` — starts with "Tom Lee".

## Column-level filter config

```tsx
type ColumnFilterParams = {
  filterBarType?: FilterBarType;             // TextBox (default) / NumericTextBox / DatePicker
  operator?: string;                         // default per type
  filterOperators?: string[];                // whitelist
  type?: 'Excel' | 'CheckBox';               // override global type for this column
  params?: Partial<...> & { labelMode?: string };   // forwards props to the input
  filterTemplate?: (props: { column: ColumnProps }) => React.ReactElement;
};
```

```tsx
<Column field="Priority" headerText="Priority"
        filter={{ filterBarType: FilterBarType.TextBox, operator: 'startsWith', filterOperators: ['startsWith', 'contains', 'equal'] }} />
```

`Column.allowFilter={false}` disables the filter input / menu icon for a column.

### Custom filter template

```tsx
const filterTemplate = useCallback((props?: { column: ColumnProps }) => {
  const field = props?.column?.field;
  const op    = props?.column?.filter?.operator ?? 'contains';

  const onChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const v = e.target.value;
    if (!v) gridRef.current?.clearFilter([field]);
    else gridRef.current?.filterByColumn(field, op, v);
  };

  const onKey = (e: React.KeyboardEvent) => {
    if (e.key === 'Enter') {
      const v = (e.target as HTMLInputElement).value;
      gridRef.current?.filterByColumn(field, op, v);
    }
  };

  return <input onChange={onChange} onKeyUp={onKey} placeholder="Filter…" />;
}, []);

<Column field="Customer" filterTemplate={filterTemplate} />
```

For DatePicker/NumericTextBox/DropDownList inside a template, mirror the OnEnter pattern (`e.keyCode === 13`) via `onKeyUp`.

To suppress the default filter input on a template column while keeping the filter row visible: return `<span />` from `filterTemplate`.

## Programmatic filter

```tsx
const gridRef = useRef<GridRef>(null);

// Apply
gridRef.current?.filterByColumn(
  'Customer',
  'contains',
  'VINET',
  /*predicate*/ undefined,  // 'and' | 'or' (only when adding a second predicate)
  /*matchCase*/ undefined,  // boolean; only adds another predicate
  /*ignoreAccent*/ undefined);

// Clear all
gridRef.current?.clearFilter();

// Clear specific columns
gridRef.current?.clearFilter(['Customer', 'Country']);
```

The last three arguments apply **only** when adding a second predicate to the same column (i.e., creating a compound condition); they're ignored on the first predicate.

## `onFilter` event

```ts
FilterEvent: {
  action: ActionType.Filtering | ActionType.ClearFiltering;
  currentFilterColumn: ColumnProps;
  currentFilterPredicate: FilterPredicates | FilterPredicates[];
}
```

```tsx
const handleFilter = useCallback((args: FilterEvent) => {
  if (args.action === ActionType.Filtering) {
    const p = Array.isArray(args.currentFilterPredicate) ? args.currentFilterPredicate[0] : args.currentFilterPredicate;
    console.log(`Filtered ${p?.field} ${p?.operator} ${p?.value}`);
  }
}, []);
<Grid onFilter={handleFilter} ... />
```

## Custom filtering (programmatic / state-driven)

Two patterns:

**State-driven:** keep `FilterSettings` in state, mutate `filterSettings.columns`:

```tsx
const [filter, setFilter] = useState<FilterSettings>({ enabled: true });
const addPredicate = (field: string, op: string, val: string) => setFilter(prev => ({
  ...prev,
  columns: [...(prev.columns ?? []), { field, operator: op, value: val }],
}));
```

**Ref-driven:** call `gridRef.current?.filterByColumn` / `clearFilter` directly.

For search-as-you-type:

```tsx
import { debounce } from '@syncfusion/react-base';
const debounced = useMemo(() =>
  debounce((v: string) => gridRef.current?.filterByColumn('Product', 'contains', v), 300),
  [gridRef]);
useEffect(() => { debounced?.(searchTerm); }, [searchTerm]);
```

## Common patterns

| Need | Configuration |
|---|---|
| Inline header filter | default (FilterBar + `mode: 'Immediate'`) |
| Apply on Enter | `filterSettings={{ mode: 'OnEnter' }}` |
| Excel-like dropdown | `filterSettings={{ type: 'Excel' }}` |
| Multi-condition per column | `Menu` mode, set `predicate: 'and'/'or'` in `columns` |
| Whitelist operators | `Column.filter.filterOperators: ['equal','contains']` |
| Numeric column filter | `Column.filter.filterBarType: FilterBarType.NumericTextBox` |
| Date range | `Menu` plus two `columns` with `greaterThanOrEqual` + `lessThanOrEqual` |
| Locked column | `Column.allowFilter={false}` |

## Constraints & guardrails

- **Wildcard/LIKE filtering** only applies to string columns.
- **Filter-bar expressions** (`=`, `>`, `<=`, `*`, `%`) only work with `filterBarType: FilterBarType.TextBox`. Other types ignore symbols.
- **`filterSettings.operators`** is honored only by the **Menu** filter mode.
- **`Column.filter.type`** overrides the global type for a single column (only effective when global type is `'Excel'` or `'CheckBox'`).
- **`onFilterDialogBeforeOpen`** documented option keys (`disableSearchOption`, `disableSortOption`, `dataSource`) are the supported mutation surface. Other fields on `args.options` are not part of the public API.
- **`mode: 'Immediate'`** + remote data should be paired with `immediateModeDelay` or use `OnEnter` to limit round-trip cost.
- **`getCurrentViewRecords()`** is the helper when you need to resolve a row index by primary key inside templates (e.g., boolean single-click update).
- **Integrate the change**: when adding a filter operator, also bump the export menu (Excel/PDF) and refresh `onFilter` consumers — silent feature-regressions are most common here.