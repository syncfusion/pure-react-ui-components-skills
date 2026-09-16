# Crosshair Reference

Use `ChartCrosshair` to draw pointer-following reference lines that help users read values precisely along chart axes. Place it directly inside `Chart`.

## Component hierarchy

```tsx
import {
  Chart,
  ChartCrosshair,
  ChartPrimaryXAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { month: "Jan", value: 35 },
  { month: "Feb", value: 42 },
  { month: "Mar", value: 38 },
];

<Chart>
  <ChartPrimaryXAxis valueType="Category" />

  <ChartCrosshair enable={true} />

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="month"
      yField="value"
      type="Line"
    />
  </ChartSeriesCollection>
</Chart>
```

Do not place `ChartCrosshair` inside a series, axis, or series collection.

## Crosshair properties

The Pure React `ChartCrosshairProps` API documents:

- `enable`: enables the crosshair; default `false`
- `lineType`: controls line orientation; default `Both`
- `lineStyle`: configures the line color, width, dash pattern, and opacity
- `snap`: snaps to the nearest visible point; default `true`
- `highlightCategory`: highlights the hovered category range; default `false`

```tsx
<ChartCrosshair
  enable={true}
  lineType="Both"
  snap={true}
/>
```

## Line type

Use exact `lineType` values:

- `Vertical`: displays only the vertical line
- `Horizontal`: displays only the horizontal line
- `Both`: displays both lines

```tsx
<ChartCrosshair
  enable={true}
  lineType="Vertical"
/>
```

Do not use `None`, lowercase values, or EJ2-specific alternatives.

## Line appearance

Configure appearance through `lineStyle`.

```tsx
<ChartCrosshair
  enable={true}
  lineType="Both"
  lineStyle={{
    color: "#777777",
    width: 1,
    dashArray: "4,2",
    opacity: 0.8,
  }}
/>
```

Use:

- `color` for a valid CSS color
- `width` for line thickness
- `dashArray` for an SVG-style dash pattern
- `opacity` for transparency from `0` through `1`

The documented default line style uses width `1` with empty color and dash values.

## Snap behavior

`ChartCrosshair.snap` controls whether the crosshair follows data points or the free pointer position.

```tsx
<ChartCrosshair
  enable={true}
  snap={true}
/>
```

- `snap={true}` aligns the crosshair with the nearest visible data point.
- `snap={false}` lets the crosshair move freely using the pointer coordinates.

Use snapping when exact point inspection matters. Disable it when users need free coordinate inspection across the plot area.

## Category highlighting

Set `highlightCategory={true}` to shade the complete category band under the pointer.

```tsx
<Chart>
  <ChartPrimaryXAxis valueType="Category" />

  <ChartCrosshair
    enable={true}
    highlightCategory={true}
  />
</Chart>
```

This behavior applies to category axes. Do not enable it for `Double`, `DateTime`, or `Logarithmic` axes and expect category-band highlighting.

## Crosshair axis labels

Use `ChartCrosshairTooltip` inside each axis that needs a crosshair label.

```tsx
import {
  ChartCrosshairTooltip,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
} from "@syncfusion/react-charts";

<ChartPrimaryXAxis valueType="DateTime">
  <ChartCrosshairTooltip enable={true} />
</ChartPrimaryXAxis>

<ChartPrimaryYAxis valueType="Double">
  <ChartCrosshairTooltip enable={true} />
</ChartPrimaryYAxis>
```

The required hierarchy is:

```text
Chart
├── ChartPrimaryXAxis
│   └── ChartCrosshairTooltip
├── ChartPrimaryYAxis
│   └── ChartCrosshairTooltip
└── ChartCrosshair
```

`ChartCrosshair` remains chart-level configuration. `ChartCrosshairTooltip` is axis-level configuration.

## Crosshair tooltip properties

The Pure React `ChartCrosshairTooltipProps` API documents:

- `enable`: displays the axis tooltip; default `false`
- `fill`: tooltip background color; default empty
- `formatter`: formats the tooltip text
- `textStyle`: configures tooltip text appearance

```tsx
<ChartCrosshairTooltip
  enable={true}
  fill="#333333"
  textStyle={{
    color: "#FFFFFF",
    fontFamily: "Arial",
    fontSize: "12px",
    fontStyle: "Normal",
    fontWeight: "Normal",
    opacity: 1,
  }}
/>
```

Use `fontSize`, not `size`, within `textStyle`.

## Crosshair tooltip formatter

The documented formatter signature is:

```tsx
(value: number, text: string) => string | boolean
```

Example:

```tsx
const formatValue = (
  value: number,
  text: string,
): string => {
  return `${text} units`;
};

<ChartPrimaryYAxis valueType="Double">
  <ChartCrosshairTooltip
    enable={true}
    formatter={formatValue}
  />
</ChartPrimaryYAxis>
```

Use the provided `text` when the chart's existing number or date formatting should be preserved. Return a replacement string only when custom output is required.

Do not assume the formatter receives a point, series, or axis object.

## DateTime crosshair

Use real date values with a DateTime axis.

```tsx
const data = [
  { time: new Date(2026, 8, 7, 10, 0), value: 42 },
  { time: new Date(2026, 8, 7, 10, 5), value: 57 },
  { time: new Date(2026, 8, 7, 10, 10), value: 49 },
];

<ChartPrimaryXAxis
  valueType="DateTime"
  interval={5}
  intervalType="Minutes"
>
  <ChartAxisLabel format="hm" />
  <ChartCrosshairTooltip enable={true} />
</ChartPrimaryXAxis>
```

Keep the DateTime axis label format and crosshair tooltip text appropriate for the displayed interval.

## Crosshair and regular tooltip

`ChartCrosshairTooltip` and `ChartTooltip` serve different purposes:

- `ChartCrosshairTooltip` displays an axis value at the crosshair intersection.
- `ChartTooltip` displays information about a chart data point.

Use both only when the chart requires both axis reading and point details.

```tsx
<Chart>
  <ChartCrosshair enable={true} />
  <ChartTooltip enable={true} />

  <ChartPrimaryXAxis valueType="Category">
    <ChartCrosshairTooltip enable={true} />
  </ChartPrimaryXAxis>
</Chart>
```

Do not use `ChartTooltip` as a substitute for crosshair axis labels.

## Crosshair with multiple axes

Place `ChartCrosshairTooltip` inside every primary or named axis that must display a crosshair label.

```tsx
<ChartPrimaryYAxis valueType="Double">
  <ChartCrosshairTooltip enable={true} />
</ChartPrimaryYAxis>

<ChartAxes>
  <ChartAxis
    name="growthAxis"
    valueType="Double"
    opposedPosition={true}
  >
    <ChartCrosshairTooltip
      enable={true}
      formatter={(value, text) => `${text}%`}
    />
  </ChartAxis>
</ChartAxes>
```

Keep each tooltip inside its owning axis. Do not configure axis tooltips through a single chart-level collection.

## Complete example

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartCrosshair,
  ChartCrosshairTooltip,
  ChartMarker,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { time: new Date(2026, 8, 7, 10, 0), value: 42 },
  { time: new Date(2026, 8, 7, 10, 5), value: 57 },
  { time: new Date(2026, 8, 7, 10, 10), value: 49 },
  { time: new Date(2026, 8, 7, 10, 15), value: 63 },
  { time: new Date(2026, 8, 7, 10, 20), value: 55 },
];

