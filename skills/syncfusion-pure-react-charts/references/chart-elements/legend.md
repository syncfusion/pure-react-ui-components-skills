# Legend Reference

Use `ChartLegend` to identify chart series, points, ranges, or gradients and to control legend-item interaction, position, layout, appearance, and accessibility.

## Required component hierarchy

Place `ChartLegend` directly inside `Chart`. Place every Cartesian series inside `ChartSeriesCollection` and provide a `name` for each series that needs a meaningful legend entry.

```tsx
import {
  Chart,
  ChartLegend,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

<Chart>
  <ChartLegend visible={true} />

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={salesData}
      xField="month"
      yField="sales"
      type="Column"
      name="Sales"
    />
    <ChartSeries
      dataSource={revenueData}
      xField="month"
      yField="revenue"
      type="Line"
      name="Revenue"
    />
  </ChartSeriesCollection>
</Chart>
```

Do not place `ChartLegend` inside `ChartSeries`, `ChartSeriesCollection`, or an axis.

## Core properties

The official `ChartLegendProps` API documents:

- `visible: boolean`, default `true`
- `position: "Auto" | "Top" | "Bottom" | "Left" | "Right" | "Custom"`, default `"Auto"`
- `align`: horizontal or vertical alignment according to legend position; default `"Center"`
- `location: { x, y }`, default `{ x: 0, y: 0 }`
- `width: string`, default `""`
- `height: string`, default `""`
- `background: string`, default `"transparent"`
- `border: { color, width, dashArray }`, default `{ width: 1, color: "", dashArray: "" }`
- `containerPadding: { left, right, top, bottom }`, default `0` on every side
- `padding: number`, default `8`
- `margin: { left, right, top, bottom }`, default `0` on every side
- `itemPadding: number | null`, default `null`
- `opacity: number`, default `1`
- `textStyle`: legend-item font configuration
- `shapeWidth: number`, default `10`
- `shapeHeight: number`, default `10`
- `shapePadding: number`, default `8`
- `fixedWidth: boolean`, default `false`
- `maxLabelWidth: number | null`, default `null`
- `enablePages: boolean`, default `true`
- `mode: "Series" | "Point" | "Range" | "Gradient"`, default `"Series"`
- `toggleVisibility: boolean`, default `true`
- `reverse: boolean`, default `false`
- `inversed: boolean`, default `false`
- `title: string`, default `""`
- `titleAlign: "Left" | "Center" | "Right"`, default `"Center"`
- `titleStyle`: title font configuration
- `titleOverflow: "Wrap" | "Trim" | "None"`, default `"Wrap"`
- `maxTitleWidth: number`, default `100`
- `accessibility`: legend accessibility configuration; default includes `tabIndex: 0` and `focusable: true`

Use only properties owned by `ChartLegend`. Series-level legend properties such as `name` and `legendShape` belong to `ChartSeries`.

## Basic legend

The legend is visible by default. Set `visible={false}` only when the user explicitly wants to hide it.

```tsx
<ChartLegend visible={true} />
```

Provide a name for each series:

```tsx
<ChartSeries
  dataSource={data}
  xField="month"
  yField="sales"
  type="Line"
  name="Sales"
/>
```

Do not rely on unnamed series when meaningful legend text is required.

## Position

Use exact `position` values:

- `Auto`: choose a position from the available layout
- `Top`: horizontal legend above the chart
- `Bottom`: horizontal legend below the chart
- `Left`: vertical legend to the left
- `Right`: vertical legend to the right
- `Custom`: use explicit `location` coordinates

```tsx
<ChartLegend
  visible={true}
  position="Bottom"
/>
```

Use `Auto` when no specific position is requested.

## Alignment

The valid `align` values depend on the selected position.

For horizontal positions `Top`, `Bottom`, and `Auto`:

- `Left`
- `Center`
- `Right`

```tsx
<ChartLegend
  position="Bottom"
  align="Left"
/>
```

For vertical positions `Left` and `Right`:

