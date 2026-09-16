# Tooltips Reference

Use `ChartTooltip` to display contextual information when a user hovers over or taps a chart point. Place it directly inside `Chart`.

## Component hierarchy

```tsx
import {
  Chart,
  ChartSeries,
  ChartSeriesCollection,
  ChartTooltip,
} from "@syncfusion/react-charts";

<Chart>
  <ChartTooltip enable={true} />

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="x"
      yField="y"
      type="Column"
    />
  </ChartSeriesCollection>
</Chart>
```

Do not place `ChartTooltip` inside a series, axis, or series collection.

## Tooltip properties

The Pure React `ChartTooltipProps` API documents:

- `enable`: enables tooltips; default `false`
- `shared`: combines points with the same X value; default `false`
- `format`: formats tooltip content; default `''`
- `formatter`: transforms generated tooltip text; default `null`
- `template`: renders custom JSX; default `null`
- `headerText`: customizes header text; default `''`
- `showHeaderLine`: shows the header separator; default `false`
- `showMarker`: shows series markers; default `true`
- `showNearestPoint`: includes the nearest point in a shared tooltip; default `true`
- `showNearestTooltip`: displays the nearest tooltip for supported series; default `true`
- `followPointer`: tooltip tracks the pointer position; default `false`
- `distance`: pixel gap between tooltip and point when not following the pointer; default `0`
- `fill`, `border`, `textStyle`, and `opacity`: appearance settings
- `location`: fixed tooltip location; default `{ x: 0, y: 0 }`
- `duration`: transition duration; default `300`
- `enableAnimation`: enables transitions; default `true`
- `fadeOutDuration`: fade duration; default `1000`
- `fadeOutMode`: `Move` or `Click`; default `Move`

## Basic tooltip

```tsx
<ChartTooltip enable={true} />
```

Tooltips are disabled by default. Once enabled, the chart displays point information on hover or supported touch interaction.

A series can disable its own tooltip while chart tooltips remain enabled:

```tsx
<ChartSeries
  dataSource={data}
  xField="x"
  yField="y"
  type="Line"
  enableTooltip={false}
/>
```

`ChartSeries.enableTooltip` defaults to `true`.

## Shared tooltip

Set `shared={true}` to display points from multiple series that share the same X value.

```tsx
<ChartTooltip
  enable={true}
  shared={true}
/>
```

Shared tooltips are useful for multi-series comparisons.

```tsx
<ChartSeriesCollection>
  <ChartSeries
    dataSource={data}
    xField="month"
    yField="sales"
    type="Line"
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

Keep X values compatible across the series so shared grouping is meaningful.

## Tooltip format

Use `format` with documented placeholders:

- `${point.x}`: point X value
- `${point.y}`: point Y value
- `${series.name}`: series name

```tsx
<ChartTooltip
  enable={true}
  format="${series.name}: ${point.x} - ${point.y}"
/>
```

Because JavaScript template literals also interpret `${...}`, use a quoted JSX string as shown above. If the format is stored in a variable, escape or construct it so the placeholders reach the chart unchanged.

## Series-specific format

Use `ChartSeries.tooltipFormat` when one series requires its own tooltip format.

```tsx
<ChartSeries
  dataSource={data}
  xField="month"
  yField="sales"
  type="Column"
  name="Sales"
  tooltipFormat="${series.name}: ${point.y}"
/>
```

Use `ChartTooltip.format` for a chart-wide format and `ChartSeries.tooltipFormat` for series-specific content.

## Tooltip field mapping

Map custom data through `ChartSeries.tooltipField`.

```tsx
const data = [
  {
    month: "Jan",
    sales: 100,
    note: "Quarter opening",
  },
  {
    month: "Feb",
    sales: 120,
    note: "Campaign period",
  },
];

<Chart>
  <ChartTooltip
    enable={true}
    format="${point.tooltip}"
  />

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="month"
      yField="sales"
      tooltipField="note"
      type="Column"
    />
  </ChartSeriesCollection>
</Chart>
```

The mapped value is stored in the point's tooltip field and can be accessed through tooltip formatting or template context.

Ensure every relevant record contains the mapped field.

## Formatter

Use `formatter` to transform the generated tooltip text.

The documented signature is:

```tsx
(text: string | string[]) => string | string[] | boolean
```

```tsx
const formatTooltip = (
  text: string | string[],
): string | string[] => {
  if (Array.isArray(text)) {
    return text.map((item) => `${item} units`);
  }

  return `${text} units`;
};

