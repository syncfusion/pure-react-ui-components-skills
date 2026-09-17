---
name: grouping-and-aggregates
description: Group rows by columns and compute footer / caption aggregates for the Syncfusion React Data Grid — GroupSettings, GroupType ('GroupRows'/'SingleColumn'/'MultipleColumns'), shouldExpandGroup, groupCaptionAggregateType, aggregates with Sum/Average/Min/Max/Count/TrueCount/FalseCount/Custom, footerTemplate, and cellClass. Load when grouping rows, building aggregate footers, computing custom summaries, or styling group captions.
---

# Grouping & aggregates

Group rows by column(s) and compute per-column aggregates — footer rows or group captions.

## Modules

```tsx
import { Grid, Columns, Column,
         FilterModule, PagerModule, GroupModule, AggregateModule,
         GroupSettings, GroupType, AggregateType } from '@syncfusion/react-grid';

const modules = { FilterModule, PagerModule, GroupModule, AggregateModule };
```

`SortModule` is built into the grid — sort by the grouping column is required for ordered groups.

## Group enable

```tsx
import { GroupSettings } from '@syncfusion/react-grid/src/grid/types/grouping.interfaces';

const [group] = useState<GroupSettings>({
  enabled: true,
  columns: ['country'],                  // initial grouped column(s)
  showDropArea: true,                     // drag-and-drop header chips to group
  defaultExpanded: true,                  // or a number (e.g., 1 to expand first level only)
  type: GroupType.GroupRows,             // default
});

<Grid dataSource={data}
      groupSettings={group}
      sortSettings={{ enabled: true }}
      filterSettings={{ enabled: true }}
      pageSettings={{ enabled: true, pageSize: 12 }}
      modules={modules} />
```

`GroupSettings` shape:

```ts
{
  enabled: boolean;
  columns?: string[];                       // field names of grouping columns
  showDropArea?: boolean;                   // drag-to-group UI
  defaultExpanded?: boolean | number;       // `true` expands all; number for top-N levels
  type?: GroupType;
}
```

## Group display type

`GroupType` enum:

| Value | Behavior |
|---|---|
| `GroupType.GroupRows` (default) | Caption rows with expand/collapse. |
| `GroupType.SingleColumn` | Single column with indentation; no separate caption rows. Requires `<Column type={ColumnType.SingleGroup} visible={visibleFlag} />`. |
| `GroupType.MultipleColumns` | Each grouped field shown as its own column. No caption rows. |

Toggle the type via state to show all three side-by-side:

```tsx
const [group, setGroup] = useState<GroupSettings>({ enabled: true, columns: ['country', 'olympicYear'], type: GroupType.GroupRows, defaultExpanded: 1 });

const setType = (newType: GroupType) => setGroup(prev => ({ ...prev, type: newType }));
```

The order of columns in `<Columns>` determines grouping order.

## Per-column allowGroup

`allowGroup={false}` prevents drag-to-group for that column. Often combined with `visible={false}` to hide the source column while still grouping by it.

## `shouldExpandGroup` callback

```tsx
const shouldExpandGroup = useCallback((props: ShouldExpandGroupEvent) => {
  // props: { groupKey: string }
  return ['Alice', 'Fiona'].includes(props.groupKey);
}, []);

<Grid shouldExpandGroup={shouldExpandGroup} ... />
```

## Group caption customization

`Column.groupCaptionAggregateType` shows an aggregate inline in the caption. Combined with a custom `template` you can fully control the caption cell:

```tsx
const groupCaptionTemplate = useCallback((props?: ColumnTemplateProps<T>) => {
  const data = props?.data as T | GroupedData<T>;
  const isCaption = (data as GroupedData<T>)?.flattedKey
                 && (data as T)?.[props?.column?.field as keyof T];

  if (isCaption) {
    const label = props?.column?.groupCaptionAggregateType === 'Sum' ? 'Total'
                 : props?.column?.groupCaptionAggregateType;
    const formatted = props?.column?.formatFn?.(Number((data as T)?.[props?.column?.field as keyof T])) ?? (data as T)?.[props?.column?.field as keyof T];
    return <b>{label + ': ' + formatted}</b>;
  }
  return <>{String((data as T)?.[props?.column?.field as keyof T] ?? '')}</>;
}, []);

<Column field='Gold' headerText='Gold' type={ColumnType.Number}
        groupCaptionAggregateType={AggregateType.Sum}
        template={groupCaptionTemplate}
        formatFn={(v: number) => `${v} medals`} />
```

Group caption aggregates are supported for `SingleColumn` and `MultipleColumns` only — `GroupRows` does not render caption aggregates.

## Aggregates