- `Top`
- `Center`
- `Bottom`

```tsx
<ChartLegend
  position="Right"
  align="Top"
/>
```

Do not use `Near` or `Far`. Those values are not part of the verified Pure React legend-alignment contract.

Incorrect:

```tsx
<ChartLegend
  position="Right"
  align="Near"
/>
```

Correct:

```tsx
<ChartLegend
  position="Right"
  align="Top"
/>
```

If the alignment is invalid for the current legend orientation, the component falls back to center alignment.

## Custom position

Set `position="Custom"` and provide `location`.

```tsx
<ChartLegend
  visible={true}
  position="Custom"
  location={{
    x: 180,
    y: 40,
  }}
/>
```

`location` has no custom-position effect unless `position="Custom"` is set.

Use custom coordinates only when the chart container has predictable dimensions. Verify that the legend does not cover plot content, titles, or controls.

## Width and height

Use string values for legend dimensions.

```tsx
<ChartLegend
  visible={true}
  width="240px"
  height="120px"
/>
```

The documented defaults are empty strings. Dimensions are described as pixel-based legend-area sizes. Do not pass numeric values unless the current public TypeScript API changes the property type.

Width and height constrain the legend area. They do not directly define a row or column count.

## Paging overflowed items

`enablePages` controls navigation when all legend items do not fit in the available legend area. The documented default is `true`.

```tsx
<ChartLegend
  visible={true}
  width="240px"
  height="100px"
  enablePages={true}
/>
```

Do not claim that `width` and `height` directly create a fixed number of rows or columns. Use them to constrain the legend, and let paging handle overflow when enabled.

## Legend mode

Use `mode` to choose how legend items are generated.

- `Series`: one item for each series
- `Point`: one item for each unique data point
- `Range`: items from range color mapping
- `Gradient`: one linear gradient bar from range color mapping

```tsx
<ChartLegend
  visible={true}
  mode="Series"
/>
```

`Series` is the documented default.

Use `Point`, `Range`, or `Gradient` only when the chart configuration and requested visualization support those modes.

## Toggle series visibility

`toggleVisibility` controls the default click behavior for legend items. Its documented default is `true`.

```tsx
<ChartLegend
  visible={true}
  toggleVisibility={true}
/>
```

Set `toggleVisibility={false}` when the legend must remain informational and clicks must not hide or show series.

```tsx
<ChartLegend
  visible={true}
  toggleVisibility={false}
/>
```

Do not implement custom state solely to reproduce the built-in show/hide behavior when `toggleVisibility` already satisfies the request.

## Legend click event

The legend click handler belongs to the root `Chart`, not `ChartLegend`.

```tsx
const handleLegendClick = (args: LegendClickProps): string => {
  return args.series?.name ?? "";
};

<Chart legendClick={handleLegendClick}>
  <ChartLegend visible={true} />
</Chart>
```

Use the exact event prop and exported argument type documented by the current Pure React `ChartProps` API. Do not assume EJ2-style event argument fields or names.

In particular, do not automatically generate:

```tsx
<Chart onLegendClick={handleLegendClick}>
```

unless `onLegendClick` is confirmed by the current Pure React API. The Pure React package may expose event names without the `on` prefix.

When handling the event, inspect only fields documented by the exported event type. Do not assume `args.series.name`, `args.seriesName`, `preventDefault`, or cancellation behavior without verification.

## Background, border, and opacity

```tsx
<ChartLegend
  visible={true}
  background="#FFFFFF"
  border={{
    color: "#D0D0D0",
    width: 1,
    dashArray: "",
  }}
  opacity={1}
/>
```

- `background` accepts a valid CSS color.
- `opacity` ranges from `0` through `1`.
- `border.dashArray` accepts a comma-separated SVG-style dash pattern.

The documented default background is `transparent`.

## Padding and margin

Use `containerPadding` for spacing between the legend border and its content.

