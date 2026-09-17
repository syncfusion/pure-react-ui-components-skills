---
name: performance-and-scrolling
description: Performance, virtualization, and infinite scroll configuration for the Syncfusion React Data Grid — VirtualizationSettings (Rows/Columns/Both, viewPortBuffer, enableCache), fixed vs dynamic row height tradeoff, infinite-scroll patterns via onDataRequest + DataManager, buffer tuning, and supported browser DOM height limits. Load when scaling to >1000 rows, designing streaming grids, turning on infinite scroll, or troubleshooting scroll jumpiness.
---

# Performance & scrolling

The grid is built for large datasets. Performance comes from **fixed `rowHeight`**, **virtualization** (DOM windowing), and **lazy server-driven loading**. This reference covers the knobs and the constraints.

## Quick wins (every grid should default to these)

1. **Fixed `rowHeight`** — set to the average height of your content (`36` is a good default).
2. **Memoize templates + the entire Grid JSX** — `useCallback` for `template`/`cellClass`/`valueAccessor`, `React.memo` for components, `useMemo` for the `<Grid>`.
3. **Set `enableDevMode={false}`** on every grid.
4. **`@syncfusion/react-grid` is bundled as a feature-modular package** — only register the modules you need (`PagerModule`, `FilterModule`, etc.). Use `GridAllModule` only in prototypes.

## `VirtualizationSettings`

```ts
{
  enabled?: boolean;                        // default true when `height` is set
  type?: 'Rows' | 'Columns' | 'Both';       // auto-detected if omitted
  scrollMode?: 'Auto' | 'Virtual' | 'Infinite';
  enableCache?: boolean;                    // default true
  preventMaxRenderedRows?: boolean;         // default false (overrides an internal 500-row cap)
  viewPortBuffer?: { rows: number; columns: number };  // default { rows: 5, columns: 5 }
}
```

| Setting | Default | Use when |
|---|---|---|
| `enabled` | `true` | Set `false` for small grids where DOM weight is acceptable. |
| `type: 'Rows'` | row-only virtualization | Wide tables, fewer rows. |
| `type: 'Columns'` | column-only virtualization | Many columns, narrow rows. |
| `type: 'Both'` | row + col virtualization | Many columns × many rows. |
| `scrollMode: 'Auto'` | auto-pick | Default when virtualization is enabled but unconfigured. |
| `scrollMode: 'Virtual'` | virtual scroll | Server-driven pagination chunked per buffer. |
| `scrollMode: 'Infinite'` | infinite scroll | Total count unknown; load until exhaustion. |
| `enableCache: false` | cache by default | Set false on live / streaming grids to refresh scrolls. |
| `viewPortBuffer.rows` | 5 | Increase for fast scroll on remote data; decrease to save memory. |
| `viewPortBuffer.columns` | 5 | Same logic per column. |

### Buffer tuning guidance

| Scenario | Buffer |
|---|---|
| Local data, fast scroll | 8–10 |
| Memory-constrained | 2–3 |
| Remote data, fast server | smaller |
| Remote data, slow server | larger |

A well-tuned buffer eliminates blank rows during fast scrolling and minimizes memory.

## Row height strategies

| Mode | When to use | Tradeoff |
|---|---|---|
| Fixed `rowHeight={N}` | Best perf, default for virt + streaming | All rows must visually fit `N` |
| Dynamic `getRowHeight={info => N}` | Variable content rows running on a Same row | Scrollbar jumps; perf degrades |
| `autoHeight: true` on a column | Mixed heights via templates | Disables column virtualization |

Browsing rows stay aligned when `rowHeight` is fixed. Dynamic heights and `autoHeight` are acceptable for non-virtualized grids under hundreds of rows.

## Virtual scroll (server-driven)

```tsx
const [virtualization] = useState<VirtualizationSettings>({
  enabled: true,
  scrollMode: ScrollMode.Virtual,        // 'Virtual'
  enableCache: true,
  viewPortBuffer: { rows: 5, columns: 5 },
});
const [page]                            = useState<PageSettings>({ pageSize: 50 });
const [query]                           = useState<Query>(new Query().addParams('dataCount', '100000'));

const data = new DataManager({
  url: 'https://services.syncfusion.com/js/production/api/UrlDataSource',
  adaptor: new UrlAdaptor(),
});

<Grid<T>
  dataSource={data}
  height="100%"
  query={query}
  virtualizationSettings={virtualization}
  pageSettings={page}
  clipMode={ClipMode.EllipsisWithTooltip}
/>
```

Server request: `{ skip, take, requiresCounts, sorted, filters, group, search, ... }` (UrlAdaptor POST, OData `$skip/$top`, WebApi).

Skeleton rendering between requests: place a `Skeleton` from `@syncfusion/react-notifications` inside `Column.template` while `args.data === undefined`.