```tsx
import { Grid, Aggregates, AggregateRow, AggregateColumn, AggregateType } from '@syncfusion/react-grid';

<Grid dataSource={data} modules={modules}>
  <Columns>
    <Column field='id' headerText='ID' />
    <Column field='Product' headerText='Product' />
    <Column field='Price' headerText='Price' format='C2' textAlign={TextAlign.Right} />
    <Column field='InStock' headerText='Stock' type={ColumnType.Number} />
  </Columns>
  <Aggregates>
    <AggregateRow>
      <AggregateColumn field='Price' type={AggregateType.Sum} format='C2'
                       footerTemplate={(props) => <span>Total: ${props?.Sum?.toFixed(2)}</span>} />
      <AggregateColumn field='InStock' type={AggregateType.Max} format='N0' />
      <AggregateColumn field='id' type={AggregateType.Count} format='N0' />
    </AggregateRow>
    <AggregateRow>
      <AggregateColumn field='Price' type={AggregateType.Average} format='C2' />
    </AggregateRow>
  </Aggregates>
</Grid>
```

### `AggregateType` enum

| Value | Description | Available on `args` |
|---|---|---|
| `Sum` | Sum | `Sum` |
| `Average` | Mean | `Average` |
| `Min` | Min | `Min` |
| `Max` | Max | `Max` |
| `Count` | Non-null count | `Count` |
| `TrueCount` | Boolean true count | `TrueCount` |
| `FalseCount` | Boolean false count | `FalseCount` |
| `Custom` | User-defined | `Custom` |

### `AggregateColumn` props

```ts
{
  field: string;
  type: AggregateType;
  format?: string;                          // e.g., 'C2', 'N0', 'yMd', 'P0'
  cellClass?: (props: CellClassProps) => string;
  customAggregate?: (data: object[] | object, column: AggregateColumnProps) => number;  // required when type === 'Custom'
  footerTemplate?: (props: TemplateProps) => React.ReactElement;
}
```

### `Custom` aggregate

```tsx
customAggregate: (data, column) => {
  const rows = Array.isArray(data) ? data : (data as { result: T[] })?.result ?? [];
  // e.g., median
  const values = rows.map((row: T) => Number((row as Record)[column.field!])) ?? [];
  values.sort((a, b) => a - b);
  return values[Math.floor(values.length / 2)] ?? 0;
}
```

### Multiple aggregates for one field

Define multiple `<AggregateRow>` blocks (one per aggregate computation). Each row renders in the footer area.

## Footer template signature

`footerTemplate` receives the computed aggregate keyed by `AggregateType`:

```tsx
footerTemplate={(props) => <span>Total: ${Number(props?.Sum).toFixed(2)}</span>}
```

Available keys match the `AggregateType` value applied to that column (`Sum`, `Average`, `Min`, `Max`, `Count`, `TrueCount`, `FalseCount`, `Custom`). Aggregate result is **untyped** in many flows — narrow via props key access and your format string.

## Group caption aggregates

`Column.groupCaptionAggregateType` is independent from footer aggregates. Use it to show the running aggregate alongside each group header (e.g., "Total: 12,400"). Pair with `template` for full styling control.

## Common patterns

| Need | Configuration |
|---|---|
| Group by a single category | `groupSettings={{ enabled: true, columns: ['category'] }}` |
| Multi-level grouping | `columns: ['country', 'state']` |
| Show drop area | `showDropArea: true` |
| Indented grouping (no caption rows) | `type: GroupType.SingleColumn` + `<Column type={ColumnType.SingleGroup}>` |
| Inline group total | `Column.groupCaptionAggregateType={AggregateType.Sum}` + `template={groupCaptionTemplate}` |
| Footer total only | `<Aggregates>` with one `<AggregateRow>` |
| Two aggregates per field | `<AggregateRow>` × 2 with the same field, different `type` |

## Constraints & guardrails

- **Caption aggregates** are only supported by `GroupType.SingleColumn` and `GroupType.MultipleColumns` — not by `GroupRows` (single-cell caption per group).
- `groupSettings` requires `sortSettings.enabled: true` — sort is what defines group ordering.
- `cellClass` as a function runs every render — for high-frequency aggregates prefer `footerTemplate`.
- **Integration**: when adding aggregates + grouping to a virtualized grid, footer rows participate in viewport math and may double-render during scroll. Set `viewPortBuffer: { rows: 5, columns: 5 }` to stabilize.
- **Custom aggregate signature**: `customAggregate(data: object[] | object, column: AggregateColumnProps) => number`. The data can be an array or a wrapper ({ result }); guard both.
- **Grouping is not supported with Infinite scroll** (`ScrollMode.Infinite`): infinite scroll requires the full dataset shape that grouping alters.
- **Aggregates are not supported with Infinite scroll** for the same reason.
- **Disable grouping/aggregates** when switching to infinite mode (see `references/performance-and-scrolling.md`).