```tsx
<ChartLegend
  containerPadding={{
    left: 12,
    right: 12,
    top: 8,
    bottom: 8,
  }}
/>
```

Use `padding` for internal spacing between the legend border and content elements. Its documented default is `8`.

```tsx
<ChartLegend padding={10} />
```

Use `margin` for external spacing around the legend.

```tsx
<ChartLegend
  margin={{
    left: 0,
    right: 0,
    top: 8,
    bottom: 8,
  }}
/>
```

Use `itemPadding` to adjust spacing between adjacent legend items.

```tsx
<ChartLegend itemPadding={12} />
```

Do not treat these properties as interchangeable.

## Legend text style

Use exact `ChartFontProps` names inside `textStyle`.

```tsx
<ChartLegend
  textStyle={{
    fontFamily: "Arial",
    fontSize: "12px",
    fontStyle: "Normal",
    fontWeight: "Normal",
    color: "#333333",
    opacity: 1,
  }}
/>
```

Use `fontSize`, not `size`. Use a CSS-size string such as `"12px"`.

## Legend shapes

Set shape dimensions on `ChartLegend`:

```tsx
<ChartLegend
  shapeWidth={12}
  shapeHeight={12}
  shapePadding={8}
/>
```

Select the legend symbol on the owning series with `legendShape`.

```tsx
<ChartSeries
  name="Sales"
  legendShape="Diamond"
/>
```

Verified legend-shape values include:

- `SeriesType`
- `Circle`
- `Rectangle`
- `Cross`
- `Diamond`
- `HorizontalLine`
- `VerticalLine`
- `Triangle`
- `Pentagon`
- `InvertedTriangle`
- `Image`

Use `SeriesType` when the legend symbol should follow the series type.

## Fixed-width items and long labels

Set `fixedWidth={true}` to give all legend items the same width.

```tsx
<ChartLegend
  fixedWidth={true}
  maxLabelWidth={120}
/>
```

`maxLabelWidth` limits individual legend-label width in pixels. Its documented default is `null`.

Use fixed-width items when a uniform grid-like legend layout improves scanning. Do not enable it automatically for a small legend.

## Reverse and inversed

`reverse` and `inversed` perform different operations.

### Reverse item sequence

Set `reverse={true}` to show legend items in reverse order.

```tsx
<ChartLegend reverse={true} />
```

### Reverse shape and text order

Set `inversed={true}` to place text before the shape within each legend item.

```tsx
<ChartLegend inversed={true} />
```

Do not describe `inversed` as reversing the series sequence. Use `reverse` for sequence order.

## Legend title

Use `title` for the heading displayed above the legend items.

```tsx
<ChartLegend
  title="Metrics"
  titleAlign="Center"
  maxTitleWidth={140}
  titleOverflow="Wrap"
  titleStyle={{
    fontFamily: "Arial",
    fontSize: "13px",
    fontStyle: "Normal",
    fontWeight: "Bold",
    color: "#222222",
    opacity: 1,
  }}
/>
```

Verified title options:

- `titleAlign`: `Left`, `Center`, or `Right`
- `titleOverflow`: `Wrap`, `Trim`, or `None`
- `maxTitleWidth`: numeric width; default `100`

## Accessibility

Use `accessibility` to control legend focus behavior and accessible metadata supported by `ChartAccessibilityProps`.

```tsx
<ChartLegend
  accessibility={{
    ariaLabel: "Chart legend",
    role: "group",
    focusable: true,
    tabIndex: 0,
  }}
/>
```

The legend API documents default accessibility values including `tabIndex: 0` and `focusable: true`.

Do not claim complete screen-reader behavior or compliance guarantees based only on enabling this object. Validate the resulting chart with the application's accessibility-testing process.

## Complete example

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

const data = [
  { month: "Jan", sales: 35, revenue: 42 },
  { month: "Feb", sales: 42, revenue: 48 },
  { month: "Mar", sales: 38, revenue: 45 },
  { month: "Apr", sales: 51, revenue: 59 },
];

