# Axis Configuration Reference

Use this umbrella reference to route axis-related requests to the correct detailed page. In Pure React Chart, configure primary axes with `ChartPrimaryXAxis` and `ChartPrimaryYAxis`, and place axis-specific child components inside the axis they customize.

## Includes

- [Axis customization](./axis-configuration/axis-customization.md)
- [Axis labels](./axis-configuration/axis-labels.md)
- [Axis position](./axis-configuration/axis-position.md)
- [Category axis](./axis-configuration/category-axis.md)
- [DateTime axis](./axis-configuration/datetime-axis.md)
- [Numeric axis](./axis-configuration/numeric-axis.md)
- [Logarithmic axis](./axis-configuration/log-axis.md)
- [Multiple axes](./axis-configuration/multiple-axes.md)
- [Multiple panes](./axis-configuration/multiple-panes.md)

## Base axis hierarchy

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
    <ChartAxisTitle text="Month" />
    <ChartAxisLabel edgeLabelPlacement="Shift" />
  </ChartPrimaryXAxis>

  <ChartPrimaryYAxis valueType="Double">
    <ChartAxisTitle text="Sales" />
    <ChartAxisLabel format="{value}" />
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

Keep `ChartAxisTitle`, `ChartAxisLabel`, grid-line, tick-line, stripline, scrollbar, and crosshair-tooltip components inside the axis they configure.

## Axis value types

Use exact `valueType` values:

- `Double`: continuous numeric values
- `Category`: discrete text or category values
- `DateTime`: real date and time values
- `Logarithmic`: positive numeric values spanning a wide range

```tsx
<ChartPrimaryXAxis valueType="Category" />
<ChartPrimaryYAxis valueType="Double" />
```

Do not use `Numeric`; use `Double`.

## Core axis properties

The current Pure React axis API includes properties for:

- range: `minimum`, `maximum`, `interval`, `rangePadding`, and `startFromZero`
- direction and placement: `inverted`, `opposedPosition`, and `tickPosition`
- layout: `rowIndex`, `columnIndex`, `span`, and `plotOffset`
- identity: `name`
- type-specific behavior: `valueType`, `intervalType`, `logBase`, and DateTime skeleton settings
- zoom state: `zoomFactor` and `zoomPosition`
- appearance: `lineStyle`
- label density: `desiredIntervals`, `maxLabelDensity`, and `minorTicksPerInterval`

Apply only properties relevant to the selected axis type.

## Axis child configuration

Use child components instead of guessed nested objects or EJ2 axis properties.

```tsx
<ChartPrimaryYAxis
  valueType="Double"
  lineStyle={{
    color: "#555555",
    width: 1,
    dashArray: "",
  }}
>
  <ChartAxisTitle
    text="Revenue"
    fontSize="14px"
    fontWeight="Bold"
  />
  <ChartAxisLabel
    format="${value}"
    fontSize="12px"
    color="#333333"
  />
</ChartPrimaryYAxis>
```

Do not use an EJ2-style `title={{ text: "Revenue" }}` or `labelStyle={{ ... }}` when the Pure React architecture provides `ChartAxisTitle` and `ChartAxisLabel` children.

## Additional axes

Define additional axes with `ChartAxes` and `ChartAxis`. Give each axis a unique `name`, then map the series through the matching `xAxisName` or `yAxisName`.

```tsx
<ChartAxes>
  <ChartAxis
    name="growthAxis"
    valueType="Double"
    opposedPosition={true}
  >
    <ChartAxisTitle text="Growth (%)" />
    <ChartAxisLabel format="{value}%" />
  </ChartAxis>
</ChartAxes>

<ChartSeries
  dataSource={data}
  xField="month"
  yField="growth"
  type="Line"
  yAxisName="growthAxis"
/>
```

Do not invent `ChartSecondaryXAxis` or `ChartSecondaryYAxis` components.

## Multiple panes

Use chart rows and columns with axis `rowIndex`, `columnIndex`, and `span` when related series need separate plot regions. Keep each axis-to-pane and series-to-axis mapping explicit.

Read [Multiple panes](./axis-configuration/multiple-panes.md) before generating pane configuration.

## Routing rules

Use the detailed reference that matches the request:

- Titles, lines, ticks, grids, inversion, or named axes: `axis-customization.md`
- Formatting, wrapping, trimming, rotation, placement, or collision: `axis-labels.md`
- Opposed, crossed, inside, or outside placement: `axis-position.md`
- Discrete labels and category placement: `category-axis.md`
- Time-based intervals and formatting: `datetime-axis.md`
- Continuous numeric scales and range padding: `numeric-axis.md`
- Orders of magnitude and log base: `log-axis.md`
- Different units or scales: `multiple-axes.md`
- Separate plot regions: `multiple-panes.md`

## Validation checklist

Before returning any axis implementation:

1. Match `valueType` to the bound data.
2. Use `Double`, not `Numeric`, for numeric axes.
3. Keep axis child components inside the owning axis.
4. Use `ChartAxisTitle` and `ChartAxisLabel` for title and label configuration.
5. Use valid axis range and interval values.
6. Use `inverted` only to reverse axis direction.
7. Use `ChartAxes` and named `ChartAxis` components for additional axes.
8. Match `xAxisName` or `yAxisName` exactly to the additional axis name.
9. Use `rowIndex`, `columnIndex`, and `span` only with a valid pane layout.
10. Read the matching detailed reference before adding type-specific behavior.
11. Do not mix EJ2 axis objects with Pure React child components.
12. Ensure every imported symbol is used.
13. Emit valid, unescaped TSX.
