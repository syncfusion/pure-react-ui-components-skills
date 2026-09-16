# Numeric Axis Reference

Use a numeric axis when values are continuous numbers and distances on the axis must remain proportional to numeric differences.

## Component hierarchy

Set `valueType="Double"` on the numeric axis. Place the title, labels, grid lines, and tick lines inside the owning axis.

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

<Chart>
  <ChartPrimaryXAxis valueType="Category" />

  <ChartPrimaryYAxis valueType="Double">
    <ChartAxisTitle text="Sales" />
    <ChartAxisLabel format="{value}" />
  </ChartPrimaryYAxis>

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="category"
      yField="sales"
      type="Column"
    />
  </ChartSeriesCollection>
</Chart>
```

Use `Double` with exact casing. Do not use `Numeric`, `Number`, or `Linear` as the axis `valueType`.

## When to use a numeric axis

Use `Double` when:

- The mapped values are finite numbers.
- Equal numeric differences should have equal visual distances.
- The range may contain positive, zero, or negative values.
- The axis needs numeric minimum, maximum, interval, padding, or number formatting.

Use `Category` when values are identifiers or discrete labels rather than quantities. Use `DateTime` for real dates and timestamps. Use `Logarithmic` when positive values span several orders of magnitude and ratio-based spacing is required.

## Data requirements

The field plotted against a numeric axis must contain numbers.

```tsx
const data = [
  { category: "A", value: 12.5 },
  { category: "B", value: 18.2 },
  { category: "C", value: -4.7 },
];
```

Avoid numeric strings:

```tsx
const invalidData = [
  { category: "A", value: "12.5" },
];
```

Normalize external values before binding:

```tsx
const chartData = sourceData.map((item) => ({
  ...item,
  value: Number(item.value),
}));
```

Validate the conversion:

```tsx
const validData = chartData.filter((item) =>
  Number.isFinite(item.value),
);
```

Do not silently remove or replace invalid business data unless that behavior is acceptable for the request.

## Automatic range

When `minimum`, `maximum`, and `interval` are omitted, the chart calculates the numeric range from the bound data.

```tsx
<ChartPrimaryYAxis valueType="Double" />
```

Prefer automatic range for a basic chart. Set explicit range properties only when the user requests fixed boundaries, consistent scales across charts, or a specific analytical baseline.

## Explicit range

Use `minimum` and `maximum` to define the visible numeric range.

```tsx
<ChartPrimaryYAxis
  valueType="Double"
  minimum={0}
  maximum={100}
/>
```

Validation rules:

- Both values must be finite numbers.
- `minimum` must be less than `maximum`.
- The range should contain the values that must remain visible.
- A fixed minimum of zero should be used only when zero is analytically meaningful or requested.
- Negative minimum values are valid on a `Double` axis.

```tsx
<ChartPrimaryYAxis
  valueType="Double"
  minimum={-50}
  maximum={50}
/>
```

## Interval

Use `interval` to set the numeric distance between major labels and ticks.

```tsx
<ChartPrimaryYAxis
  valueType="Double"
  minimum={0}
  maximum={100}
  interval={20}
/>
```

This produces major positions at 0, 20, 40, 60, 80, and 100.

Use a positive interval. Choose an interval that keeps labels readable and does not suggest more precision than the data supports.

## Desired intervals

Use `desiredIntervals` when an approximate interval count is preferred over a fixed interval.

```tsx
<ChartPrimaryYAxis
  valueType="Double"
  desiredIntervals={5}
/>
```

The actual interval count may vary according to the visible range and chart size.

Do not set both `interval` and `desiredIntervals` without a clear reason. Use `interval` for exact numeric spacing and `desiredIntervals` for automatic spacing guided by an approximate count.

## Range padding

Use `rangePadding` to control the space around the automatically calculated numeric minimum and maximum.

Supported values:

- `Auto`: automatically select padding behavior
- `None`: use the calculated data range without additional padding
- `Normal`: apply normal numeric range padding
- `Additional`: add one interval beyond the calculated range at both ends
- `Round`: round the boundaries to interval-aligned values

```tsx
<ChartPrimaryYAxis
  valueType="Double"
  rangePadding="Round"