export default function LegendChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis valueType="Category">
        <ChartAxisTitle text="Month" />
        <ChartAxisLabel edgeLabelPlacement="Shift" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double">
        <ChartAxisTitle text="Value" />
        <ChartAxisLabel format="{value}" />
      </ChartPrimaryYAxis>

      <ChartLegend
        visible={true}
        position="Bottom"
        align="Center"
        title="Metrics"
        titleAlign="Center"
        titleOverflow="Wrap"
        maxTitleWidth={120}
        background="#FFFFFF"
        border={{
          color: "#D0D0D0",
          width: 1,
          dashArray: "",
        }}
        containerPadding={{
          left: 8,
          right: 8,
          top: 6,
          bottom: 6,
        }}
        padding={8}
        itemPadding={12}
        shapeWidth={12}
        shapeHeight={12}
        shapePadding={8}
        textStyle={{
          fontFamily: "Arial",
          fontSize: "12px",
          fontStyle: "Normal",
          fontWeight: "Normal",
          color: "#333333",
          opacity: 1,
        }}
        titleStyle={{
          fontFamily: "Arial",
          fontSize: "13px",
          fontStyle: "Normal",
          fontWeight: "Bold",
          color: "#222222",
          opacity: 1,
        }}
        toggleVisibility={true}
        enablePages={true}
        opacity={1}
      />

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="sales"
          type="Column"
          name="Sales"
          legendShape="Rectangle"
        />
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="revenue"
          type="Line"
          name="Revenue"
          legendShape="HorizontalLine"
          width={2}
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Common errors

### Wrong hierarchy

Incorrect:

```tsx
<ChartSeries>
  <ChartLegend visible={true} />
</ChartSeries>
```

Correct:

```tsx
<Chart>
  <ChartLegend visible={true} />
  <ChartSeriesCollection>
    <ChartSeries name="Sales" />
  </ChartSeriesCollection>
</Chart>
```

### Unsupported alignment

Incorrect:

```tsx
<ChartLegend align="Near" />
```

Correct for a horizontal legend:

```tsx
<ChartLegend align="Left" />
```

Correct for a vertical legend:

```tsx
<ChartLegend position="Right" align="Top" />
```

### Using `alignment`

Incorrect:

```tsx
<ChartLegend alignment="Center" />
```

Correct:

```tsx
<ChartLegend align="Center" />
```

### Treating size as a row or column count

`width` and `height` constrain the legend area. They do not set the number of legend rows or columns.

### Confusing `reverse` and `inversed`

- Use `reverse` to reverse legend-item sequence.
- Use `inversed` to place item text before its shape.

### Guessing event names and fields

Do not reuse `onLegendClick`, `args.series.name`, or EJ2 event patterns without verifying the current Pure React `ChartProps` event and argument type.

## Validation checklist

Before returning a legend implementation:

1. Import `ChartLegend` from `@syncfusion/react-charts`.
2. Place `ChartLegend` directly inside `Chart`.
3. Give every series a meaningful `name` when it needs a legend item.
4. Use only `Auto`, `Top`, `Bottom`, `Left`, `Right`, or `Custom` for `position`.
5. Use `Left`, `Center`, or `Right` for horizontal legend alignment.
6. Use `Top`, `Center`, or `Bottom` for vertical legend alignment.
7. Do not use `Near`, `Far`, or `alignment`.
8. Use `location` only with `position="Custom"`.
9. Use string values for `width` and `height`.
10. Use `containerPadding`, `padding`, `margin`, and `itemPadding` according to their separate roles.
11. Use `fontSize`, not `size`, in `textStyle` and `titleStyle`.
12. Use `toggleVisibility` for built-in show/hide behavior.
13. Use `reverse` for item sequence and `inversed` for shape/text order.
14. Use `legendShape` on `ChartSeries`, not `ChartLegend`.
15. Verify the root Chart legend-click event name and event fields before generating a handler.
16. Ensure every imported symbol is used.
17. Emit valid, unescaped TSX.
