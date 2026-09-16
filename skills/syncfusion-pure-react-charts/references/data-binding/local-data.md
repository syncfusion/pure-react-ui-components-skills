# Local Data Binding Reference

Use local data binding when chart data already exists in memory as an array of objects, a derived array, or React state. Bind the array to `ChartSeries.dataSource` and map its fields with the properties required by the selected series type.

## Required component hierarchy

Place each `ChartSeries` inside `ChartSeriesCollection`.

```tsx
import {
  Chart,
  ChartPrimaryXAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { month: "Jan", sales: 35 },
  { month: "Feb", sales: 42 },
  { month: "Mar", sales: 38 },
];

<Chart>
  <ChartPrimaryXAxis valueType="Category" />

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="month"
      yField="sales"
      type="Line"
      name="Sales"
    />
  </ChartSeriesCollection>
</Chart>
```

The official local-data guidance binds an in-memory object array directly to the series `dataSource`. 

## Core binding contract

For a standard Cartesian series, configure:

- `dataSource`: the local array of objects
- `xField`: the property containing X values
- `yField`: the property containing Y values
- `type`: an exact supported series type

```tsx
const data = [
  { category: "A", value: 42 },
  { category: "B", value: 58 },
];

<ChartSeries
  dataSource={data}
  xField="category"
  yField="value"
  type="Column"
/>
```

Every mapped field name must exist in every applicable data object.

## Data shape consistency

Keep the object shape consistent across the array.

Correct:

```tsx
const data = [
  { month: "Jan", sales: 35 },
  { month: "Feb", sales: 42 },
  { month: "Mar", sales: 38 },
];
```

Incorrect:

```tsx
const data = [
  { month: "Jan", sales: 35 },
  { label: "Feb", value: 42 },
];
```

The second object does not contain the mapped `month` and `sales` fields.

Do not invent mappings to repair inconsistent input silently. Normalize the data explicitly before binding.

## Axis compatibility

Match the X-axis `valueType` to the mapped X values.

### Category values

```tsx
const data = [
  { product: "Laptop", sales: 48 },
  { product: "Mobile", sales: 62 },
];

<ChartPrimaryXAxis valueType="Category" />

<ChartSeries
  dataSource={data}
  xField="product"
  yField="sales"
  type="Column"
/>
```

### Numeric values

```tsx
const data = [
  { distance: 10, speed: 22 },
  { distance: 25, speed: 46 },
];

<ChartPrimaryXAxis valueType="Double" />

<ChartSeries
  dataSource={data}
  xField="distance"
  yField="speed"
  type="Scatter"
/>
```

### Date values

```tsx
const data = [
  { date: new Date(2026, 0, 1), sales: 35 },
  { date: new Date(2026, 1, 1), sales: 42 },
];

<ChartPrimaryXAxis valueType="DateTime" />

<ChartSeries
  dataSource={data}
  xField="date"
  yField="sales"
  type="Line"
/>
```

Prefer valid `Date` objects when elapsed-time spacing matters. Use `Category` when date-like labels should remain equally spaced.

## Numeric field validation

Fields used as Y values, financial values, ranges, bubble sizes, errors, and indicator inputs must contain finite numbers.

```tsx
const normalized = sourceData.map((item) => ({
  month: item.month,
  sales: Number(item.sales),
}));

const validData = normalized.filter((item) =>
  Number.isFinite(item.sales),
);
```

Do not bind numeric strings when the chart expects numeric values.

Incorrect:

```tsx
const data = [
  { month: "Jan", sales: "35" },
];
```

Correct:

```tsx
const data = [
  { month: "Jan", sales: 35 },
];
```

## Series-specific mappings

Do not assume every series uses only `xField` and `yField`.

### Standard Cartesian series

```tsx
<ChartSeries
  dataSource={data}
  xField="x"
  yField="y"
  type="Line"
/>
```

### Range series

Map both `low` and `high`.

```tsx
const data = [
  { day: "Mon", low: 18, high: 29 },
  { day: "Tue", low: 20, high: 31 },
];

<ChartSeries
  dataSource={data}
  xField="day"
  low="low"
  high="high"
  type="RangeArea"
/>
```

Do not use `yField` as one of the range boundaries.

### Financial series

```tsx
const data = [
  {
    date: new Date(2026, 0, 2),
    open: 101,
    high: 108,
    low: 98,
    close: 106,
  },
];

<ChartSeries
  dataSource={data}
  xField="date"
  open="open"
  high="high"
  low="low"
  close="close"
  type="Candle"
/>
```

