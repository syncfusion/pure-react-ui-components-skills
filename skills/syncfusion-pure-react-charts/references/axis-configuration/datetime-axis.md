# DateTime Axis Reference

Use a DateTime axis when point spacing must reflect actual elapsed time rather than equal category spacing.

## Component hierarchy

Set `valueType="DateTime"` on the axis. Put label formatting inside `ChartAxisLabel`.

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartPrimaryXAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

<Chart>
  <ChartPrimaryXAxis valueType="DateTime">
    <ChartAxisLabel format="MMM" />
  </ChartPrimaryXAxis>

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="date"
      yField="value"
      type="Line"
    />
  </ChartSeriesCollection>
</Chart>
```

The field mapped by `xField` must contain date-compatible values. Prefer JavaScript `Date` objects for unambiguous local date construction.

## When to use DateTime

Use `DateTime` when:

- Point positions must represent real elapsed time.
- Gaps between dates should produce corresponding visual gaps.
- The axis must use calendar-based intervals such as years, months, days, hours, minutes, or seconds.
- Financial, telemetry, monitoring, or time-series data uses actual timestamps.

Use `Category` instead when date-looking labels should remain equally spaced and source order is more important than elapsed time.

## Date data

Recommended:

```tsx
const data = [
  { date: new Date(2026, 0, 1), value: 35 },
  { date: new Date(2026, 0, 15), value: 42 },
  { date: new Date(2026, 2, 1), value: 38 },
];
```

Avoid locale-dependent date strings such as `"01/02/2026"`, because parsing can differ by environment. If the source provides strings, normalize them before binding.

```tsx
const chartData = sourceData.map((item) => ({
  ...item,
  date: new Date(item.date),
}));
```

Validate parsed dates before rendering:

```tsx
const isValidDate = (value: Date): boolean =>
  !Number.isNaN(value.getTime());
```

## Automatic range

When `minimum` and `maximum` are omitted, the axis calculates its visible range from the earliest and latest dates in the bound data.

```tsx
<ChartPrimaryXAxis valueType="DateTime" />
```

Use automatic range unless the user requests a fixed reporting period, synchronized charts, or a viewport that extends beyond existing points.

## Explicit range

Set `minimum` and `maximum` on the axis using date-compatible values.

```tsx
<ChartPrimaryXAxis
  valueType="DateTime"
  minimum={new Date(2026, 0, 1)}
  maximum={new Date(2026, 11, 31)}
/>
```

Validation rules:

- `minimum` must be earlier than `maximum`.
- Both boundaries must be valid dates.
- Use consistent timezone handling throughout the data and range values.
- Points outside the explicit range are not part of the visible range.

## Interval and interval type

Use `interval` with `intervalType` to control time-based label and tick spacing.

```tsx
<ChartPrimaryXAxis
  valueType="DateTime"
  interval={1}
  intervalType="Months"
/>
```

Supported `intervalType` values are:

- `Auto`
- `Years`
- `Months`
- `Days`
- `Hours`
- `Minutes`
- `Seconds`

The documented default is `Auto`. When `interval` is omitted, the chart calculates an interval from the range and available space.

### Interval examples

One label per year:

```tsx
<ChartPrimaryXAxis
  valueType="DateTime"
  interval={1}
  intervalType="Years"
/>
```

One label every three months:

```tsx
<ChartPrimaryXAxis
  valueType="DateTime"
  interval={3}
  intervalType="Months"
/>
```

One label every six hours:

```tsx
<ChartPrimaryXAxis
  valueType="DateTime"
  interval={6}
  intervalType="Hours"
/>
```

Use a positive interval. Do not use calendar interval types on axes whose `valueType` is not `DateTime` unless another dedicated axis reference explicitly supports them.

## Desired intervals and label density

`desiredIntervals` requests an approximate number of axis intervals. The actual number can vary with the range and available space.

```tsx
<ChartPrimaryXAxis
  valueType="DateTime"
  desiredIntervals={6}
/>
```

`maxLabelDensity` limits the maximum labels per 100 pixels of axis length. Its documented default is `3`.

```tsx
<ChartPrimaryXAxis
  valueType="DateTime"
  maxLabelDensity={2}
/>
```

Do not use `desiredIntervals`, `interval`, and density settings indiscriminately. Prefer explicit `interval` and `intervalType` when exact calendar spacing is required; otherwise allow automatic calculation.

## Range padding

Use `rangePadding` to control space around the calculated minimum and maximum.

Supported values:

- `Auto`: choose padding according to axis orientation and behavior
- `None`: no additional padding
- `Normal`: apply standard calculated padding
- `Additional`: add one interval at each end
- `Round`: round the visible boundaries to interval-aligned values

```tsx
<ChartPrimaryXAxis
  valueType="DateTime"
  rangePadding="Round"