export default function CrosshairChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis
        valueType="DateTime"
        interval={5}
        intervalType="Minutes"
      >
        <ChartAxisTitle text="Time" />
        <ChartAxisLabel format="hm" />
        <ChartCrosshairTooltip
          enable={true}
          fill="#333333"
          textStyle={{
            color: "#FFFFFF",
            fontFamily: "Arial",
            fontSize: "12px",
            fontStyle: "Normal",
            fontWeight: "Normal",
            opacity: 1,
          }}
        />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double">
        <ChartAxisTitle text="Value" />
        <ChartAxisLabel format="{value}" />
        <ChartCrosshairTooltip
          enable={true}
          fill="#333333"
          formatter={(value, text) => `${text} units`}
          textStyle={{
            color: "#FFFFFF",
            fontFamily: "Arial",
            fontSize: "12px",
            fontStyle: "Normal",
            fontWeight: "Normal",
            opacity: 1,
          }}
        />
      </ChartPrimaryYAxis>

      <ChartCrosshair
        enable={true}
        lineType="Both"
        snap={true}
        lineStyle={{
          color: "#777777",
          width: 1,
          dashArray: "4,2",
          opacity: 0.8,
        }}
      />

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="time"
          yField="value"
          type="Line"
          name="Value"
          width={2}
        >
          <ChartMarker
            visible={true}
            shape="Circle"
            width={7}
            height={7}
          />
        </ChartSeries>
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Migration from EJ2 React

Do not copy the EJ2 object-style crosshair configuration into a Pure React chart.

EJ2-style configuration:

```tsx
<ChartComponent
  crosshair={{
    enable: true,
    lineType: "Both",
  }}
/>
```

Pure React component configuration:

```tsx
<Chart>
  <ChartCrosshair
    enable={true}
    lineType="Both"
  />
</Chart>
```

Likewise, configure crosshair axis labels with the Pure React `ChartCrosshairTooltip` child inside the owning axis rather than using an EJ2 axis settings object.

## Common errors

### Wrong component owner

Incorrect:

```tsx
<ChartSeries>
  <ChartCrosshair enable={true} />
</ChartSeries>
```

Correct:

```tsx
<Chart>
  <ChartCrosshair enable={true} />
</Chart>
```

### Using `visible`

Incorrect:

```tsx
<ChartCrosshair visible={true} />
```

Correct:

```tsx
<ChartCrosshair enable={true} />
```

### Unsupported line type

Incorrect:

```tsx
<ChartCrosshair lineType="None" />
```

Correct:

```tsx
<ChartCrosshair lineType="Both" />
```

### Wrong tooltip hierarchy

Incorrect:

```tsx
<Chart>
  <ChartCrosshairTooltip enable={true} />
</Chart>
```

Correct:

```tsx
<ChartPrimaryXAxis>
  <ChartCrosshairTooltip enable={true} />
</ChartPrimaryXAxis>
```

### Confusing tooltip types

Do not use `ChartTooltip` when the requirement is to display axis values at the crosshair intersection.

### Incorrect formatter arguments

Do not assume the formatter receives a point object:

```tsx
// Incorrect
formatter={(args) => args.point.y}
```

Use the documented value and text arguments:

```tsx
formatter={(value, text) => `${text} units`}
```

## Validation checklist

Before returning a crosshair implementation:

1. Import `ChartCrosshair` from `@syncfusion/react-charts`.
2. Place `ChartCrosshair` directly inside `Chart`.
3. Set `enable={true}` when the feature is requested.
4. Use only `Both`, `Vertical`, or `Horizontal` for `lineType`.
5. Configure line appearance through `lineStyle`.
6. Keep line opacity between `0` and `1`.
7. Use `snap` deliberately according to the precision requirement.
8. Use `highlightCategory` only with a Category axis.
9. Import `ChartCrosshairTooltip` when axis labels are required.
10. Place `ChartCrosshairTooltip` inside each owning axis.
11. Use the formatter signature `(value, text) => string | boolean`.
12. Use `fontSize`, not `size`, inside `textStyle`.
13. Keep regular point tooltips separate from crosshair axis tooltips.
14. Do not mix EJ2 object-style configuration with Pure React child components.
15. Ensure every imported symbol is used.
16. Emit valid, unescaped TSX.