Use `HiloOpenClose` for OHLC and `Hilo` for high-low series.

### Bubble series

```tsx
const data = [
  { x: 10, y: 20, size: 12 },
  { x: 15, y: 28, size: 20 },
];

<ChartSeries
  dataSource={data}
  xField="x"
  yField="y"
  size="size"
  type="Bubble"
/>
```

The mapped size field must contain finite non-negative numeric values.

### Histogram

```tsx
const data = [
  { value: 15 },
  { value: 18 },
  { value: 23 },
  { value: 31 },
];

<ChartSeries
  dataSource={data}
  yField="value"
  type="Histogram"
  histogramSettings={{
    binInterval: 10,
    showNormalDistribution: true,
  }}
/>
```

Use the histogram mapping and settings documented by the chart-type reference.

## One data array, multiple Y fields

Several series may use one shared local array when the objects contain all required fields.

```tsx
const data = [
  { month: "Jan", sales: 35, revenue: 42 },
  { month: "Feb", sales: 42, revenue: 48 },
  { month: "Mar", sales: 38, revenue: 45 },
];

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

This keeps X values aligned and avoids duplicate arrays when the records naturally belong together.

## Separate arrays for separate series

Use separate arrays when series have different observations or schemas.

```tsx
const actualData = [
  { month: "Jan", value: 35 },
  { month: "Feb", value: 42 },
];

const forecastData = [
  { month: "Mar", value: 46 },
  { month: "Apr", value: 51 },
];

<ChartSeriesCollection>
  <ChartSeries
    dataSource={actualData}
    xField="month"
    yField="value"
    type="Line"
    name="Actual"
  />
  <ChartSeries
    dataSource={forecastData}
    xField="month"
    yField="value"
    type="Line"
    name="Forecast"
  />
</ChartSeriesCollection>
```

Ensure each array independently satisfies its series mappings.

## Binding React state

Bind state directly to `dataSource`. The official local-data guidance supports React state so the chart reflects state updates. 

```tsx
import { useState } from "react";

const initialData = [
  { month: "Jan", sales: 35 },
  { month: "Feb", sales: 42 },
];

export default function StatefulChart() {
  const [data, setData] = useState(initialData);

  const addPoint = () => {
    setData((current) => [
      ...current,
      { month: "Mar", sales: 38 },
    ]);
  };

  return (
    <>
      <button type="button" onClick={addPoint}>
        Add March
      </button>

      <Chart>
        <ChartPrimaryXAxis valueType="Category" />
        <ChartSeriesCollection>
          <ChartSeries
            dataSource={data}
            xField="month"
            yField="sales"
            type="Line"
          />
        </ChartSeriesCollection>
      </Chart>
    </>
  );
}
```

Use immutable updates such as array spread, `map`, and `filter` so React receives a new array reference.

## Updating a point immutably

```tsx
setData((current) =>
  current.map((point) =>
    point.month === "Feb"
      ? { ...point, sales: 48 }
      : point,
  ),
);
```

Do not mutate existing state:

```tsx
// Avoid this.
data[1].sales = 48;
setData(data);
```

Mutation reuses the same array reference and makes update behavior harder to reason about.

## Removing a point immutably

```tsx
setData((current) =>
  current.filter((point) => point.month !== "Feb"),
);
```

## Preparing local data with useMemo

Use `useMemo` when deriving chart records from other in-memory values and the transformation is worth memoizing.

```tsx
import { useMemo } from "react";

const chartData = useMemo(
  () =>
    records.map((record) => ({
      month: record.month,
      sales: Number(record.sales),
    })),
  [records],
);
```

Do not use `useMemo` automatically for a small constant array declared outside the component.

## Preparing data with useEffect

The official local-data guidance also supports preparing or loading in-memory data after mount with `useEffect`. 

```tsx
import { useEffect, useState } from "react";

type SalesPoint = {
  month: string;
  sales: number;
};