/>
```

Do not assume padding changes the source data. It changes only the displayed axis range.

## Date label format

Place date formatting on `ChartAxisLabel` using `format`.

Documented Globalize-style examples include:

- `EEEE`: full weekday name, such as Monday
- `yMd`: year, month, and day
- `MMM`: abbreviated month, such as Jul
- `hm`: hour and minute
- `hms`: hour, minute, and second

```tsx
<ChartPrimaryXAxis
  valueType="DateTime"
  interval={1}
  intervalType="Months"
>
  <ChartAxisLabel format="MMM" />
</ChartPrimaryXAxis>
```

For daily labels:

```tsx
<ChartAxisLabel format="yMd" />
```

For time-of-day labels:

```tsx
<ChartAxisLabel format="hm" />
```

Use a format appropriate for the configured interval. For example, `MMM` is useful for monthly intervals but insufficient when multiple years are displayed without additional year context.

## Dense DateTime labels

Use label collision handling inside `ChartAxisLabel`.

```tsx
<ChartPrimaryXAxis
  valueType="DateTime"
  interval={1}
  intervalType="Months"
  maxLabelDensity={2}
>
  <ChartAxisLabel
    format="MMM"
    intersectMode="Rotate45"
    edgeLabelPlacement="Shift"
  />
</ChartPrimaryXAxis>
```

Relevant label options include:

- `intersectMode`: `None`, `Hide`, `Trim`, `Wrap`, `MultipleRows`, `Rotate45`, or `Rotate90`
- `rotationAngle`: manual angle
- `edgeLabelPlacement`: `None`, `Hide`, or `Shift`
- `enableTrim` and `maxLabelWidth`
- `enableWrap` and `maxLabelWidth`

## Inverted DateTime axis

Set `inverted={true}` to reverse the chronological direction. The documented default is `false`.

```tsx
<ChartPrimaryXAxis
  valueType="DateTime"
  inverted={true}
/>
```

This reverses the axis direction. It does not sort or mutate the source data.

## Complete monthly example

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

const data = [
  { date: new Date(2026, 0, 1), sales: 35 },
  { date: new Date(2026, 1, 1), sales: 42 },
  { date: new Date(2026, 2, 1), sales: 38 },
  { date: new Date(2026, 3, 1), sales: 51 },
  { date: new Date(2026, 4, 1), sales: 47 },
  { date: new Date(2026, 5, 1), sales: 59 },
];

export default function DateTimeAxisChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis
        valueType="DateTime"
        minimum={new Date(2026, 0, 1)}
        maximum={new Date(2026, 5, 30)}
        interval={1}
        intervalType="Months"
        rangePadding="Round"
        maxLabelDensity={3}
      >
        <ChartAxisTitle text="Month" />
        <ChartAxisLabel
          format="MMM"
          edgeLabelPlacement="Shift"
          intersectMode="Rotate45"
        />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double">
        <ChartAxisTitle text="Sales" />
        <ChartAxisLabel format="{value}" />
      </ChartPrimaryYAxis>

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="date"
          yField="sales"
          type="Line"
          name="Sales"
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Common errors

### Using Category for irregular dates

Incorrect when elapsed time should control spacing:

```tsx
<ChartPrimaryXAxis valueType="Category" />
```

Correct:

```tsx
<ChartPrimaryXAxis valueType="DateTime" />
```

### Mapping invalid date values

Incorrect:

```tsx
const data = [{ date: "not-a-date", value: 10 }];
```

Normalize and validate source date values before binding.

### Putting label format on the axis

Do not guess that `format` belongs directly to `ChartPrimaryXAxis`. In this Pure React structure, place it on `ChartAxisLabel`:

```tsx
<ChartPrimaryXAxis valueType="DateTime">
  <ChartAxisLabel format="MMM" />
</ChartPrimaryXAxis>
```

### Using unsupported interval values

Use exact plural literal values such as `Months`, `Days`, and `Hours`. Do not emit `Month`, `Day`, or lowercase values.

## Validation checklist

Before returning a DateTime-axis implementation:

1. Import all used axis components from `@syncfusion/react-charts`.
2. Set `valueType="DateTime"` on the intended axis.
3. Ensure the mapped date field exists in every applicable data object.
4. Prefer valid `Date` objects or consistently parsed date-compatible values.
5. Verify every date with `getTime()` when parsing uncertain external input.
6. Ensure `minimum` is earlier than `maximum`.
7. Use only `Auto`, `Years`, `Months`, `Days`, `Hours`, `Minutes`, or `Seconds` for `intervalType`.
8. Use a positive `interval` when explicitly configured.
9. Put date formatting on `ChartAxisLabel.format`.
10. Match the date format to the interval granularity.
11. Use label collision handling when the timeline is dense.
12. Use `Category` instead when equal spacing is intended.
13. Emit valid, unescaped TSX.