## Infinite scroll

```tsx
const [page] = useState<PageSettings>({ pageSize: 50, estimatedTotalRecordsCount: 100 });
const [virtualization] = useState<VirtualizationSettings>({ scrollMode: ScrollMode.Infinite });

const [data, setData] = useState<DataResult>({ result: airportBaggageData.slice(0, 50), hasMore: true });

const onDataRequest = useCallback((args: DataRequestEvent) => {
  const dm = new DataManager(airportBaggageData);
  const q = new Query();
  const skip = args.skip ?? 0;
  const take = args.take ?? pageSettings.pageSize ?? 50;
  q.page(Math.floor(skip / take) + 1, take);
  if (args.requiresCounts) q.requiresCount();
  dm.executeQuery(q).then(e => {
    const r = (e as DataResult).result as any[];
    setData({ ...e, hasMore: airportBaggageData.length > skip + r.length } as DataResult);
  });
}, [pageSettings.pageSize]);

<Grid
  ref={gridRef}
  dataSource={data}
  onDataRequest={onDataRequest}
  virtualizationSettings={virtualization}
  pageSettings={page}
  height={400}
/>
```

End-of-data signals (auto-detected by the grid from `DataResult`):

| Backend | Field | Means end-of-data |
|---|---|---|
| OData | `@odata.nextLink` | absent |
| Azure Cosmos DB | `continuationToken` | null/absent |
| REST | `hasMore` | `false` |
| GraphQL | `nextCursor` | null/absent |
| Custom APIs | continuation field | null/absent |

### Filter/sort in infinite mode

On change, `onDataRequest` re-fires with `where`, `search`, `sort`. Translate via `Predicate` / `Query.sortBy` / `Query.search` and rebuild the `Query`. For filter dropdown choices, the grid sets `requestType === 'filterChoiceRequest'` — respond with `DataUtil.distinct`.

### Selection in infinite mode

- Selection is index-based. Define `isPrimaryKey` to stabilize selection across sort/filter refetches.
- `<Column type='checkbox' width='40' />` for checkbox selection.

### Constraints

- Aggregations and grouping are **not supported** in infinite scroll mode — both require the full dataset.
- `enableCache: false` is recommended for live streams to avoid stale data.

## Data loading strategies — choosing what fits

| Strategy | Use when |
|---|---|
| Client-side | Entire dataset fits in memory; fast access |
| Server-side (DataManager adaptor) | Known interface (OData/WebApi/Url); explicit payload |
| `onDataRequest` + custom API | Arbitrary backend; auth headers; custom framing |
| Infinite scroll | Total count unknown; COUNT not supported by backend |

For each strategy, decide upfront: virtualization (`Auto`/`Virtual`), buffer sizes, fixed vs dynamic row height.

## Browser DOM height limits (`100k+` rows)

Browsers cap scroll-area heights:

| Browser | Max height (px) | Rows @ 100 px/row |
|---|---|---|
| Chrome 120+ / Edge 120+ | ~32,000,000 | ~320,000 |
| Firefox 121+ | ~32,000,000 | ~320,000 |
| Safari 17+ | ~16,000,000 | ~160,000 |

For million-row datasets, use virtualization so the DOM never hits the limit.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Blank rows when scrolling fast | Increase `viewPortBuffer.rows` to 8–10. |
| Lag / freeze during scroll | Reduce buffer; ensure `type: Both`. |
| Scrollbar thumb wrong size | Set fixed `rowHeight`. |
| Memory growth on long sessions | Reduce buffer; set `enableCache: false`. |
| Misaligned rows on sort/filter | Force remount with `<Grid key={JSON.stringify(data)} />`. |
| Data shifts after scroll revisit | Switch from `getRowHeight` to fixed `rowHeight`. |
| `Column.autoHeight: true` causing scroll lag | Disable it, use `minHeight` on the template instead. |

## Constraints & guardrails

- **`autoHeight: true` disables column virtualization**. Avoid for grids over ~1k rows.
- **Column virtualization does not support `width='auto'`** — columns default to 100 px when width is missing.
- **Grouping + aggregates** are not supported with infinite scroll (full-dataset assumption).
- **`preventMaxRenderedRows: false`** applies an internal 500-row cap when virtualization is disabled.
- **Streaming datasets**: fixed `rowHeight`, `enableHover=false`, `allowKeyboard=false`, `selectionSettings.enabled=false`, `enableCache=false` for the lowest re-render cost.
- **Detecting deployment gaps**: a grid with `height` set but no virtualization tuning will render all rows. For 100+ rows, this will trigger performance complaints — wire the perf settings by default.
- **High-risk change**: switching between `Virtual` and `Infinite` scroll modes silently disables features (aggregation/grouping). Confirm the user's intent before changing.