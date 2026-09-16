# Axis Customization Reference

Use this reference for axis titles, axis lines, major and minor tick lines, major and minor grid lines, axis inversion, and additional named axes in Syncfusion Pure React Charts.

## Component hierarchy

Use `ChartPrimaryXAxis` and `ChartPrimaryYAxis` as the primary axis hosts. Place axis-specific child configuration components inside the owning axis.

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

<Chart>
  <ChartPrimaryXAxis valueType="Category">
    <ChartAxisTitle text="Month" />
    <ChartAxisLabel />
    <ChartMajorGridLines width={0} />
    <ChartMajorTickLines width={1} height={5} />
    <ChartMinorGridLines width={0} />
    <ChartMinorTickLines width={0} height={0} />
  </ChartPrimaryXAxis>

  <ChartPrimaryYAxis>
    <ChartAxisTitle text="Sales" />
    <ChartAxisLabel format="{value}" />
    <ChartMajorGridLines width={1} color="#E0E0E0" />
    <ChartMajorTickLines width={1} height={5} />
  </ChartPrimaryYAxis>

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="month"
      yField="sales"
      type="Column"
    />
  </ChartSeriesCollection>
</Chart>
```

Do not move axis child components directly under `Chart`. Keep each title, label, grid-line, and tick-line component inside the axis it configures.

## Axis value types

Set the axis `valueType` to match the mapped values:

- `Double`: continuous numeric values
- `Category`: string or discrete category values
- `DateTime`: date and time values
- `Logarithmic`: positive numeric values displayed on a logarithmic scale

```tsx
<ChartPrimaryXAxis valueType="Category" />
<ChartPrimaryYAxis valueType="Double" />
```

Do not use `Numeric` as an axis `valueType`; use `Double`.

## Axis title

Place `ChartAxisTitle` inside the relevant axis. Its documented properties include:

- `text: string`, default `""`
- `align: "Left" | "Center" | "Right"`, default `"Center"`
- `color: string`, default `""`
- `fontFamily: string`, default `""`
- `fontSize: string`, default `""`
- `fontStyle: string`, default `""`
- `fontWeight: string`, default `""`
- `opacity: number`, default `1`
- `overflow: "Wrap" | "Trim" | "None"`, default `"Wrap"`
- `padding: number`, default `5`
- `rotationAngle: number | null`, default `null`

```tsx
<ChartPrimaryXAxis>
  <ChartAxisTitle
    text="Month"
    align="Center"
    fontSize="14px"
    fontWeight="Bold"
    color="#333333"
    padding={8}
  />
</ChartPrimaryXAxis>
```

`rotationAngle` accepts an angle value. When omitted, the component calculates the title angle from the axis orientation and position.

```tsx
<ChartPrimaryYAxis>
  <ChartAxisTitle text="Revenue" rotationAngle={270} />
</ChartPrimaryYAxis>
```

## Axis line

Configure the axis line through the axis `lineStyle` object:

- `color: string`, default `""`
- `dashArray: string`, default `""`
- `width: number`, default `1`

```tsx
<ChartPrimaryXAxis
  lineStyle={{
    color: "#555555",
    width: 1,
    dashArray: "4,2",
  }}
/>
```

Use `width: 0` to hide the axis line when required.

## Major tick lines

Place `ChartMajorTickLines` inside the owning axis. Use:

- `width` for line thickness
- `height` for tick length
- `color` for the tick color

```tsx
<ChartPrimaryXAxis>
  <ChartMajorTickLines width={1} height={8} color="#555555" />
</ChartPrimaryXAxis>
```

## Minor tick lines

Minor ticks require an axis-level minor tick count. Set `minorTicksPerInterval` on the axis, then configure their appearance with `ChartMinorTickLines`.

```tsx
<ChartPrimaryYAxis minorTicksPerInterval={4}>
  <ChartMinorTickLines width={1} height={4} color="#999999" />
</ChartPrimaryYAxis>
```

The documented default of `minorTicksPerInterval` is `0`, so minor ticks are not created unless a positive count is configured.

## Major grid lines

Place `ChartMajorGridLines` inside the owning axis. Use:

- `width` for thickness
- `color` for color
- `dashArray` for an SVG-style dash pattern

```tsx
<ChartPrimaryYAxis>
  <ChartMajorGridLines
    width={1}
    color="#D9D9D9"
    dashArray="4,2"
  />