/>
```

The public `ChartRangePadding` default is `Auto`. In automatic behavior, the chart uses orientation-appropriate padding.

### Padding examples

No extra padding:

```tsx
<ChartPrimaryYAxis
  valueType="Double"
  rangePadding="None"
/>
```

Add surrounding space:

```tsx
<ChartPrimaryYAxis
  valueType="Double"
  rangePadding="Additional"
/>
```

Round the range to clear interval boundaries:

```tsx
<ChartPrimaryYAxis
  valueType="Double"
  interval={10}
  rangePadding="Round"
/>
```

Range padding affects the displayed axis range, not the underlying data.

## Numeric label formatting

Place number formatting inside `ChartAxisLabel`.

```tsx
<ChartPrimaryYAxis valueType="Double">
  <ChartAxisLabel format="N2" />
</ChartPrimaryYAxis>
```

Documented formatting patterns include:

- `N` or `N2`: number format, optionally with decimal precision
- `C` or `C2`: currency format, optionally with decimal precision
- `P` or `P2`: percentage format, optionally with decimal precision

### Currency

```tsx
<ChartAxisLabel format="C0" />
```

### Percentage

```tsx
<ChartAxisLabel format="P1" />
```

Percentage formatting must match the value representation. For example, a value of `0.25` represents 25 percent when a standard percentage format is used.

### Custom units

Use `{value}` to add units or surrounding text.

```tsx
<ChartAxisLabel format="{value} kg" />
```

```tsx
<ChartAxisLabel format="$ {value}K" />
```

Do not place `format` directly on the axis when the Pure React component hierarchy requires `ChartAxisLabel`.

## Custom numeric formatter

Use `formatter` when a format string is insufficient.

```tsx
const compactNumber = (value: number, text: string): string => {
  if (Math.abs(value) >= 1_000_000) {
    return `${value / 1_000_000}M`;
  }

  if (Math.abs(value) >= 1_000) {
    return `${value / 1_000}K`;
  }

  return text;
};

<ChartPrimaryYAxis valueType="Double">
  <ChartAxisLabel formatter={compactNumber} />
</ChartPrimaryYAxis>
```

Keep the formatter deterministic and lightweight because it runs for each label.

## Decimal values

The numeric axis supports decimal values and intervals.

```tsx
<ChartPrimaryYAxis
  valueType="Double"
  minimum={0}
  maximum={1}
  interval={0.2}
>
  <ChartAxisLabel format="N1" />
</ChartPrimaryYAxis>
```

Do not convert decimal measurements to categories merely to control display text. Keep the numeric values and format their labels.

## Negative and positive values

A `Double` axis can display values on both sides of zero.

```tsx
const data = [
  { month: "Jan", change: -12 },
  { month: "Feb", change: 8 },
  { month: "Mar", change: 18 },
  { month: "Apr", change: -5 },
];

<ChartPrimaryYAxis
  valueType="Double"
  minimum={-20}
  maximum={20}
  interval={10}
>
  <ChartAxisLabel format="{value}%" />
</ChartPrimaryYAxis>
```

Use a logarithmic axis only for positive values. Use `Double` when zero or negative values are required.

## Inverted numeric axis

Set `inverted={true}` to display larger values before smaller values. The documented default is `false`.

```tsx
<ChartPrimaryYAxis
  valueType="Double"
  inverted={true}
/>
```

Inversion changes the visual direction only. It does not alter, sort, or negate the source values.

This is useful for lower-is-better metrics, ranking-style scales, depth, or other cases where descending visual order is intentional.

## Minor ticks and grid lines

Set `minorTicksPerInterval` on the axis, then add the minor tick and grid child components.

```tsx
<ChartPrimaryYAxis
  valueType="Double"
  interval={20}
  minorTicksPerInterval={4}
>
  <ChartMajorGridLines width={1} color="#D9D9D9" />
  <ChartMinorGridLines width={0.7} color="#EFEFEF" dashArray="2,2" />
  <ChartMajorTickLines width={1} height={6} color="#555555" />
  <ChartMinorTickLines width={1} height={3} color="#999999" />
</ChartPrimaryYAxis>
```

The documented default of `minorTicksPerInterval` is `0`.

## Numeric X-axis

A numeric axis can be used on the X-axis for scatter, bubble, and other continuous numeric-domain charts.

```tsx
const data = [
  { distance: 0, speed: 0 },
  { distance: 10, speed: 22 },
  { distance: 25, speed: 46 },
  { distance: 40, speed: 61 },
];

