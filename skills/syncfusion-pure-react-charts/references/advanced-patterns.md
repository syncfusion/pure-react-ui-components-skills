# Advanced Patterns Reference

Use these patterns for production React Charts that require multiple axes, live data, dynamic series, application state, data transformation, resilience, testing, and coordinated interactions. Verify each chart-specific API against `@syncfusion/react-charts`; do not copy EJ2 component names, event names, or imperative methods into Pure React code.

## Table of contents

1. [Multiple axes](#multiple-axes)
2. [Real-time streaming](#real-time-streaming)
3. [Dynamic series](#dynamic-series)
4. [Performance](#performance)
5. [Reusable chart components](#reusable-chart-components)
6. [State management](#state-management)
7. [Data transformations](#data-transformations)
8. [Error handling](#error-handling)
9. [Testing](#testing)
10. [Interactivity](#interactivity)
11. [Production checklist](#production-checklist)

## Multiple axes

Define additional axes with `ChartAxes` and `ChartAxis`. Give every additional axis a unique `name`, then map the intended series with the matching `xAxisName` or `yAxisName`.

Do not use invented components such as `ChartSecondaryXAxis` or `ChartSecondaryYAxis`.

### Secondary Y-axis

```tsx
import {
  Chart,
  ChartAxes,
  ChartAxis,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
  ChartTitle,
} from "@syncfusion/react-charts";

const data = [
  { month: "Jan", revenue: 5000, growth: 45 },
  { month: "Feb", revenue: 6200, growth: 52 },
  { month: "Mar", revenue: 5800, growth: 48 },
];

export default function MultiAxisChart() {
  return (
    <Chart>
      <ChartTitle text="Revenue and growth" />

      <ChartPrimaryXAxis valueType="Category">
        <ChartAxisTitle text="Month" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis
        valueType="Double"
        minimum={0}
        maximum={10000}
        interval={2000}
      >
        <ChartAxisTitle text="Revenue" />
        <ChartAxisLabel format="${value}" />
      </ChartPrimaryYAxis>

      <ChartAxes>
        <ChartAxis
          name="growthAxis"
          valueType="Double"
          opposedPosition={true}
          minimum={0}
          maximum={100}
          interval={20}
        >
          <ChartAxisTitle text="Growth (%)" />
          <ChartAxisLabel format="{value}%" />
        </ChartAxis>
      </ChartAxes>

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="revenue"
          type="Column"
          name="Revenue"
        />
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="growth"
          type="Line"
          name="Growth"
          yAxisName="growthAxis"
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

A series uses the primary axis when no matching axis name is supplied. Do not map the primary series to an invented name such as `PrimaryAxis`.

### Multiple additional axes

```tsx
<ChartAxes>
  <ChartAxis name="temperatureAxis" opposedPosition={true}>
    <ChartAxisTitle text="Temperature (°C)" />
  </ChartAxis>

  <ChartAxis name="pressureAxis" opposedPosition={true}>
    <ChartAxisTitle text="Pressure (kPa)" />
  </ChartAxis>
</ChartAxes>

<ChartSeriesCollection>
  <ChartSeries yField="sales" name="Sales" />
  <ChartSeries
    yField="temperature"
    name="Temperature"
    yAxisName="temperatureAxis"
  />
  <ChartSeries
    yField="pressure"
    name="Pressure"
    yAxisName="pressureAxis"
  />
</ChartSeriesCollection>
```

Use multiple axes only when units or scales cannot be compared honestly on one axis. Too many axes can make the chart difficult to interpret.

### Additional X-axis

```tsx
<ChartAxes>
  <ChartAxis
    name="dateAxis"
    valueType="DateTime"
    opposedPosition={true}
  >
    <ChartAxisTitle text="Date" />
  </ChartAxis>
</ChartAxes>

<ChartSeries
  dataSource={dateData}
  xField="date"
  yField="value"
  xAxisName="dateAxis"
  type="Line"
/>
```

The additional axis `name` and series mapping must match exactly.

## Real-time streaming

Use React state for chart data, keep history bounded, validate incoming messages, and clean up connections when the component unmounts.

### Typed WebSocket stream with a bounded buffer

```tsx
import { useEffect, useState } from "react";
import {
  Chart,
  ChartPrimaryXAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

interface Reading {
  time: Date;
  value: number;
}

const MAX_POINTS = 100;

function isReading(value: unknown): value is { time: string; value: number } {
  if (typeof value !== "object" || value === null) {
    return false;
  }

  const candidate = value as Record<string, unknown>;
  return (
    typeof candidate.time === "string" &&
    typeof candidate.value === "number" &&
    Number.isFinite(candidate.value)
  );
}

export default function RealtimeChart() {
  const [data, setData] = useState<Reading[]>([]);
  const [status, setStatus] = useState("Connecting");

  useEffect(() => {
    const socket = new WebSocket("wss://stream.example.com/data");

    socket.onopen = () => setStatus("Connected");

    socket.onmessage = (event) => {
      try {
        const parsed: unknown = JSON.parse(event.data);

        if (!isReading(parsed)) {
          return;
        }

        const nextPoint: Reading = {
          time: new Date(parsed.time),
          value: parsed.value,
        };

        setData((current) =>
          [...current, nextPoint].slice(-MAX_POINTS),
        );
      } catch (error) {
        console.error("Unable to parse chart reading", error);
      }
    };

    socket.onerror = () => setStatus("Connection error");
    socket.onclose = () => setStatus("Disconnected");

    return () => socket.close();
  }, []);

  return (
    <section>
      <p aria-live="polite">Stream status: {status}</p>

      <Chart>
        <ChartPrimaryXAxis valueType="DateTime" />
        <ChartSeriesCollection>
          <ChartSeries
            dataSource={data}
            xField="time"
            yField="value"
            type="Line"
            name="Reading"
            animation={{ enable: false, duration: 0, delay: 0 }}
          />
        </ChartSeriesCollection>
      </Chart>
    </section>
  );
}
```

Use `wss://` for production HTTPS pages. Implement reconnection only when the application requirements define retry limits, backoff, and user messaging.

### Batched updates

For high-frequency streams, collect readings in a ref and update React state at a controlled interval.

```tsx
const bufferRef = useRef<Reading[]>([]);

useEffect(() => {
  const flushId = window.setInterval(() => {
    if (bufferRef.current.length === 0) {
      return;
    }

    const batch = bufferRef.current;
    bufferRef.current = [];

    setData((current) =>
      [...current, ...batch].slice(-MAX_POINTS),
    );
  }, 250);

  return () => window.clearInterval(flushId);
}, []);
```

Batching is generally more appropriate than debouncing for a continuous stream because debouncing can postpone updates indefinitely while messages continue arriving.

### Downsampling and aggregation

Do not label simple index filtering as statistical aggregation. Choose a method that preserves the information users need.

```tsx
interface Bucket {
  timestamp: number;
  sum: number;
  count: number;
}

function averageByInterval(
  points: Reading[],
  intervalMs: number,
): Reading[] {
  const buckets = new Map<number, Bucket>();

  for (const point of points) {
    const timestamp = point.time.getTime();
    const key = Math.floor(timestamp / intervalMs) * intervalMs;
    const bucket = buckets.get(key) ?? {
      timestamp: key,
      sum: 0,
      count: 0,
    };

    bucket.sum += point.value;
    bucket.count += 1;
    buckets.set(key, bucket);
  }

  return [...buckets.values()]
    .sort((a, b) => a.timestamp - b.timestamp)
    .map((bucket) => ({
      time: new Date(bucket.timestamp),
      value: bucket.sum / bucket.count,
    }));
}
```

For peaks, use min/max preservation rather than averages. For financial values, use the domain's required OHLC or volume aggregation.

## Dynamic series

Render series from stable application state. Use stable IDs as React keys and update arrays immutably.

```tsx
import { useState } from "react";

interface Point {
  x: number;
  y: number;
}

interface SeriesModel {
  id: string;
  name: string;
  data: Point[];
  visible: boolean;
}

function createSeries(index: number): SeriesModel {
  return {
    id: crypto.randomUUID(),
    name: `Series ${index}`,
    visible: true,
    data: Array.from({ length: 12 }, (_, pointIndex) => ({
      x: pointIndex + 1,
      y: Math.round(Math.random() * 100),
    })),
  };
}

export default function DynamicSeriesChart() {
  const [series, setSeries] = useState<SeriesModel[]>([
    createSeries(1),
  ]);

  const addSeries = () => {
    setSeries((current) => [
      ...current,
      createSeries(current.length + 1),
    ]);
  };

  const removeSeries = (id: string) => {
    setSeries((current) =>
      current.filter((item) => item.id !== id),
    );
  };

  const toggleSeries = (id: string) => {
    setSeries((current) =>
      current.map((item) =>
        item.id === id
          ? { ...item, visible: !item.visible }
          : item,
      ),
    );
  };

  return (
    <section>
      <button type="button" onClick={addSeries}>
        Add series
      </button>

      <Chart>
        <ChartSeriesCollection>
          {series
            .filter((item) => item.visible)
            .map((item) => (
              <ChartSeries
                key={item.id}
                dataSource={item.data}
                xField="x"
                yField="y"
                name={item.name}
                type="Line"
              />
            ))}
        </ChartSeriesCollection>
      </Chart>

      {series.map((item) => (
        <div key={item.id}>
          <button
            type="button"
            onClick={() => toggleSeries(item.id)}
          >
            {item.visible ? "Hide" : "Show"} {item.name}
          </button>
          <button
            type="button"
            onClick={() => removeSeries(item.id)}
          >
            Remove {item.name}
          </button>
        </div>
      ))}
    </section>
  );
}
```

### Replace one series data source

Do not mutate the nested series object.

```tsx
const replaceSeriesData = (
  id: string,
  nextData: Point[],
): void => {
  setSeries((current) =>
    current.map((item) =>
      item.id === id
        ? { ...item, data: nextData }
        : item,
    ),
  );
};
```

Avoid array-position identity when series can be reordered or removed.

## Performance

Measure before optimizing. Point count, series type, labels, markers, animation, interaction, device performance, and update frequency all affect rendering cost.

### Memoize transformations

```tsx
const chartData = useMemo(
  () => transformData(rawData),
  [rawData],
);
```

`useMemo` avoids repeating a transformation when its dependencies are unchanged. It does not make an expensive transformation cheap on its first execution.

### Point color callback

`pointRender` belongs to the root `Chart`, receives `PointRenderProps`, and returns the point color.

```tsx
import type { PointRenderProps } from "@syncfusion/react-charts";

const handlePointRender = useCallback(
  (args: PointRenderProps): string => {
    return Number(args.yValue) > 100
      ? "#C62828"
      : args.color;
  },
  [],
);

<Chart pointRender={handlePointRender}>
  <ChartSeriesCollection>
    <ChartSeries
      dataSource={chartData}
      xField="x"
      yField="y"
      type="Column"
    />
  </ChartSeriesCollection>
</Chart>
```

Do not attach `onPointRender` to `ChartSeries`, access `args.data`, or mutate `args.fill`.

### Animation strategy

```tsx
<ChartSeries
  dataSource={data}
  xField="x"
  yField="y"
  type="Line"
  animation={{
    enable: shouldAnimate,
    duration: 600,
    delay: 0,
  }}
/>
```

Do not use a universal cutoff such as exactly 1,000 points. Decide from profiling on target devices and consider user reduced-motion preferences.

### Reduce visual density

For dense charts, consider:

- aggregating or downsampling data
- showing only the visible time window
- disabling unnecessary markers
- reducing data labels
- disabling animation for rapid updates
- limiting simultaneously visible series
- moving detail into tooltips or drill-down views

Do not call filtered or aggregated rendering “virtualization” unless the chart actually virtualizes rendering.

### Code splitting

Code splitting is framework-specific. For Next.js client-only loading:

```tsx
import dynamic from "next/dynamic";

const SalesChart = dynamic(
  () => import("./SalesChart"),
  { ssr: false },
);
```

Use this only when server rendering is incompatible or deferred loading materially improves the application.

## Reusable chart components

Create a domain wrapper rather than passing an arbitrary chart component through a loosely typed higher-order component.

```tsx
import type { ReactNode } from "react";

interface ChartShellProps {
  title: string;
  children: ReactNode;
  theme?: "light" | "dark";
}

export function ChartShell({
  title,
  children,
  theme = "light",
}: ChartShellProps) {
  const dark = theme === "dark";

  return (
    <Chart
      background={dark ? "#1E1E1E" : "#FFFFFF"}
      palettes={
        dark
          ? ["#8AB4F8", "#81C995", "#FDD663"]
          : ["#005A9C", "#107C10", "#A4262C"]
      }
      accessibility={{
        ariaLabel: title,
        role: "img",
        focusable: true,
        tabIndex: 0,
      }}
    >
      <ChartTitle
        text={title}
        color={dark ? "#FFFFFF" : "#242424"}
      />
      {children}
    </Chart>
  );
}
```

Usage:

```tsx
<ChartShell title="Monthly sales" theme="dark">
  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="month"
      yField="sales"
      type="Column"
    />
  </ChartSeriesCollection>
</ChartShell>
```

Keep `ChartSeriesCollection` ownership clear. Do not wrap arbitrary children in a second collection when callers may already provide one.

## State management

The chart should receive normalized, render-ready data from application state. Keep fetching, caching, and domain transformations outside the chart tree when possible.

### Redux selector pattern

```tsx
const chartData = useSelector(selectVisibleSalesPoints);
const filters = useSelector(selectSalesFilters);
const dispatch = useDispatch<AppDispatch>();

useEffect(() => {
  void dispatch(fetchChartData(filters));
}, [dispatch, filters]);

<ChartSeries
  dataSource={chartData}
  xField="month"
  yField="sales"
  type="Column"
/>
```

Memoize derived selectors when they perform non-trivial transformations.

### Typed Context pattern

```tsx
import {
  createContext,
  useContext,
  useMemo,
  useState,
} from "react";
import type { ReactNode } from "react";

type ChartTheme = "light" | "dark";

interface ChartThemeContextValue {
  theme: ChartTheme;
  setTheme: (theme: ChartTheme) => void;
}

const ChartThemeContext =
  createContext<ChartThemeContextValue | undefined>(undefined);

export function ChartThemeProvider({
  children,
}: {
  children: ReactNode;
}) {
  const [theme, setTheme] = useState<ChartTheme>("light");

  const value = useMemo(
    () => ({ theme, setTheme }),
    [theme],
  );

  return (
    <ChartThemeContext.Provider value={value}>
      {children}
    </ChartThemeContext.Provider>
  );
}

export function useChartTheme(): ChartThemeContextValue {
  const context = useContext(ChartThemeContext);

  if (!context) {
    throw new Error(
      "useChartTheme must be used inside ChartThemeProvider",
    );
  }

  return context;
}
```

Do not call `useContext("light")`. `useContext` requires a context object, while the provider owns state through `useState`.

## Data transformations

Keep transformations pure, typed, and testable. Normalize data before binding it to `ChartSeries`.

### Group and sum

```tsx
interface RawItem {
  date: string;
  category: string;
  value: number;
}

interface DailyTotal {
  date: string;
  total: number;
  count: number;
}

function aggregateByDate(items: RawItem[]): DailyTotal[] {
  const totals = new Map<string, DailyTotal>();

  for (const item of items) {
    const current = totals.get(item.date) ?? {
      date: item.date,
      total: 0,
      count: 0,
    };

    current.total += item.value;
    current.count += 1;
    totals.set(item.date, current);
  }

  return [...totals.values()].sort((a, b) =>
    a.date.localeCompare(b.date),
  );
}
```

### Pivot series fields

```tsx
interface LongPoint {
  x: string;
  series: string;
  y: number;
}

type PivotPoint = {
  x: string;
} & Record<string, number | string>;

function pivotData(items: LongPoint[]): PivotPoint[] {
  const rows = new Map<string, PivotPoint>();

  for (const item of items) {
    const row = rows.get(item.x) ?? { x: item.x };
    row[item.series] = item.y;
    rows.set(item.x, row);
  }

  return [...rows.values()];
}
```

Bind every generated series with the shared `xField`.

```tsx
const pivoted = pivotData(raw);

<ChartSeriesCollection>
  <ChartSeries
    dataSource={pivoted}
    xField="x"
    yField="SeriesA"
    type="Line"
  />
  <ChartSeries
    dataSource={pivoted}
    xField="x"
    yField="SeriesB"
    type="Line"
  />
</ChartSeriesCollection>
```

### Time-series resampling

Use epoch timestamps for bucket calculations and convert to `Date` for a DateTime axis.

```tsx
function resampleAverage(
  points: Array<{ timestamp: number; value: number }>,
  intervalMs: number,
) {
  const grouped = new Map<number, number[]>();

  for (const point of points) {
    const bucket =
      Math.floor(point.timestamp / intervalMs) * intervalMs;
    const values = grouped.get(bucket) ?? [];
    values.push(point.value);
    grouped.set(bucket, values);
  }

  return [...grouped.entries()]
    .sort(([a], [b]) => a - b)
    .map(([timestamp, values]) => ({
      timestamp: new Date(timestamp),
      average:
        values.reduce((sum, value) => sum + value, 0) /
        values.length,
    }));
}
```

Validate `intervalMs > 0` and decide how empty buckets should be represented.

## Error handling

### Error boundary

```tsx
import React from "react";
import type { ErrorInfo, ReactNode } from "react";

interface ErrorBoundaryProps {
  children: ReactNode;
}

interface ErrorBoundaryState {
  error: Error | null;
}

class ChartErrorBoundary extends React.Component<
  ErrorBoundaryProps,
  ErrorBoundaryState
> {
  state: ErrorBoundaryState = { error: null };

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { error };
  }

  componentDidCatch(error: Error, info: ErrorInfo): void {
    console.error("Chart render failed", error, info);
  }

  render() {
    if (this.state.error) {
      return (
        <p role="alert">
          The chart could not be displayed. Try again later.
        </p>
      );
    }

    return this.props.children;
  }
}
```

Do not expose raw exception messages to end users. Log diagnostic details through the application's approved telemetry service.

### Fetch with cancellation and fallback

```tsx
useEffect(() => {
  const controller = new AbortController();

  async function loadData() {
    try {
      setStatus("loading");
      const response = await fetch("/api/chart-data", {
        signal: controller.signal,
      });

      if (!response.ok) {
        throw new Error(`Request failed: ${response.status}`);
      }

      const result: unknown = await response.json();
      const normalized = normalizeChartData(result);
      setData(normalized);
      setStatus("ready");
    } catch (error) {
      if (error instanceof DOMException && error.name === "AbortError") {
        return;
      }

      setData(getCachedData());
      setStatus("fallback");
    }
  }

  void loadData();
  return () => controller.abort();
}, []);
```

Tell users when cached or partial data is displayed and include the data timestamp when available.

## Testing

Test transformations separately from chart rendering. For integration tests, assert accessible output and application behavior rather than private SVG implementation details.

### Transformation unit test

```tsx
test("aggregates values by date", () => {
  const result = aggregateByDate([
    { date: "2026-01-01", category: "A", value: 100 },
    { date: "2026-01-01", category: "B", value: 50 },
  ]);

  expect(result).toEqual([
    { date: "2026-01-01", total: 150, count: 2 },
  ]);
});
```

### Data update integration test

```tsx
import { render, screen } from "@testing-library/react";

const { rerender } = render(
  <SalesChart data={initialData} />,
);

rerender(<SalesChart data={updatedData} />);

expect(
  screen.getByText("Updated through April 2026"),
).toBeInTheDocument();
```

Always keep each `ChartSeries` inside `ChartSeriesCollection` in production and test samples.

### Accessibility test

```tsx
const { container } = render(
  <Chart
    accessibility={{
      ariaLabel: "Monthly sales chart",
      role: "img",
      focusable: true,
      tabIndex: 0,
    }}
  >
    <ChartSeriesCollection>
      <ChartSeries
        dataSource={data}
        xField="month"
        yField="sales"
        type="Column"
      />
    </ChartSeriesCollection>
  </Chart>,
);

const results = await axe(container);
expect(results).toHaveNoViolations();
```

Automated checks supplement, but do not replace, keyboard and screen-reader testing.

## Interactivity

### Point drill-down

Use the documented `onPointClick` event with `PointClickEvent`.

```tsx
import type { PointClickEvent } from "@syncfusion/react-charts";

const handlePointClick = (args: PointClickEvent): void => {
  const selected = data[args.pointIndex];
  setSelectedCategory(selected?.category ?? null);
};

<Chart onPointClick={handlePointClick}>
  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="category"
      yField="value"
      type="Column"
    />
  </ChartSeriesCollection>
</Chart>
```

Do not use `onChartClick` for point drill-down. Use bounds checks when event indexes access application arrays.

### Legend interaction

The built-in legend already toggles series visibility when `toggleVisibility={true}`. Use `onLegendClick` only when custom application behavior is required.

```tsx
import type { LegendClickEvent } from "@syncfusion/react-charts";

const handleLegendClick = (args: LegendClickEvent): void => {
  trackLegendInteraction(args.seriesName);
};

<Chart onLegendClick={handleLegendClick}>
  <ChartLegend
    visible={true}
    toggleVisibility={true}
  />
</Chart>
```

`LegendClickEvent` exposes `seriesName`, not `args.series.name`. Set `args.cancel = true` only when intentionally replacing the default legend action.

### Synchronized charts

The documented `onMouseMove` event provides chart target and pointer coordinates, not a guaranteed `pointIndex`. Prefer `onPointClick` for exact point synchronization or synchronize controlled axis zoom ranges through verified axis properties.

```tsx
const handleSourcePointClick = (
  args: PointClickEvent,
): void => {
  setSelectedIndexes([
    {
      seriesIndex: args.seriesIndex,
      pointIndex: args.pointIndex,
    },
  ]);
};

<Chart onPointClick={handleSourcePointClick}>
  {/* source chart */}
</Chart>

<Chart>
  <ChartSelection
    mode="Point"
    selectedDataIndexes={selectedIndexes}
  />
  {/* target chart */}
</Chart>
```

`selectedDataIndexes` requires objects with both `seriesIndex` and `pointIndex`, not a number array.

### Dynamic point emphasis

For data-driven point color, map a color field when possible.

```tsx
const displayData = data.map((point, pointIndex) => ({
  ...point,
  color: highlightedIndexes.has(pointIndex)
    ? "#F9A825"
    : "#1976D2",
}));

<ChartSeries
  dataSource={displayData}
  xField="x"
  yField="y"
  colorField="color"
  type="Column"
/>
```

Or use the verified root `pointRender` callback when the color rule depends on render arguments.

### Export and printing

Do not assume `Chart` exposes `ref.current.export()` or other imperative export methods when those methods are absent from the current Pure React API. Use only package APIs documented for the installed version, or implement an application-level export workflow that is explicitly tested.

## Production checklist

Before returning an advanced chart implementation:

1. Use only components exported by `@syncfusion/react-charts`.
2. Place every `ChartSeries` inside `ChartSeriesCollection`.
3. Define additional axes with `ChartAxes` and uniquely named `ChartAxis` components.
4. Match every series axis name exactly.
5. Use `ChartAxisTitle` and `ChartAxisLabel` as axis children.
6. Keep streamed history bounded.
7. Validate external data before binding it.
8. Clean up WebSockets, timers, subscriptions, and fetches.
9. Batch continuous updates when appropriate.
10. Choose a domain-correct aggregation method.
11. Update arrays and nested objects immutably.
12. Use stable IDs as React keys.
13. Profile before applying data thresholds.
14. Do not call downsampling virtualization.
15. Use root `pointRender` and return a color.
16. Use `onPointClick` for point drill-down.
17. Use `onLegendClick` with `LegendClickEvent`.
18. Pass `ChartIndexesProps[]` to `selectedDataIndexes`.
19. Do not invent imperative chart export or zoom methods.
20. Provide loading, empty, fallback, and failure states.
21. Add meaningful chart and series accessibility descriptions.
22. Test keyboard access and screen-reader output.
23. Unit-test transformations independently.
24. Avoid assertions against private chart DOM details.
25. Do not mix EJ2 APIs with Pure React APIs.
26. Ensure every imported symbol is used.
27. Emit valid, unescaped TSX.
