# Axis Position Reference

Use this reference to move an axis to the opposite side of the chart or make one axis cross another at a specific numeric, date, or category value.

## Component ownership

Axis-position settings belong to the axis component:

- Primary axes: `ChartPrimaryXAxis` and `ChartPrimaryYAxis`
- Additional axes: a named `ChartAxis` inside `ChartAxes`

Do not create `ChartSecondaryXAxis` or `ChartSecondaryYAxis` components. Use `ChartAxes` and named `ChartAxis` children for additional axes.

## Opposed position

Set `opposedPosition={true}` to move an axis to the side opposite its default position.

- Primary X-axis: moves from the bottom to the top
- Primary Y-axis: moves from the left to the right
- Default: `false`

```tsx
import {
  Chart,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

<Chart>
  <ChartPrimaryXAxis
    valueType="Category"
    opposedPosition={true}
  />
  <ChartPrimaryYAxis opposedPosition={true} />

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

`opposedPosition` changes the side on which the axis is rendered. It does not reverse axis values. Use `inverted={true}` when the requirement is to reverse the axis range.

## Named additional axis position

Place an additional axis inside `ChartAxes`, assign a unique `name`, and map the intended series using the matching `xAxisName` or `yAxisName`.

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
    <ChartAxis
      name="growthAxis"
      opposedPosition={true}
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
```

The axis `name` and the series axis-name mapping are case-sensitive and must match exactly.

## Axis crossing

Use the axis `crossAt` property to control where the current axis line intersects a target axis.

```tsx
crossAt={{
  value: 0,
  axis: "primaryYAxis",
  allowOverlap: false,
}}
```

`crossAt` uses these properties:

- `value`: the target-axis value where the current axis crosses
- `axis`: the target axis name
- `allowOverlap`: whether the crossed axis line may overlap axis labels, titles, and other axis elements

The documented default object is:

```tsx
{
  value: null,
  axis: "",
  allowOverlap: true,
}
```

### Crossing at a numeric value

The following pattern places the X-axis at Y value `0`. The target-axis name must be the actual name used by the chart for that Y-axis.

```tsx
<ChartPrimaryXAxis
  valueType="Category"
  crossAt={{
    value: 0,
    axis: "primaryYAxis",
    allowOverlap: false,
  }}
/>
```

Use a numeric crossing value only when the target axis uses compatible numeric values.

### Crossing at a date value

For a `DateTime` target axis, provide a date-compatible value.

```tsx
<ChartAxis
  name="valueAxis"
  crossAt={{
    value: new Date(2026, 0, 1),
    axis: "dateAxis",
    allowOverlap: false,
  }}
/>
```

### Crossing at a category value

For a category target axis, provide a category value that exists in the axis data.

```tsx
<ChartAxis
  name="valueAxis"
  crossAt={{
    value: "Q2",
    axis: "categoryAxis",
    allowOverlap: false,
  }}
/>
```

## Complete opposed-axis example

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
  { quarter: "Q1", revenue: 42 },
  { quarter: "Q2", revenue: 55 },
  { quarter: "Q3", revenue: 48 },
  { quarter: "Q4", revenue: 67 },
];

export default function AxisPositionChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis
        valueType="Category"
        opposedPosition={true}
      >
        <ChartAxisTitle text="Quarter" />
        <ChartAxisLabel position="Outside" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis opposedPosition={true}>
        <ChartAxisTitle text="Revenue" />
        <ChartAxisLabel format="{value}M" />
      </ChartPrimaryYAxis>

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="quarter"
          yField="revenue"
          type="Column"
          name="Revenue"
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Position versus related properties

Use the correct API for the requested behavior:

- Move axis to opposite side: `opposedPosition`
- Reverse minimum-to-maximum direction: `inverted`
- Cross another axis at a value: `crossAt`
- Place labels relative to the axis line: `ChartAxisLabel.position`
- Place tick marks relative to the axis line: the tick-position API documented for the owning axis
- Assign a series to another axis: axis `name` plus series `xAxisName` or `yAxisName`
- Assign an axis to a pane: `rowIndex` or `columnIndex`

Do not substitute one of these APIs for another.

## Validation checklist

Before returning an axis-position implementation:

1. Set `opposedPosition` on the axis, not on `ChartSeries` or `Chart`.
2. Use `opposedPosition={true}` only to move an axis to the opposite side.
3. Use `inverted` instead when the request is to reverse the scale direction.
4. Use `crossAt` for axis intersection.
5. Ensure `crossAt.value` is compatible with the target axis type.
6. Ensure `crossAt.axis` identifies the intended target axis.
7. Decide whether `allowOverlap` should permit overlap with labels and titles.
8. Define additional axes through `ChartAxes` and named `ChartAxis` children.
9. Match the additional axis `name` exactly with `xAxisName` or `yAxisName`.
10. Do not invent secondary-axis component tags.
11. Keep axis titles and labels nested inside their owning axes.
12. Emit valid, unescaped TSX.