<ChartPrimaryXAxis
  valueType="Double"
  minimum={0}
  maximum={50}
  interval={10}
>
  <ChartAxisTitle text="Distance" />
  <ChartAxisLabel format="{value} km" />
</ChartPrimaryXAxis>

<ChartSeries
  dataSource={data}
  xField="distance"
  yField="speed"
  type="Scatter"
/>
```

Do not use a Category axis when proportional numeric spacing on the X-axis is required.

## Complete example

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartMajorGridLines,
  ChartMajorTickLines,
  ChartMinorGridLines,
  ChartMinorTickLines,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { month: "Jan", profit: -12.5 },
  { month: "Feb", profit: 8.2 },
  { month: "Mar", profit: 18.7 },
  { month: "Apr", profit: 26.4 },
  { month: "May", profit: 14.9 },
];

export default function NumericAxisChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis valueType="Category">
        <ChartAxisTitle text="Month" />
        <ChartAxisLabel edgeLabelPlacement="Shift" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis
        valueType="Double"
        minimum={-20}
        maximum={40}
        interval={10}
        rangePadding="Round"
        minorTicksPerInterval={4}
      >
        <ChartAxisTitle text="Profit Margin" />
        <ChartAxisLabel
          format="{value}%"
          color="#333333"
          fontSize="12px"
        />
        <ChartMajorGridLines width={1} color="#D9D9D9" />
        <ChartMinorGridLines
          width={0.7}
          color="#EFEFEF"
          dashArray="2,2"
        />
        <ChartMajorTickLines width={1} height={6} color="#555555" />
        <ChartMinorTickLines width={1} height={3} color="#999999" />
      </ChartPrimaryYAxis>

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="profit"
          type="Column"
          name="Profit Margin"
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Common errors

### Using `Numeric` as the value type

Incorrect:

```tsx
<ChartPrimaryYAxis valueType="Numeric" />
```

Correct:

```tsx
<ChartPrimaryYAxis valueType="Double" />
```

### Binding numeric strings

Incorrect:

```tsx
const data = [{ x: "A", y: "42" }];
```

Correct:

```tsx
const data = [{ x: "A", y: 42 }];
```

### Invalid range

Incorrect:

```tsx
<ChartPrimaryYAxis minimum={100} maximum={0} />
```

Correct the boundaries so `minimum < maximum`.

### Invalid interval

Incorrect:

```tsx
<ChartPrimaryYAxis interval={0} />
```

Use a positive interval or omit it for automatic calculation.

### Putting label format on the axis

Incorrect:

```tsx
<ChartPrimaryYAxis format="C0" />
```

Correct:

```tsx
<ChartPrimaryYAxis valueType="Double">
  <ChartAxisLabel format="C0" />
</ChartPrimaryYAxis>
```

### Using a Category axis for continuous numeric X values

Incorrect when numeric distance must remain proportional:

```tsx
<ChartPrimaryXAxis valueType="Category" />
```

Correct:

```tsx
<ChartPrimaryXAxis valueType="Double" />
```

## Validation checklist

Before returning a numeric-axis implementation:

1. Import all used axis components from `@syncfusion/react-charts`.
2. Set `valueType="Double"` with exact casing.
3. Ensure mapped numeric fields contain finite numbers, not unparsed strings.
4. Ensure explicit `minimum` and `maximum` are finite and satisfy `minimum < maximum`.
5. Use a positive `interval` when configured.
6. Use `desiredIntervals` only for approximate interval calculation.
7. Use only `Auto`, `None`, `Normal`, `Additional`, or `Round` for `rangePadding`.
8. Put numeric formatting on `ChartAxisLabel`.
9. Ensure percentage formatting matches the data representation.
10. Keep decimal values numeric and format labels rather than converting values to strings.
11. Use `Double` when zero or negative values must be shown.
12. Use `Logarithmic` only for valid positive ratio-based data.
13. Set `minorTicksPerInterval` when minor ticks or grid lines are required.
14. Use `inverted` only to reverse visual direction.
15. Emit valid, unescaped TSX.
