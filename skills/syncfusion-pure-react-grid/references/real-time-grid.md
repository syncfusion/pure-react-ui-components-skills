---
name: real-time-grid
description: Real-time and streaming patterns for the Syncfusion React Data Grid — fixed rowHeight + setCellValue + currentViewData for fast partial re-renders, interval-driven updates, and the order-status / streaming-trade demos. Load when wiring the grid to a WebSocket, polling endpoint, scheduler-driven refresh, or high-frequency tick updates.
---

# Real-time grid

The grid is optimized for **streaming-like** updates: change a single cell, the grid re-renders only that cell. Patterns:

- **Interval-driven** updates (`setInterval`/`setTimeout` + state).
- **`setCellValue`** for partial re-renders.
- **`currentViewData`** for read-back of what's currently rendered (also for selecting N rows to mutate).
- **Fixed `rowHeight`** + `enableHover={false}` + `allowKeyboard={false}` + `selectionSettings.enabled={false}` for the lowest re-render cost.

## Setup for streaming

```tsx
import { Grid, GridRef, SortDirection } from '@syncfusion/react-grid';

const gridRef = useRef<GridRef>(null);

// Disable expensive visuals
const [modules] = useState({ /* whatever features */ });

<Grid
  ref={gridRef}
  dataSource={data}
  rowHeight={40}                  // FIXED — required for stable virtualization
  height={400}
  enableHover={false}
  allowKeyboard={false}
  selectionSettings={{ enabled: false }}
  onDataLoad={onDataLoad}
  modules={modules}
  enableDevMode={false}
/>
```

> `rowHeight` must be fixed. Dynamic rowheight (`getRowHeight` or `autoHeight`) degrades scroll perf measurably in a streaming scenario.

## Interval updates

```tsx
useEffect(() => {
  const t = setInterval(() => updateSomeCells(), feedDelay);
  return () => clearInterval(t);
}, [feedDelay]);

const updateSomeCells = () => {
  const rows = (gridRef.current?.currentViewData ?? []) as TradeRow[];
  if (!rows.length) return;
  const sample = Math.min(rows.length, Math.floor(Math.random() * 50) + 50);
  for (let i = 0; i < sample; i++) {
    const r = rows[Math.floor(Math.random() * rows.length)];
    const newPrice = +(r.price * (1 + (Math.random() - 0.5) * 0.01)).toFixed(2);
    gridRef.current?.setCellValue?.(r.symbol, 'price', newPrice);
  }
};
```

`setCellValue(pk, field, value)` re-renders only the affected cell. Pair with `useMemo` for the `<Grid>` JSX to avoid cascading re-renders.

## `currentViewData`

`gridRef.current?.currentViewData: T[]` returns the rows currently rendered. Use this to:

- Pick a sample of rows to mutate.
- Look up the row index before calling `updateRecord` (if you don't have the primary key).
- Snapshot for downstream UI (e.g., a "Top Movers" sidebar).

## Order-status tracker pattern

```tsx
const [orders, setOrders] = useState<Order[]>(initialOrders);
useEffect(() => {
  const t = setInterval(() => setOrders(prev =>
    prev.map(o => /* mutate randomly based on o.status */ o)
  ), 5000);
  return () => clearInterval(t);
}, []);

<Grid
  dataSource={orders}
  ref={gridElementRef}
  sortSettings={{ enabled: true, columns: [{ field: 'Status', direction: SortDirection.Descending }] }}
  pageSettings={{ enabled: true, pageSize: 6 }}
  enableHover={false}
  selectionSettings={{ enabled: false }}
  allowKeyboard={false}
  onClick={(e) => e.preventDefault()}
/>
```

Common pattern: pretty-up the cell with `template` (badges, theme colors, icons). Memoize the template with `useCallback` keyed on the icons / classes. Memoize the entire `<Grid>` with `useMemo` keyed on `[orders, templateRefs]`.

## Streaming trade view pattern

```tsx
<Grid
  ref={gridRef}
  dataSource={tradeTickerData}
  enableHover={false}
  rowHeight={40}
  height={400}
  onDataLoad={onDataLoad}
  selectionSettings={{ enabled: false }}
  allowKeyboard={false}
>
  <Columns>
    <Column field="symbol" headerText="Symbol" width="120" />
    <Column field="price" headerText="Price" format="N2" textAlign={TextAlign.Right} />
    <Column field="change" headerText="Change" template={changeTemplate} />
  </Columns>
</Grid>
```

Cell update pattern with `setCellValue`:

```ts
gridRef.current?.setCellValue?.(symbol, 'price', newPrice);
gridRef.current?.setCellValue?.(symbol, 'change', newChange);
gridRef.current?.setCellValue?.(symbol, 'change_percent', newPercent);
```

Trigger the loop from UI:

```tsx
const startUpdate = () => {
  stopUpdate();
  timerRef.current = setInterval(updateSomeCells, feedDelay);
};
const stopUpdate = () => {
  if (timerRef.current) clearInterval(timerRef.current);
  timerRef.current = null;
};
```

Bind via `NumericTextBox` (`@syncfusion/react-inputs`) for `feedDelay`; a button toggles start/stop.

## Auto-start on initial load

```tsx
const initial = useRef(true);

const onDataLoad = () => {
  if (gridRef.current && initial.current) {
    document.getElementById('update1')?.click();   // button click simulator
    initial.current = false;
  }
};
```

## Accessibility considerations for streaming grids

- `enableHover=false`, `selectionSettings.enabled=false`, `allowKeyboard=false` all **disable** their accessibility affordances. Apply only when the dataset is purely informational (e.g., a monitor in a control room).
- Keep `aria-busy` usage appropriate: when the grid is updating quickly, screen readers may need explicit notifications. The grid exposes `aria-busy` automatically during async loads.

## Constraints & guardrails

- **Fixed `rowHeight` is required** for stable virtualization. Don't use `getRowHeight` or `autoHeight` in streaming.
- **Disable expensive visuals** in the streaming grid: `enableHover`, `selection`, `allowKeyboard` should be off to keep DOM light.
- **Memoization**: `useCallback` for templates, `React.memo` for cell components, `useMemo` for the entire `<Grid>`.
- **Cell value updates** with `setCellValue` are best-effort — concurrent updates from a WebSocket may race; debounce or coalesce updates at the application layer if necessary.
- **Cleanup**: clear `setInterval` in the `useEffect` return. Lost timers leak memory and trigger updates after unmount, which is a *guardrail circuit breaker* case in the project's standards.
- **Integration: feature trade-off**: switching to streaming + fixed height disables grouping/aggregates (the grid can't recompute them per-tick). Plan the data shape in advance.
- **High-risk operations**: avoid `clearFilter([])` from a streaming loop — it removes all filters and can cause flicker. Apply filters deliberately, once.
- **Sensitive data timing**: never trust unsanitized streamed payloads — sanitize before `setCellValue` (XSS can live in any `string` field).