<ChartTooltip
  enable={true}
  formatter={formatTooltip}
/>
```

The input can be an array for shared tooltip content. Do not assume the formatter receives a point or series object.

Incorrect:

```tsx
formatter={(args) => args.point.y}
```

## Custom JSX template

Use `template` when the tooltip requires custom React content.

```tsx
import type {
  ChartTooltipTemplateProps,
} from "@syncfusion/react-charts";

const tooltipTemplate = (
  props: ChartTooltipTemplateProps,
) => (
  <div
    style={{
      padding: "8px 10px",
      background: "#263238",
      color: "#FFFFFF",
      borderRadius: "4px",
    }}
  >
    <strong>{props.x}</strong>
    <div>Value: {props.y}</div>
    {props.tooltip && <div>{props.tooltip}</div>}
  </div>
);

<ChartTooltip
  enable={true}
  template={tooltipTemplate}
/>
```

`ChartTooltipTemplateProps` documents:

- `x`
- `y`
- `tooltip`
- `seriesIndex`
- `pointIndex`

Keep templates compact and avoid interactive elements that interfere with chart pointer behavior.

## Tooltip appearance

Use `fill`, `border`, `textStyle`, and `opacity`.

```tsx
<ChartTooltip
  enable={true}
  fill="#2C3E50"
  border={{
    color: "#E74C3C",
    width: 2,
    dashArray: "",
  }}
  textStyle={{
    color: "#ECF0F1",
    fontFamily: "Arial",
    fontSize: "12px",
    fontWeight: "Bold",
    fontStyle: "Normal",
    opacity: 1,
  }}
  opacity={0.9}
/>
```

`ChartBorderProps` supports `color`, `width`, and `dashArray`. Do not use an unsupported `type: "Solid"` property.

Keep both tooltip and text opacity between `0` and `1`.

## Fixed location

Use `location` to position the tooltip at fixed chart coordinates.

```tsx
<ChartTooltip
  enable={true}
  location={{ x: 100, y: 200 }}
/>
```

The location is relative to the chart. Ensure the coordinates keep the tooltip within the visible chart area.

## Nearest-point behavior

Use `showNearestTooltip` to control nearest-point tooltip display for line, area, spline, and spline-area series.

```tsx
<ChartTooltip
  enable={true}
  showNearestTooltip={true}
/>
```

Use `showNearestPoint` to include the nearest point in shared tooltip content.

```tsx
<ChartTooltip
  enable={true}
  shared={true}
  showNearestPoint={true}
/>
```

These are the verified Pure React nearest-point settings.

## Pointer-following tooltip

Use `followPointer` to make the tooltip track the pointer position instead of anchoring to the data point. It is ignored when an explicit `location` is provided. The companion `distance` property controls the pixel gap between the tooltip and the point, and applies only when `followPointer` is `false`.

```tsx
<ChartTooltip
  enable={true}
  followPointer={true}
/>
```

```tsx
<ChartTooltip
  enable={true}
  followPointer={false}
  distance={8}
/>
```

## Tooltip marker

Use `showMarker` to display or hide colored series markers inside the tooltip.

```tsx
<ChartTooltip
  enable={true}
  shared={true}
  showMarker={true}
/>
```

Markers help distinguish series in shared tooltip content.

## Tooltip header

Use `headerText` to customize the header and `showHeaderLine` to control the separator.

```tsx
<ChartTooltip
  enable={true}
  headerText="${point.x}"
  showHeaderLine={true}
  format="${series.name}: ${point.y}"
/>
```

The default header displays the series name. `showHeaderLine` defaults to `false`.

Use plain text unless the current API explicitly documents HTML interpretation for `headerText`. Do not rely on `<b>` tags being rendered as HTML.

## Animation and fade-out

```tsx
<ChartTooltip
  enable={true}
  enableAnimation={true}
  duration={300}
  fadeOutDuration={1000}
  fadeOutMode="Move"