export default function PreparedLocalDataChart() {
  const [data, setData] = useState<SalesPoint[]>([]);

  useEffect(() => {
    const prepared = rawData.map((item) => ({
      month: item.month,
      sales: Number(item.sales),
    }));

    setData(prepared);
  }, []);

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

Use `useEffect` only when preparation depends on lifecycle or another changing value. For static local data, bind the array directly.

## Empty local data

An empty array is a valid local data source.

```tsx
const data: SalesPoint[] = [];

<ChartSeries
  dataSource={data}
  xField="month"
  yField="sales"
  type="Line"
/>
```

Use the root chart's documented `noDataTemplate` when a custom empty-state presentation is requested.

```tsx
<Chart noDataTemplate="No sales data available">
  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="month"
      yField="sales"
      type="Line"
    />
  </ChartSeriesCollection>
</Chart>
```

Do not fabricate points merely to avoid an empty chart.

## Null and missing values

When a series may contain missing Y values, preserve the missing value and configure `emptyPointSettings` according to the selected series behavior.

```tsx
const data = [
  { month: "Jan", sales: 35 },
  { month: "Feb", sales: null },
  { month: "Mar", sales: 38 },
];

<ChartSeries
  dataSource={data}
  xField="month"
  yField="sales"
  type="Line"
  emptyPointSettings={{
    mode: "Gap",
    fill: "#BDBDBD",
  }}
/>
```

Use only documented empty-point modes for the selected chart family. Do not convert missing values to zero unless zero is the intended business value.

## TypeScript data types

Define a type for non-trivial local records.

```tsx
type SalesPoint = {
  month: string;
  sales: number;
};

const data: SalesPoint[] = [
  { month: "Jan", sales: 35 },
  { month: "Feb", sales: 42 },
];
```

For optional values:

```tsx
type SalesPoint = {
  month: string;
  sales: number | null;
};
```

Keep TypeScript types aligned with the actual array and chart mappings.

## Complete local-data example

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartLegend,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

type MonthlyResult = {
  month: string;
  sales: number;
  revenue: number;
};

const data: MonthlyResult[] = [
  { month: "Jan", sales: 35, revenue: 42 },
  { month: "Feb", sales: 42, revenue: 48 },
  { month: "Mar", sales: 38, revenue: 45 },
  { month: "Apr", sales: 51, revenue: 59 },
  { month: "May", sales: 47, revenue: 63 },
];

export default function LocalDataBindingChart() {
  return (
    <Chart noDataTemplate="No monthly results available">
      <ChartPrimaryXAxis valueType="Category">
        <ChartAxisTitle text="Month" />
        <ChartAxisLabel edgeLabelPlacement="Shift" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double" minimum={0}>
        <ChartAxisTitle text="Value" />
        <ChartAxisLabel format="{value}" />
      </ChartPrimaryYAxis>

      <ChartLegend
        visible={true}
        position="Bottom"
      />

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
          width={2}
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Common errors

### Missing field mapping

Incorrect:

```tsx
<ChartSeries dataSource={data} type="Line" />
```

when the chart requires explicit X and Y field mappings.

Correct:

```tsx
<ChartSeries
  dataSource={data}
  xField="month"
  yField="sales"
  type="Line"
/>
```

### Mapping a nonexistent field

Incorrect:

```tsx
const data = [{ month: "Jan", sales: 35 }];

<ChartSeries
  dataSource={data}
  xField="category"
  yField="value"
/>
```

Correct the mappings to `month` and `sales`.

### Axis and data mismatch

Incorrect:

```tsx
const data = [{ month: "Jan", sales: 35 }];
<ChartPrimaryXAxis valueType="DateTime" />
```

Use `Category` for the string month labels or change the data to valid dates.

### Mutating state

Incorrect:

```tsx
data.push(nextPoint);
setData(data);
```

Correct:

```tsx
setData((current) => [...current, nextPoint]);
```

### Using standard mappings for specialized series

Do not use only `yField` for range or financial series. Provide the required specialized mappings.

## Validation checklist

Before returning a local-data implementation:

1. Import all used chart components from `@syncfusion/react-charts`.
2. Place `ChartSeries` inside `ChartSeriesCollection`.
3. Bind the in-memory array through `dataSource`.
4. Ensure every configured mapping exists in the data objects.
5. Keep record shapes consistent or normalize them explicitly.
6. Match the axis `valueType` to the mapped X values.
7. Keep numeric fields as finite numbers.
8. Prefer valid `Date` objects for DateTime axes.
9. Supply `low` and `high` for range series.
10. Supply required OHLC fields for financial series.
11. Supply a valid size mapping for bubble series.
12. Use immutable state updates.
13. Do not add `useEffect` or `useMemo` when direct binding is sufficient.
14. Preserve null values and use documented empty-point behavior when appropriate.
15. Do not generate synthetic points unless the user explicitly requests synthetic data.
16. Ensure TypeScript types match the bound records.
17. Ensure every imported symbol is used.
18. Emit valid, unescaped TSX.