</ChartPrimaryYAxis>
```

Use `width={0}` to hide major grid lines.

## Minor grid lines

Set `minorTicksPerInterval` on the axis and configure `ChartMinorGridLines` as a child.

```tsx
<ChartPrimaryYAxis minorTicksPerInterval={4}>
  <ChartMinorGridLines
    width={0.7}
    color="#EFEFEF"
    dashArray="2,2"
  />
</ChartPrimaryYAxis>
```

The documented `ChartMinorGridLines` defaults are `width: 0.7`, `color: ""`, and `dashArray: ""`. A width of `0` hides the minor grid lines.

## Inverted axis

Set `inverted={true}` on the axis whose direction must be reversed. The documented default is `false`.

```tsx
<Chart>
  <ChartPrimaryXAxis valueType="Category" inverted={true} />
  <ChartPrimaryYAxis inverted={true} />
</Chart>
```

`inverted` reverses the axis range from maximum-to-minimum instead of changing the chart orientation. To exchange horizontal and vertical chart orientation, use the separate root `Chart` transposition behavior documented by the relevant chart-type reference.

## Additional named axes

Use `ChartAxes` for additional X or Y axes. Give every additional axis a unique `name`, then map the intended series with the matching `xAxisName` or `yAxisName`.

```tsx
import {
  Chart,
  ChartAxes,
  ChartAxis,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

<Chart>
  <ChartAxes>
    <ChartAxis name="growthAxis" opposedPosition={true}>
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
```

Do not invent `ChartSecondaryXAxis` or `ChartSecondaryYAxis` tags. The additional-axis name and the series mapping must match exactly.

Relevant additional-axis properties include:

- `name: string`, default `""`
- `opposedPosition: boolean`
- `rowIndex: number`
- `columnIndex: number`, default `0`
- `minimum`
- `maximum`
- `interval`
- `valueType`
- `inverted: boolean`, default `false`
- `lineStyle`
- `minorTicksPerInterval: number`, default `0`

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
  { month: "Jan", sales: 35 },
  { month: "Feb", sales: 42 },
  { month: "Mar", sales: 38 },
  { month: "Apr", sales: 51 },
];

export default function AxisCustomizationChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis
        valueType="Category"
        lineStyle={{ color: "#555555", width: 1, dashArray: "" }}
      >
        <ChartAxisTitle
          text="Month"
          align="Center"
          fontSize="14px"
          fontWeight="Bold"
        />
        <ChartAxisLabel />
        <ChartMajorGridLines width={0} />
        <ChartMajorTickLines width={1} height={6} color="#555555" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis
        valueType="Double"
        minorTicksPerInterval={4}
        lineStyle={{ color: "#555555", width: 1, dashArray: "" }}
      >
        <ChartAxisTitle
          text="Sales"
          align="Center"
          fontSize="14px"
          fontWeight="Bold"
        />
        <ChartAxisLabel format="{value}" />
        <ChartMajorGridLines width={1} color="#D9D9D9" />
        <ChartMinorGridLines width={0.7} color="#EFEFEF" dashArray="2,2" />
        <ChartMajorTickLines width={1} height={6} color="#555555" />
        <ChartMinorTickLines width={1} height={3} color="#999999" />
      </ChartPrimaryYAxis>

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

## Validation checklist

Before returning an axis sample:

1. Import every used axis component from `@syncfusion/react-charts`.
2. Keep axis title, label, grid-line, and tick-line components inside the owning axis.
3. Match `valueType` to the mapped data type.
4. Use `Double`, not `Numeric`, for numeric axes.
5. Set `minorTicksPerInterval` when minor ticks or minor grid lines are required.
6. Use `inverted` only to reverse axis direction.
7. Use `ChartAxes` and a named `ChartAxis` for additional axes.
8. Match the additional axis `name` exactly with the series `xAxisName` or `yAxisName`.
9. Do not use invented secondary-axis component tags.
10. Use valid TSX without HTML entities.