/>
```

Use exact `fadeOutMode` values:

- `Move`: fades after the pointer moves away
- `Click`: removes the tooltip when the chart is clicked

Use non-negative millisecond values for timing properties.

## Complete example

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartMarker,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
  ChartTooltip,
} from "@syncfusion/react-charts";

const data = [
  { month: "Jan", sales: 35, revenue: 42, note: "Opening month" },
  { month: "Feb", sales: 42, revenue: 48, note: "Campaign launch" },
  { month: "Mar", sales: 38, revenue: 45, note: "Quarter close" },
  { month: "Apr", sales: 51, revenue: 59, note: "New quarter" },
];

export default function TooltipChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis valueType="Category">
        <ChartAxisTitle text="Month" />
        <ChartAxisLabel edgeLabelPlacement="Shift" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double" minimum={0}>
        <ChartAxisTitle text="Value" />
        <ChartAxisLabel format="{value}" />
      </ChartPrimaryYAxis>

      <ChartTooltip
        enable={true}
        shared={true}
        format="${series.name}: ${point.y}"
        headerText="${point.x}"
        showHeaderLine={true}
        showMarker={true}
        showNearestPoint={true}
        fill="#263238"
        border={{
          color: "#455A64",
          width: 1,
          dashArray: "",
        }}
        textStyle={{
          color: "#FFFFFF",
          fontFamily: "Arial",
          fontSize: "12px",
          fontWeight: "Normal",
          fontStyle: "Normal",
          opacity: 1,
        }}
        opacity={0.95}
        enableAnimation={true}
        duration={300}
        fadeOutDuration={1000}
        fadeOutMode="Move"
      />

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="sales"
          tooltipField="note"
          type="Line"
          name="Sales"
          width={2}
        >
          <ChartMarker
            visible={true}
            shape="Circle"
            width={7}
            height={7}
          />
        </ChartSeries>

        <ChartSeries
          dataSource={data}
          xField="month"
          yField="revenue"
          tooltipField="note"
          type="Line"
          name="Revenue"
          width={2}
        >
          <ChartMarker
            visible={true}
            shape="Diamond"
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

Do not use the EJ2 root `tooltip={{ ... }}` object in a Pure React chart.

EJ2-style configuration:

```tsx
<ChartComponent
  tooltip={{
    enable: true,
    shared: true,
    format: "${point.x}: ${point.y}",
  }}
/>
```

Pure React component configuration:

```tsx
<Chart>
  <ChartTooltip
    enable={true}
    shared={true}
    format="${point.x}: ${point.y}"
  />
</Chart>
```

Keep tooltip configuration in the `ChartTooltip` child component.

## Common errors

### Wrong hierarchy

Incorrect:

```tsx
<ChartSeries>
  <ChartTooltip enable={true} />
</ChartSeries>
```

Correct:

```tsx
<Chart>
  <ChartTooltip enable={true} />
</Chart>
```

### Claiming tooltip is enabled by default

Incorrect. `ChartTooltip.enable` defaults to `false`.

### Unsupported border property

Incorrect:

```tsx
border={{
  color: "#E74C3C",
  width: 2,
  type: "Solid",
}}
```

Use `color`, `width`, and `dashArray`.

### Unsupported follow-pointer property

Do not use `followPointer`. Use the documented nearest-point and fixed-location options.

### Wrong formatter context

Do not expect the formatter to receive point fields. Use `template` when direct access to `x`, `y`, indexes, or mapped tooltip data is required.

### Unverified header HTML

Do not depend on HTML markup in `headerText` unless the current API explicitly guarantees HTML rendering.

## Validation checklist

Before returning a tooltip implementation:

1. Import `ChartTooltip` from `@syncfusion/react-charts`.
2. Place it directly inside `Chart`.
3. Set `enable={true}` because tooltips are disabled by default.
4. Use `shared` only when series have compatible X values.
5. Use documented format placeholders.
6. Use `tooltipField` and `tooltipFormat` on the owning series when needed.
7. Use the formatter signature `(text) => string | string[] | boolean`.
8. Handle both string and string-array formatter input.
9. Use `ChartTooltipTemplateProps` for custom JSX templates.
10. Use only `color`, `width`, and `dashArray` in `border`.
11. Use `fontSize`, not `size`, in `textStyle`.
12. Keep opacity values between `0` and `1`.
13. Use `showNearestTooltip` and `showNearestPoint` instead of `followPointer`.
14. Use only `Move` or `Click` for `fadeOutMode`.
15. Keep `ChartCrosshairTooltip` separate from regular point tooltips.
16. Do not mix EJ2 object-style configuration with Pure React components.
17. Ensure every imported symbol is used.
18. Emit valid, unescaped TSX.
