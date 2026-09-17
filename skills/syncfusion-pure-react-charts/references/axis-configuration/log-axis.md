# Logarithmic Axis Reference

Use a logarithmic axis when positive numeric values span several orders of magnitude and equal ratios should receive equal visual spacing.

## Component hierarchy

Set `valueType="Logarithmic"` on the numeric axis. Place title and label configuration inside the owning axis.

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
  <ChartPrimaryXAxis valueType="Category">
    <ChartAxisTitle text="Category" />
  </ChartPrimaryXAxis>

  <ChartPrimaryYAxis valueType="Logarithmic">
    <ChartAxisTitle text="Value" />
    <ChartAxisLabel format="{value}" />
  </ChartPrimaryYAxis>

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

Use `Logarithmic` with exact casing. Do not use `Log`, `LogAxis`, ` logarithmic`, or another guessed value.

## When to use a logarithmic axis

Use `Logarithmic` when:

- Values are positive.
- Values differ by powers or large multiplicative factors.
- Relative change is more important than absolute difference.
- A linear axis compresses smaller values so severely that meaningful variation is hidden.

Use `Double` instead when values are close in magnitude, include zero or negative values, or must be interpreted by their absolute differences.

## Data requirements

Every value plotted against a logarithmic axis must be greater than zero.

Valid:

```tsx
const data = [
  { category: "A", value: 1 },
  { category: "B", value: 10 },
  { category: "C", value: 100 },
  { category: "D", value: 1000 },
];
```

Invalid:

```tsx
const data = [
  { category: "A", value: 0 },
  { category: "B", value: -10 },
];
```

Do not silently convert zero or negative business values into positive values. If such values are meaningful, use a linear `Double` axis or transform the data only when the user explicitly requests and understands that transformation.

Validate external data before binding:

```tsx
const logarithmicData = sourceData.filter(
  (item) => Number.isFinite(item.value) && item.value > 0,
);
```

Filtering is appropriate only when removing invalid records is acceptable. Otherwise report the invalid values instead of changing the dataset silently.

## Automatic range

When `minimum` and `maximum` are omitted, the axis calculates its range from the positive values mapped to it.

```tsx
<ChartPrimaryYAxis valueType="Logarithmic" />
```

Use automatic range unless the user requests a fixed scale or multiple charts must share the same logarithmic range.

## Explicit range

Set `minimum` and `maximum` on the logarithmic axis to control the visible range.

```tsx
<ChartPrimaryYAxis
  valueType="Logarithmic"
  minimum={1}
  maximum={100000}
/>
```

Validation rules:

- `minimum` must be greater than zero.
- `maximum` must be greater than zero.
- `minimum` must be less than `maximum`.
- The range should include the positive values that need to remain visible.
- Do not pass exponent values when the API expects actual axis values.

For base 10, use `minimum={1}` and `maximum={1000}` to represent values from 10⁰ through 10³. Do not use `minimum={0}` merely to represent exponent zero.

## Logarithmic base

Use `logBase` to set the logarithm base. The documented default is `10`.

```tsx
<ChartPrimaryYAxis
  valueType="Logarithmic"
  logBase={10}
/>
```

For base 5:

```tsx
<ChartPrimaryYAxis
  valueType="Logarithmic"
  logBase={5}
/>
```

With base 5, key powers include:

- 5⁻² = 0.04
- 5⁻¹ = 0.2
- 5⁰ = 1
- 5¹ = 5
- 5² = 25

Use a valid positive base. Do not use `1`, because logarithm base 1 is undefined. Prefer a base greater than 1 for conventional increasing logarithmic scales.

`logBase` has an effect only when `valueType="Logarithmic"`.

## Interval

Use `interval` to control the spacing between logarithmic labels in powers of `logBase`.

```tsx
<ChartPrimaryYAxis
  valueType="Logarithmic"
  logBase={10}
  interval={1}
/>
```

For `logBase={10}` and `interval={1}`, major labels progress by consecutive powers, such as:

- 10⁰ = 1
- 10¹ = 10
- 10² = 100
- 10³ = 1000

For `logBase={10}` and `interval={2}`, labels progress by every second power:

```tsx
<ChartPrimaryYAxis
  valueType="Logarithmic"
  logBase={10}
  interval={2}
/>
```

This yields key labels such as 1, 100, 10000, and so on.

Use a positive interval. Larger intervals reduce label density but may hide useful intermediate scale context.

## Label formatting

Place label formatting inside `ChartAxisLabel`.

```tsx
<ChartPrimaryYAxis
  valueType="Logarithmic"
  logBase={10}
>
  <ChartAxisLabel format="{value}" />
</ChartPrimaryYAxis>
```

For custom units:

```tsx
<ChartAxisLabel format="{value} units" />
```

Use a formatter when values need compact notation:

```tsx
const formatLogLabel = (value: number, text: string): string => {
  if (value >= 1_000_000) return `${value / 1_000_000}M`;
  if (value >= 1_000) return `${value / 1_000}K`;
  return text;
};

<ChartPrimaryYAxis valueType="Logarithmic">
  <ChartAxisLabel formatter={formatLogLabel} />
</ChartPrimaryYAxis>
```

Do not change logarithmic scaling by formatting labels. Formatting changes displayed text only.

## Grid and tick lines

Major grid and tick lines align with logarithmic intervals. Configure appearance through child components inside the logarithmic axis.

```tsx
<ChartPrimaryYAxis
  valueType="Logarithmic"
  logBase={10}
  interval={1}
  minorTicksPerInterval={4}
>
  <ChartMajorGridLines width={1} color="#D9D9D9" />
  <ChartMinorGridLines width={0.7} color="#EFEFEF" dashArray="2,2" />
  <ChartMajorTickLines width={1} height={6} color="#555555" />
  <ChartMinorTickLines width={1} height={3} color="#999999" />
</ChartPrimaryYAxis>
```

Set `minorTicksPerInterval` on the axis when minor grid or tick lines are required. Its documented default is `0`.

## Inverted logarithmic axis

Set `inverted={true}` to reverse the displayed direction while retaining logarithmic scaling.

```tsx
<ChartPrimaryYAxis
  valueType="Logarithmic"
  inverted={true}
/>
```

This changes the visual direction only. It does not transform or reorder the source data.

## Complete example

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartMajorGridLines,
  ChartMajorTickLines,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { category: "Sensor A", value: 1 },
  { category: "Sensor B", value: 10 },
  { category: "Sensor C", value: 100 },
  { category: "Sensor D", value: 1000 },
  { category: "Sensor E", value: 10000 },
];

export default function LogarithmicAxisChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis valueType="Category">
        <ChartAxisTitle text="Sensor" />
        <ChartAxisLabel edgeLabelPlacement="Shift" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis
        valueType="Logarithmic"
        minimum={1}
        maximum={10000}
        logBase={10}
        interval={1}
      >
        <ChartAxisTitle text="Measurement" />
        <ChartAxisLabel format="{value}" />
        <ChartMajorGridLines width={1} color="#D9D9D9" />
        <ChartMajorTickLines width={1} height={6} color="#555555" />
      </ChartPrimaryYAxis>

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="category"
          yField="value"
          type="Column"
          name="Measurement"
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Common errors

### Including zero

Incorrect:

```tsx
const data = [{ x: "A", y: 0 }];
<ChartPrimaryYAxis valueType="Logarithmic" />
```

A logarithmic axis cannot represent zero. Use a `Double` axis or correct the data according to the user's requirement.

### Including negative values

Incorrect:

```tsx
const data = [{ x: "A", y: -100 }];
```

Negative values are not valid on the documented logarithmic axis.

### Using the wrong value type

Incorrect:

```tsx
<ChartPrimaryYAxis valueType="Log" />
```

Correct:

```tsx
<ChartPrimaryYAxis valueType="Logarithmic" />
```

### Configuring `logBase` without logarithmic value type

Incorrect:

```tsx
<ChartPrimaryYAxis valueType="Double" logBase={10} />
```

Correct:

```tsx
<ChartPrimaryYAxis valueType="Logarithmic" logBase={10} />
```

### Treating minimum and maximum as exponents

Incorrect when the intended visible values are 1 through 1000:

```tsx
<ChartPrimaryYAxis
  valueType="Logarithmic"
  minimum={0}
  maximum={3}
/>
```

Correct:

```tsx
<ChartPrimaryYAxis
  valueType="Logarithmic"
  minimum={1}
  maximum={1000}
/>
```

## Validation checklist

Before returning a logarithmic-axis implementation:

1. Import all used axis components from `@syncfusion/react-charts`.
2. Set `valueType="Logarithmic"` with exact casing.
3. Ensure every mapped value is finite and greater than zero.
4. Do not silently alter zero or negative business values.
5. Ensure explicit `minimum` and `maximum` values are positive.
6. Ensure `minimum < maximum`.
7. Use a valid `logBase`; the documented default is `10`.
8. Do not use `logBase={1}`.
9. Use a positive `interval`.
10. Interpret logarithmic `interval` as spacing between powers of the base.
11. Put formatting on `ChartAxisLabel`.
12. Set `minorTicksPerInterval` when minor lines are required.
13. Use `Double` when linear distances or non-positive values must be shown.
14. Emit valid, unescaped TSX.
