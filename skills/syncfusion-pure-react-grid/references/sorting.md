---
name: sorting
description: Sorting configuration for the Syncfusion React Data Grid — enabling, multi-column sort, initial sort, per-column disable, custom sort comparers, programmatic sort calls (sortByColumn, removeSortColumn, clearSort), and the onSort lifecycle event. Load when introducing sort behaviour, building custom comparers (including culture-aware sorting), or wiring up programmatic sort calls.
---

# Sorting

Sort is built-in; no module registration needed for basic behaviour. The header click cycle is **ascending → descending → unsorted**, with multi-column support (ctrl/click + shift/click).

## Enable

```tsx
import { Grid, SortSettings } from '@syncfusion/react-grid';

const [sort] = useState<SortSettings>({ enabled: true });

<Grid dataSource={data} sortSettings={sort} />
```

Header click cycles through states automatically. Click on a header cell raises `onSort`.

## `SortSettings`

```ts
{
  enabled: boolean;                          // default false
  mode?: 'Multiple' | 'Single';              // default 'Multiple'
  allowUnsort?: boolean;                     // default true
  columns?: Array<{ field: string; direction: SortDirection }>;
}
```

- `mode: 'Multiple'` (default) — Ctrl/click adds sort; Shift/click removes that column; plain click sets single column.
- `mode: 'Single'` — click replaces the active sort.
- `allowUnsort: false` — skips the unsorted state in the click cycle (only asc → desc).

`SortDirection` enum: `SortDirection.Ascending` and `SortDirection.Descending`.

## Initial sort

```tsx
new SortSettings({ enabled: true, columns: [{ field: 'Quantity', direction: SortDirection.Ascending }] });

// or via state:
const [sort] = useState<SortSettings>({
  enabled: true,
  columns: [{ field: 'Department' }, { field: 'Name' }],   // ascending default
});
```

Direction is optional — when omitted, defaults to ascending.

## Per-column disable

```tsx
<Column field="EmployeeID" allowSort={false} />
```

## Custom comparer

```tsx
const size = useCallback((col: ColumnProps) => {
  return (a: any, b: any) => {
    const A = parseFloat((a[col.field!] ?? '').toString());
    const B = parseFloat((b[col.field!] ?? '').toString());
    return A < B ? -1 : A > B ? 1 : 0;
  };
}, []);
<Column field="Size" sortComparer={size} />
```

`sortComparer` signature:

```ts
(
  referenceValue: ValueType,         // or whatever input type the column has
  comparerValue: ValueType,
  referenceRowData?: object,
  comparerRowData?: object,
  sortDirection?: 'Ascending' | 'Descending' | 'unsort',
) => -1 | 0 | 1
```

Culture-aware sort: use `Intl.Collator` for locale-correct comparison plus `<Provider locale={...}>` (and CLDR data via `loadCldr`). Keep the comparer pure and stable (return numbers, not booleans); long sort pipelines may invoke thousands of times per scroll.

Default null behaviour: nulls at top under descending, bottom under ascending. Custom comparers should explicitly handle nulls to override.

## Programmatic sort

```tsx
const gridRef = useRef<GridRef>(null);

gridRef.current?.sortByColumn('ServiceItem', 'Ascending');
gridRef.current?.sortByColumn('Amount',  'Descending', /*isMultiSort*/ true);
gridRef.current?.removeSortColumn('ServiceItem');
gridRef.current?.clearSort();
```

`sortByColumn(field, direction, isMultiSort?)` — `isMultiSort = true` appends without clearing; otherwise replaces.

## `onSort` event

`SortEvent`: `{ action: ActionType, field: string, direction: SortDirection }`.

`ActionType` values for sort: `Sorting` (a column was sorted) and `ClearSorting` (sort was removed).

```tsx
const handleSort = useCallback((args: SortEvent) => {
  if (args.action === ActionType.Sorting) {
    console.log(`Sorted ${args.field} ${args.direction}`);
  } else if (args.action === ActionType.ClearSorting) {
    console.log('Sort cleared');
  }
}, []);

<Grid onSort={handleSort} ... />
```

## Accessibility

Sorted columns get `aria-sort="ascending"` / `"descending"` / `"none"`. Keyboard: `Enter` toggles sort; `Ctrl+Enter` (or `⌘+Enter`) for multi-column; `Shift+Enter` clears sort on the focused column.

## Common patterns

| Need | Configuration |
|---|---|
| Multi-column sort | `sortSettings={{ mode: 'Multiple' }}` (default) |
| Single-column only | `sortSettings={{ mode: 'Single' }}` |
| Initial sort for new visitors | `sortSettings={{ columns: [{ field: 'Date', direction: SortDirection.Descending }] }}` |
| Skip unsorted state | `sortSettings={{ allowUnsort: false }}` |
| Locale-aware string sort | `sortComparer={Intl.Collator(locale).compare}` wrapping |
| Lock a primary-key column | `<Column field="ID" allowSort={false} />` |

## Constraints

- Many sort comparers called per scroll → keep them allocation-free and cheap.
- `onSort`'s `field` may be undefined for clear-all-sort; check `args.action === ActionType.ClearSorting` first.
- `sortByColumn(field, direction, true)` only appends if there is already a sort; otherwise it sets the first column.