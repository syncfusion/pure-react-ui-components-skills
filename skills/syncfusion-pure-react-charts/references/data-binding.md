# Data Binding Reference

Use `ChartSeries.dataSource` to bind local arrays, React state, or a Syncfusion `DataManager` to a Pure React Chart series. Always map the required fields explicitly and keep every `ChartSeries` inside `ChartSeriesCollection`.

## Table of contents

1. [Local data](#local-data)
2. [React state](#react-state)
3. [Remote data with Fetch](#remote-data-with-fetch)
4. [Remote data with DataManager](#remote-data-with-datamanager)
5. [Multiple series and sources](#multiple-series-and-sources)
6. [Transformations](#transformations)
7. [Validation and errors](#validation-and-errors)
8. [Performance and cleanup](#performance-and-cleanup)
9. [Validation checklist](#validation-checklist)

## Core binding rules

`ChartSeriesProps` accepts an array-like object or `DataManager` through `dataSource`. The mapped field names must exist in the bound records.

```tsx
<ChartSeries
  dataSource={data}
  xField="month"
  yField="sales"
  type="Column"
/>
```

Use the mapping required by the selected series type:

- Cartesian value series: `xField` and `yField`
- Bubble series: `xField`, `yField`, and `sizeField`
- Range series: `xField`, `low`, and `high`
- Financial series: `xField`, `open`, `high`, `low`, and `close` as required
- Point colors: `colorField`
- Custom tooltip content: `tooltipField`

Do not use `size="fieldName"` for bubble data. The current Pure React property is `sizeField`.

## Local data

### Array of objects

```tsx
import {
  Chart,
  ChartPrimaryXAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

interface SalesPoint {
  month: string;
  sales: number;
}

const data: SalesPoint[] = [
  { month: "Jan", sales: 100 },
  { month: "Feb", sales: 120 },
  { month: "Mar", sales: 110 },
  { month: "Apr", sales: 150 },
];

export default function LocalDataChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis valueType="Category" />

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="sales"
          type="Column"
          name="Sales"
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

A module-level constant is suitable for static data. Local data can still be updated later when it is stored in React state.

### Multiple mapped fields

```tsx
interface BubblePoint {
  year: number;
  value: number;
  population: number;
  color: string;
}

const data: BubblePoint[] = [
  { year: 2024, value: 35, population: 12, color: "#1976D2" },
  { year: 2025, value: 42, population: 18, color: "#388E3C" },
  { year: 2026, value: 48, population: 16, color: "#F57C00" },
];

<ChartSeries
  dataSource={data}
  xField="year"
  yField="value"
  sizeField="population"
  colorField="color"
  type="Bubble"
/>
```

The bubble size field must contain meaningful numeric values. A category string is not a valid size measure.

### Nested source objects

Do not assume dot-path field mapping is supported unless the installed Pure React package explicitly documents it. Normalize nested records into flat chart points before binding.

```tsx
interface ApiItem {
  month: string;
  metrics: {
    sales: number;
    revenue: number;
  };
}

const chartData = source.map((item: ApiItem) => ({
  month: item.month,
  sales: item.metrics.sales,
  revenue: item.metrics.revenue,
}));

<ChartSeries
  dataSource={chartData}
  xField="month"
  yField="sales"
  type="Column"
/>
```

Flattening the records also makes validation and TypeScript typing clearer.

### Derived data

Transform records before rendering rather than embedding calculations inside JSX.

```tsx
interface RawPoint {
  date: string;
  count: number;
}

const processedData = rawData.map((item: RawPoint) => ({
  date: new Date(item.date),
  count: item.count,
  percentage: (item.count / targetCount) * 100,
  trend: item.count >= targetCount ? "At or above target" : "Below target",
}));

<ChartSeries
  dataSource={processedData}
  xField="date"
  yField="percentage"
  tooltipField="trend"
  type="Line"
/>
```

Guard against a zero denominator before calculating percentages.

## React state

Pass state directly to `dataSource`. The chart reflects updates when React receives a new state value.

```tsx
import { useState } from "react";

interface Point {
  x: number;
  y: number;
}

export default function StateDataChart() {
  const [data, setData] = useState<Point[]>([
    { x: 1, y: 10 },
    { x: 2, y: 20 },
    { x: 3, y: 15 },
  ]);

  const addPoint = (): void => {
    setData((current) => [
      ...current,
      {
        x: current.length + 1,
        y: Math.round(Math.random() * 100),
      },
    ]);
  };

  return (
    <section>
      <button type="button" onClick={addPoint}>
        Add point
      </button>

      <Chart>
        <ChartSeriesCollection>
          <ChartSeries
            dataSource={data}
            xField="x"
            yField="y"
            type="Line"
          />
        </ChartSeriesCollection>
      </Chart>
    </section>
  );
}
```

Use functional updates when the next value depends on current state. Never mutate the existing array.

Incorrect:

```tsx
data.push(newPoint);
setData(data);
```

Correct:

```tsx
setData((current) => [...current, newPoint]);
```

### Update one point

```tsx
const updatePoint = (x: number, nextY: number): void => {
  setData((current) =>
    current.map((point) =>
      point.x === x ? { ...point, y: nextY } : point,
    ),
  );
};
```

### Bounded real-time window

```tsx
const MAX_POINTS = 100;

setData((current) =>
  [...current, newPoint].slice(-MAX_POINTS),
);
```

Choose the retained window from the product requirement and measured performance. Do not prescribe a universal 60, 100, or 1,000 point limit.

### Timer-driven updates

```tsx
useEffect(() => {
  const intervalId = window.setInterval(() => {
    const nextPoint = {
      timestamp: new Date(),
      value: Math.sin(Date.now() / 1000) * 100,
    };

    setData((current) =>
      [...current, nextPoint].slice(-MAX_POINTS),
    );
  }, 1000);

  return () => window.clearInterval(intervalId);
}, []);
```

Bind `Date` values to a `DateTime` axis instead of formatted time strings when the spacing should reflect real elapsed time.

### Dynamic series

Use stable IDs as React keys and update the series array immutably.

```tsx
interface SeriesModel {
  id: string;
  name: string;
  data: Point[];
}

const [series, setSeries] = useState<SeriesModel[]>(initialSeries);

const addSeries = (nextSeries: SeriesModel): void => {
  setSeries((current) => [...current, nextSeries]);
};

<ChartSeriesCollection>
  {series.map((item) => (
    <ChartSeries
      key={item.id}
      dataSource={item.data}
      xField="x"
      yField="y"
      type="Line"
      name={item.name}
    />
  ))}
</ChartSeriesCollection>
```

Do not use the array index as a key when series may be reordered or removed.

## Remote data with Fetch

Keep loading, success, empty, and error states explicit. Check the HTTP result and validate the response before binding it.

```tsx
import { useEffect, useState } from "react";

interface SalesPoint {
  month: string;
  sales: number;
}

type LoadState = "loading" | "ready" | "empty" | "error";

function isSalesPoint(value: unknown): value is SalesPoint {
  if (typeof value !== "object" || value === null) {
    return false;
  }

  const item = value as Record<string, unknown>;
  return (
    typeof item.month === "string" &&
    typeof item.sales === "number" &&
    Number.isFinite(item.sales)
  );
}

export default function RemoteDataChart() {
  const [data, setData] = useState<SalesPoint[]>([]);
  const [status, setStatus] = useState<LoadState>("loading");

  useEffect(() => {
    const controller = new AbortController();

    async function loadData(): Promise<void> {
      try {
        setStatus("loading");

        const response = await fetch("/api/sales-data", {
          signal: controller.signal,
        });

        if (!response.ok) {
          throw new Error(`Request failed with status ${response.status}`);
        }

        const result: unknown = await response.json();

        if (!Array.isArray(result) || !result.every(isSalesPoint)) {
          throw new Error("Unexpected sales data format");
        }

        setData(result);
        setStatus(result.length === 0 ? "empty" : "ready");
      } catch (error) {
        if (error instanceof DOMException && error.name === "AbortError") {
          return;
        }

        console.error("Unable to load chart data", error);
        setStatus("error");
      }
    }

    void loadData();
    return () => controller.abort();
  }, []);

  if (status === "loading") {
    return <p>Loading chart data...</p>;
  }

  if (status === "error") {
    return <p role="alert">Chart data could not be loaded.</p>;
  }

  if (status === "empty") {
    return <p>No chart data is available.</p>;
  }

  return (
    <Chart>
      <ChartPrimaryXAxis valueType="Category" />
      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="sales"
          type="Column"
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

Do not display raw server error details to end users.

### Fetch when a dependency changes

```tsx
useEffect(() => {
  const controller = new AbortController();

  async function loadCategory(): Promise<void> {
    const response = await fetch(
      `/api/data?category=${encodeURIComponent(category)}`,
      { signal: controller.signal },
    );

    if (!response.ok) {
      throw new Error(`Request failed with status ${response.status}`);
    }

    const result: unknown = await response.json();
    setData(normalizeData(result));
  }

  void loadCategory().catch((error) => {
    if (!(error instanceof DOMException && error.name === "AbortError")) {
      console.error("Unable to load category data", error);
    }
  });

  return () => controller.abort();
}, [category]);
```

Abort the previous request when the dependency changes so stale responses do not overwrite newer state.

### Polling without overlapping requests

Use recursive scheduling after each request completes instead of starting another request while the previous one may still be running.

```tsx
useEffect(() => {
  const controller = new AbortController();
  let timeoutId: number | undefined;
  let active = true;

  async function poll(): Promise<void> {
    try {
      const response = await fetch("/api/live-data", {
        signal: controller.signal,
      });

      if (!response.ok) {
        throw new Error(`Request failed with status ${response.status}`);
      }

      const result: unknown = await response.json();
      if (active) {
        setData(normalizeData(result));
      }
    } catch (error) {
      if (!(error instanceof DOMException && error.name === "AbortError")) {
        console.error("Polling failed", error);
      }
    } finally {
      if (active) {
        timeoutId = window.setTimeout(poll, 5000);
      }
    }
  }

  void poll();

  return () => {
    active = false;
    controller.abort();
    if (timeoutId !== undefined) {
      window.clearTimeout(timeoutId);
    }
  };
}, []);
```

### WebSocket updates

```tsx
useEffect(() => {
  const socket = new WebSocket("wss://stream.example.com/data");

  socket.onmessage = (event) => {
    try {
      const result: unknown = JSON.parse(event.data);
      const point = normalizePoint(result);

      setData((current) =>
        [...current, point].slice(-MAX_POINTS),
      );
    } catch (error) {
      console.error("Invalid stream message", error);
    }
  };

  return () => socket.close();
}, []);
```

Use `wss://` on production HTTPS pages. Define retry and backoff behavior from application requirements rather than reconnecting indefinitely.

## Remote data with DataManager

The official Pure React remote-data guide uses `DataManager` and adaptors from `@syncfusion/react-data`. `ChartSeries.dataSource` accepts a `DataManager` instance.

### Installation

```bash
npm install @syncfusion/react-data
```

### UrlAdaptor

```tsx
import {
  DataManager,
  Query,
  UrlAdaptor,
} from "@syncfusion/react-data";

const dataManager = new DataManager({
  url: "https://api.example.com/data",
  adaptor: new UrlAdaptor(),
});

const query = new Query()
  .where("sales", "greaterThan", 50)
  .sortByDesc("date")
  .take(100);

<ChartSeries
  dataSource={dataManager}
  query={query}
  xField="month"
  yField="sales"
  type="Column"
/>
```

Pass the `DataManager` to `dataSource` and the `Query` to the series `query` prop. Do not pass the Promise returned by `dataManager.executeQuery(query)` as `dataSource`.

Incorrect:

```tsx
<ChartSeries
  dataSource={dataManager.executeQuery(query)}
/>
```

### UrlAdaptor response shape

A `UrlAdaptor` endpoint commonly returns records and count metadata in this shape:

```json
{
  "result": [
    { "month": "Jan", "sales": 100 },
    { "month": "Feb", "sales": 120 }
  ],
  "count": 2
}
```

Match the endpoint response format to the selected adaptor.

### ODataV4Adaptor

```tsx
import {
  DataManager,
  ODataV4Adaptor,
  Query,
} from "@syncfusion/react-data";

const dataManager = new DataManager({
  url: "https://services.example.com/odata/Sales",
  adaptor: new ODataV4Adaptor(),
});

const query = new Query()
  .select(["Month", "Sales"])
  .sortBy("Month")
  .take(100);
```

Use `ODataV4Adaptor` only with an OData v4-compatible endpoint.

### WebApiAdaptor

```tsx
import {
  DataManager,
  Query,
  WebApiAdaptor,
} from "@syncfusion/react-data";

const dataManager = new DataManager({
  url: "https://api.example.com/values",
  adaptor: new WebApiAdaptor(),
});

const query = new Query().take(100);
```

Use `WebApiAdaptor` when the endpoint follows the response and query contract expected by that adaptor.

### Query operations

`Query` supports operations such as:

- `where` for filtering
- `sortBy` or `sortByDesc` for ordering
- `skip` and `take` for range selection
- `page` for paging
- `select` for field selection
- `search` for text search
- `requiresCount` for total-count retrieval

```tsx
const query = new Query()
  .where("sales", "greaterThan", 50)
  .sortByDesc("date")
  .skip(0)
  .take(100)
  .requiresCount();
```

Ensure filtering and sorting fields match the server schema.

## Multiple series and sources

### Shared source with different Y fields

```tsx
<ChartSeriesCollection>
  <ChartSeries
    dataSource={data}
    xField="month"
    yField="sales"
    type="Column"
    name="Sales"
  />
  <ChartSeries
    dataSource={data}
    xField="month"
    yField="revenue"
    type="Line"
    name="Revenue"
  />
</ChartSeriesCollection>
```

Use one normalized array when the series share the same X domain.

### Different remote sources

```tsx
useEffect(() => {
  const controller = new AbortController();

  async function loadAll(): Promise<void> {
    const [salesResponse, revenueResponse] = await Promise.all([
      fetch("/api/sales", { signal: controller.signal }),
      fetch("/api/revenue", { signal: controller.signal }),
    ]);

    if (!salesResponse.ok || !revenueResponse.ok) {
      throw new Error("One or more chart requests failed");
    }

    const [salesResult, revenueResult]: [unknown, unknown] =
      await Promise.all([
        salesResponse.json(),
        revenueResponse.json(),
      ]);

    setSalesData(normalizeSales(salesResult));
    setRevenueData(normalizeRevenue(revenueResult));
  }

  void loadAll().catch((error) => {
    if (!(error instanceof DOMException && error.name === "AbortError")) {
      console.error("Unable to load chart sources", error);
    }
  });

  return () => controller.abort();
}, []);
```

If the metrics require different scales, define an additional named axis with `ChartAxes` and `ChartAxis`. Do not use `ChartSecondaryYAxis`.

```tsx
<ChartAxes>
  <ChartAxis
    name="revenueAxis"
    valueType="Double"
    opposedPosition={true}
  >
    <ChartAxisTitle text="Revenue" />
  </ChartAxis>
</ChartAxes>

<ChartSeries
  dataSource={revenueData}
  xField="month"
  yField="revenue"
  yAxisName="revenueAxis"
  type="Line"
/>
```

### Conditional series

```tsx
<ChartSeriesCollection>
  <ChartSeries
    dataSource={salesData}
    xField="month"
    yField="sales"
    type="Column"
  />

  {showRevenue && (
    <ChartSeries
      dataSource={revenueData}
      xField="month"
      yField="revenue"
      type="Line"
    />
  )}
</ChartSeriesCollection>
```

Keep field mappings explicit on every conditional series.

## Transformations

### Aggregate by key

```tsx
interface RawItem {
  date: string;
  category: string;
  value: number;
}

interface DailyTotal {
  date: string;
  total: number;
}

function aggregateByDate(items: RawItem[]): DailyTotal[] {
  const totals = new Map<string, number>();

  for (const item of items) {
    totals.set(item.date, (totals.get(item.date) ?? 0) + item.value);
  }

  return [...totals.entries()]
    .sort(([first], [second]) => first.localeCompare(second))
    .map(([date, total]) => ({ date, total }));
}
```

Choose sum, average, minimum, maximum, or another calculation according to the domain. Do not call point sampling an aggregation.

### Percentage calculation

```tsx
function addPercentages(items: Array<{ name: string; value: number }>) {
  const total = items.reduce((sum, item) => sum + item.value, 0);

  if (total === 0) {
    return items.map((item) => ({ ...item, percentage: 0 }));
  }

  return items.map((item) => ({
    ...item,
    percentage: (item.value / total) * 100,
  }));
}
```

### Memoized transformation

```tsx
const chartData = useMemo(
  () => aggregateByDate(rawData),
  [rawData],
);
```

Memoize only when the transformation is non-trivial or referential stability has a practical purpose.

## Validation and errors

### Correct runtime validation

Use explicit parentheses so type checks are evaluated correctly.

```tsx
interface Point {
  x: string | number;
  y: number;
}

function isPoint(value: unknown): value is Point {
  if (typeof value !== "object" || value === null) {
    return false;
  }

  const point = value as Record<string, unknown>;
  const validX =
    typeof point.x === "string" ||
    typeof point.x === "number";

  return (
    validX &&
    typeof point.y === "number" &&
    Number.isFinite(point.y)
  );
}

function isPointArray(value: unknown): value is Point[] {
  return Array.isArray(value) && value.every(isPoint);
}
```

The original expression combining `&&` and `||` without parentheses can accept invalid records because `&&` has higher precedence than `||`.

### Empty and missing values

Use `emptyPointSettings` when null or undefined values intentionally represent missing observations.

```tsx
<ChartSeries
  dataSource={data}
  xField="x"
  yField="y"
  type="Line"
  emptyPointSettings={{
    mode: "Gap",
    fill: "#9E9E9E",
    border: {
      color: "#616161",
      width: 1,
    },
  }}
/>
```

Do not convert missing values to zero unless zero is the correct domain value.

### Field mismatch diagnosis

If no points render, verify:

1. `dataSource` is a non-empty supported source.
2. `xField` and `yField` exactly match record keys.
3. Numeric fields contain finite numbers.
4. DateTime values are valid `Date` objects or supported date values.
5. The axis `valueType` matches the X values.
6. Range, bubble, and financial series contain all required mappings.

## Performance and cleanup

- Keep live data windows bounded.
- Batch high-frequency stream updates rather than updating React state for every message.
- Abort obsolete fetches.
- Clear timers and close WebSockets during cleanup.
- Memoize expensive transformations, not every small array operation.
- Disable or shorten animation based on measured performance and reduced-motion requirements.
- Aggregate or downsample with a method appropriate to the data.
- Do not describe ordinary filtering as virtualization.

A generic debounce helper must not call React hooks internally outside a component or custom hook. Prefer a properly scoped custom hook, a ref-based buffer, or batching logic with explicit cleanup.

## Common errors

### Missing field mappings

Incorrect:

```tsx
<ChartSeries dataSource={data} type="Line" />
```

Correct:

```tsx
<ChartSeries
  dataSource={data}
  xField="x"
  yField="y"
  type="Line"
/>
```

### Wrong bubble field

Incorrect:

```tsx
<ChartSeries size="population" type="Bubble" />
```

Correct:

```tsx
<ChartSeries sizeField="population" type="Bubble" />
```

### Promise passed as data source

Incorrect:

```tsx
<ChartSeries
  dataSource={dataManager.executeQuery(query)}
/>
```

Correct:

```tsx
<ChartSeries
  dataSource={dataManager}
  query={query}
/>
```

### Invented secondary axis

Do not use `ChartSecondaryYAxis`. Use `ChartAxes`, a named `ChartAxis`, and matching `yAxisName`.

### Unsupported generic series syntax

Do not assume `ChartSeries<ChartDataPoint>` is supported JSX syntax. Type the data array and helper functions instead.

### Unescaped URLs from rich text

Use normal quoted URL strings in TSX. Do not paste HTML anchor markup into a JavaScript string.

## Validation checklist

Before returning a data-binding implementation:

1. Keep every `ChartSeries` inside `ChartSeriesCollection`.
2. Bind a supported local array or `DataManager` to `dataSource`.
3. Map `xField` and the required value fields explicitly.
4. Match each mapping to an actual record key.
5. Use `sizeField` for bubble sizes.
6. Use `colorField` only with a valid color field.
7. Flatten nested data unless dot-path mapping is currently documented.
8. Use immutable React state updates.
9. Use functional updates when next state depends on current state.
10. Use stable IDs for dynamic series keys.
11. Check `response.ok` for Fetch requests.
12. Validate unknown remote data before binding it.
13. Abort stale requests and clean up timers or sockets.
14. Use loading, empty, error, and fallback states.
15. Pass `DataManager` directly to `dataSource`.
16. Pass `Query` through the series `query` prop.
17. Match the adaptor to the service protocol and response shape.
18. Use named additional axes instead of secondary-axis components.
19. Guard calculations against invalid numbers and zero denominators.
20. Keep live data bounded and profile target devices.
21. Avoid arbitrary performance limits presented as universal rules.
22. Do not mix EJ2 data-binding patterns with Pure React components.
23. Ensure every imported symbol is used.
24. Emit valid, unescaped TSX.
