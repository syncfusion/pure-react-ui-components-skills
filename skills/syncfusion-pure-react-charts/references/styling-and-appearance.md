# Styling and Appearance Reference

Use this reference to customize the Pure React Chart container, plot area, titles, axes, palettes, dimensions, themes, legends, and series. Prefer documented component props and child components over EJ2 configuration objects, undocumented CSS selectors, or handcrafted SVG.

## Table of contents

1. [Chart container](#chart-container)
2. [Chart area](#chart-area)
3. [Titles and subtitles](#titles-and-subtitles)
4. [Axis styling](#axis-styling)
5. [Color palettes](#color-palettes)
6. [Dimensions and responsiveness](#dimensions-and-responsiveness)
7. [Themes](#themes)
8. [Legend styling](#legend-styling)
9. [Series styling](#series-styling)
10. [Advanced patterns](#advanced-patterns)
11. [Validation checklist](#validation-checklist)

## Chart container

Use root `Chart` props for the chart background, background image, border, internal margin, dimensions, theme, palette, and focus outline.

```tsx
<Chart
  width="100%"
  height="420px"
  background="#FFFFFF"
  border={{
    color: "#D1D1D1",
    width: 1,
    dashArray: "",
  }}
  margin={{
    top: 16,
    right: 16,
    bottom: 16,
    left: 16,
  }}
  focusOutline={{
    color: "#005FCC",
    width: 2,
    offset: 2,
  }}
>
  {/* chart children */}
</Chart>
```

Use `dashArray` for dashed borders. Do not use an unsupported `type: "Solid"` property.

Apply card shadows and rounded corners to a wrapper element rather than assuming the `Chart` root accepts a general React `style` prop.

```tsx
<section className="chart-card">
  <Chart width="100%" height="420px">
    {/* chart children */}
  </Chart>
</section>
```

```css
.chart-card {
  min-width: 0;
  padding: 16px;
  border: 1px solid #d1d1d1;
  border-radius: 8px;
  background: #ffffff;
  box-shadow: 0 2px 8px rgb(0 0 0 / 10%);
}
```

## Chart area

Place `ChartArea` directly inside `Chart` to style the plotting region.

```tsx
<Chart>
  <ChartArea
    background="#FAFAFA"
    opacity={1}
    border={{
      color: "#E5E5E5",
      width: 1,
      dashArray: "",
    }}
    margin={{
      top: 8,
      right: 8,
      bottom: 8,
      left: 8,
    }}
  />

  <ChartSeriesCollection>
    {/* series */}
  </ChartSeriesCollection>
</Chart>
```

`ChartArea` supports `background`, `backgroundImage`, `border`, `margin`, `opacity`, and a pixel-based `width`. Prefer a plain color unless an image is necessary and remains readable behind the data.

## Titles and subtitles

Use `ChartTitle` and `ChartSubtitle` as direct children of `Chart`. Keep the visible chart title concise and put long explanations in surrounding HTML.

```tsx
<Chart>
  <ChartTitle
    text="Quarterly sales"
    fontSize="18px"
    fontFamily="Arial"
    fontStyle="Normal"
    fontWeight="Bold"
    color="#242424"
    align="Center"
    overflow="Wrap"
  />

  <ChartSubtitle
    text="Fiscal year 2026"
    fontSize="12px"
    color="#616161"
    align="Center"
    overflow="Wrap"
  />
</Chart>
```

Use CSS-style strings such as `"18px"` for documented text sizes. Do not add title position, background, border, object padding, or rotation unless the installed API explicitly documents those properties.

## Axis styling

Keep axis-specific child components inside the axis they customize.

```tsx
<ChartPrimaryXAxis
  valueType="Category"
  lineStyle={{
    color: "#616161",
    width: 1,
    dashArray: "",
  }}
>
  <ChartAxisTitle
    text="Month"
    fontSize="13px"
    fontWeight="Bold"
    color="#242424"
    padding={8}
  />

  <ChartAxisLabel
    fontFamily="Arial"
    fontSize="12px"
    fontWeight="Normal"
    color="#424242"
    edgeLabelPlacement="Shift"
    intersectAction="Trim"
    maxLabelWidth={80}
  />

  <ChartMajorTickLines
    width={1}
    height={6}
    color="#616161"
  />
</ChartPrimaryXAxis>
```

Use `ChartAxisLabel.format`, `formatter`, `skeleton`, or `rotationAngle` rather than EJ2-style `labelFormat`, `labelStyle`, or `labelRotationAngle` props on the axis.

```tsx
<ChartPrimaryXAxis
  valueType="DateTime"
  interval={1}
  intervalType="Months"
>
  <ChartAxisLabel
    skeleton="MMM y"
    rotationAngle={-45}
  />
</ChartPrimaryXAxis>
```

### Grid lines

```tsx
<ChartPrimaryYAxis valueType="Double">
  <ChartMajorGridLines
    visible={true}
    width={1}
    color="#E5E5E5"
    dashArray="4,2"
  />

  <ChartMinorGridLines
    visible={true}
    width={0.5}
    color="#F2F2F2"
    dashArray=""
  />
</ChartPrimaryYAxis>
```

Use minor grids and ticks only when the extra subdivisions help users read the scale.

### Additional axes

Define additional axes with `ChartAxes` and a named `ChartAxis`. Do not invent `ChartSecondaryYAxis`.

```tsx
<ChartAxes>
  <ChartAxis
    name="revenueAxis"
    valueType="Double"
    opposedPosition={true}
    lineStyle={{
      color: "#A4262C",
      width: 1,
      dashArray: "",
    }}
  >
    <ChartAxisTitle text="Revenue" color="#A4262C" />
    <ChartAxisLabel color="#A4262C" />
  </ChartAxis>
</ChartAxes>

<ChartSeries
  dataSource={data}
  xField="month"
  yField="revenue"
  yAxisName="revenueAxis"
  type="Line"
/>
```

`inverted` reverses scale direction. It is not a styling property or an RTL switch.

## Color palettes

Use `Chart.palettes` to assign series colors sequentially.

```tsx
<Chart
  palettes={[
    "#005A9C",
    "#107C10",
    "#A4262C",
    "#5C2D91",
  ]}
>
  {/* chart children */}
</Chart>
```

Override a palette color through the series `fill` when a series has a fixed semantic color.

```tsx
<ChartSeries
  dataSource={actualData}
  xField="month"
  yField="value"
  type="Column"
  name="Actual"
  fill="#005A9C"
/>
```

Do not use pure red and green alone to convey bad and good states. Add labels, shapes, patterns, or explanatory text.

### Point-level colors

Map a color field directly when the data owns the color.

```tsx
const data = [
  { category: "A", value: 42, color: "#107C10" },
  { category: "B", value: 31, color: "#F7630C" },
  { category: "C", value: 18, color: "#A4262C" },
];

<ChartSeries
  dataSource={data}
  xField="category"
  yField="value"
  colorField="color"
  type="Column"
/>
```

For computed colors, use the root `pointRender` callback. It receives `PointRenderProps` and returns a color.

```tsx
import type { PointRenderProps } from "@syncfusion/react-charts";

const colorPoint = (args: PointRenderProps): string => {
  const value = Number(args.yValue);

  if (value >= 100) {
    return "#107C10";
  }

  if (value < 50) {
    return "#A4262C";
  }

  return "#F7630C";
};

<Chart pointRender={colorPoint}>
  {/* chart children */}
</Chart>
```

Do not attach `onPointRender` to `ChartSeries`, read an undocumented `args.data`, or mutate `args.fill`.

## Dimensions and responsiveness

`width` and `height` are strings. Use pixel or percentage strings rather than numbers.

```tsx
<Chart width="800px" height="500px">
  {/* chart children */}
</Chart>
```

For responsive width and percentage height, give the parent a resolved height.

```tsx
<div className="chart-canvas">
  <Chart width="100%" height="100%">
    {/* chart children */}
  </Chart>
</div>
```

```css
.chart-canvas {
  width: 100%;
  height: 420px;
  min-width: 0;
}

@media (max-width: 640px) {
  .chart-canvas {
    height: 340px;
  }
}
```

Prefer CSS media queries or a reusable media-query hook. Do not add a new third-party breakpoint package to a simple sample unless the project already uses it.

## Themes

Use exact themes documented by the installed Pure React Chart API. Current root themes include:

- `Material`
- `MaterialDark`
- `Tailwind`
- `TailwindDark`
- `Bootstrap`
- `BootstrapDark`

```tsx
<Chart theme={darkMode ? "TailwindDark" : "Tailwind"}>
  {/* chart children */}
</Chart>
```

Do not assume EJ2 theme names such as `Fluent`, `Bootstrap4`, or `HighContrast` are valid for the installed Pure React package.

Import one compatible Syncfusion theme stylesheet at application level, and keep chart backgrounds, text, gridlines, tooltips, focus outlines, and surrounding cards aligned with the same theme state.

## Legend styling

Place `ChartLegend` directly inside `Chart`.

```tsx
<ChartLegend
  visible={true}
  position="Bottom"
  align="Center"
  background="#FFFFFF"
  border={{
    color: "#D1D1D1",
    width: 1,
    dashArray: "",
  }}
  padding={8}
  itemPadding={12}
  shapeWidth={12}
  shapeHeight={12}
  shapePadding={8}
  maxLabelWidth={160}
  enablePages={true}
  textStyle={{
    fontFamily: "Arial",
    fontSize: "12px",
    fontWeight: "Normal",
    color: "#242424",
  }}
/>
```

Use `align`, not `alignment`. Valid alignment depends on legend position. The API supports paging and sizing controls, but it does not document a `columnCount` property.

Use bottom legends for many responsive dashboard layouts. Side legends require enough horizontal space.

## Series styling

Use series props relevant to the selected type.

```tsx
<ChartSeries
  dataSource={data}
  xField="month"
  yField="sales"
  type="Line"
  name="Sales"
  fill="#005A9C"
  opacity={0.9}
  width={2}
  dashArray="6,3"
  border={{
    color: "#003E6B",
    width: 1,
  }}
  animation={{
    enable: true,
    duration: 600,
    delay: 0,
  }}
/>
```

Do not add an undocumented animation `option`. Use only the documented animation fields.

### Column and bar spacing

```tsx
<ChartSeries
  dataSource={data}
  xField="category"
  yField="value"
  type="Column"
  fill="#005A9C"
  columnWidth={0.65}
  columnSpacing={0.15}
  cornerRadius={{
    topLeft: 4,
    topRight: 4,
    bottomLeft: 0,
    bottomRight: 0,
  }}
/>
```

Both `columnWidth` and `columnSpacing` use values between `0` and `1`.

### Markers and data labels

```tsx
<ChartSeries
  dataSource={data}
  xField="month"
  yField="sales"
  type="Line"
>
  <ChartMarker
    visible={true}
    shape="Circle"
    width={7}
    height={7}
    fill="#005A9C"
  >
    <ChartDataLabel
      visible={true}
      position="Top"
      intersectMode="Hide"
      font={{
        color: "#242424",
        fontFamily: "Arial",
        fontSize: "12px",
        fontStyle: "Normal",
        fontWeight: "Normal",
        opacity: 1,
      }}
    />
  </ChartMarker>
</ChartSeries>
```

Avoid markers and labels on every point in a dense series.

## Advanced patterns

### Application theme tokens

Keep theme values in one typed object and pass supported values to chart props.

```tsx
const chartTokens = darkMode
  ? {
      theme: "TailwindDark" as const,
      background: "#1F1F1F",
      text: "#F5F5F5",
      grid: "#4A4A4A",
      palette: ["#75B6FF", "#6CCB75", "#FFB86B"],
    }
  : {
      theme: "Tailwind" as const,
      background: "#FFFFFF",
      text: "#242424",
      grid: "#E5E5E5",
      palette: ["#005A9C", "#107C10", "#F7630C"],
    };
```

Do not assume arbitrary CSS custom properties are consumed internally by the chart.

### Gradient and image fills

Use only gradient or image mechanisms documented and tested for the installed series type. Do not inject hidden SVG definitions and assume `fill="url(#gradient)"` will work across chart renderers, exports, themes, or server rendering.

A solid theme color is the safest default for readability and export quality.

### High contrast

Do not claim a palette is WCAG AAA merely from its hex values. Contrast depends on foreground-background pairs, text size, adjacent colors, opacity, and state.

Use high-contrast colors together with non-color cues and verify the rendered chart with automated and manual accessibility testing.

### Print styling

Apply shadows and page decorations to wrappers and remove them through print CSS when needed.

```css
@media print {
  .chart-card {
    border: 1px solid #000000;
    border-radius: 0;
    box-shadow: none;
    break-inside: avoid;
  }
}
```

### Performance

- Profile gradients, templates, shadows, labels, markers, and animations on target devices.
- Disable unnecessary animation for rapid live updates or reduced-motion users.
- Avoid universal thresholds such as disabling animation above exactly 1,000 points.
- Memoize derived theme objects only when referential stability or computation cost matters.
- Use CSS variables normally in application styling unless profiling identifies an actual bottleneck.

## CSS integration

Style surrounding layout through stable wrapper classes.

```tsx
<div className="sales-chart">
  <Chart width="100%" height="420px">
    {/* chart children */}
  </Chart>
</div>
```

```css
.sales-chart {
  min-width: 0;
  color: #242424;
}
```

Do not target undocumented internal selectors such as `.e-chart` or `.e-data-label` in Pure React samples. Internal class names may differ from EJ2 and may change. Prefer documented component props and wrapper-level CSS.

## Common errors

### Invalid border type

Incorrect:

```tsx
border={{ color: "#CCCCCC", width: 1, type: "Solid" }}
```

Correct:

```tsx
border={{ color: "#CCCCCC", width: 1, dashArray: "" }}
```

### Numeric dimensions

Incorrect:

```tsx
<Chart width={800} height={600} />
```

Correct:

```tsx
<Chart width="800px" height="600px" />
```

### EJ2 axis label objects

Incorrect:

```tsx
<ChartPrimaryXAxis
  labelFormat="{value:MMM dd}"
  labelStyle={{ fontSize: "12px" }}
  labelRotationAngle={45}
/>
```

Correct:

```tsx
<ChartPrimaryXAxis valueType="DateTime">
  <ChartAxisLabel
    skeleton="MMM dd"
    fontSize="12px"
    rotationAngle={45}
  />
</ChartPrimaryXAxis>
```

### Invented secondary axis

Use `ChartAxes` and a named `ChartAxis`, not `ChartSecondaryYAxis`.

### Wrong legend alignment

Use `align="Center"`, not `alignment="Center"` or EJ2 values such as `Near` and `Far`.

### Wrong point-render contract

Use root `pointRender` and return the desired color. Do not mutate `args.fill`.

### Undocumented global range mapping

Do not add `rangeColorMapping` to `Chart` or claim it styles pie slices. Use `colorField`, multi-colored series configuration, or the pie-chart family's documented palette and color mapping.

## Validation checklist

Before returning a styling implementation:

1. Use documented Pure React components and props.
2. Keep wrapper CSS separate from internal chart configuration.
3. Use `dashArray` for dashed chart borders.
4. Keep `ChartArea` directly inside `Chart`.
5. Use CSS-style strings for text sizes and chart dimensions.
6. Place `ChartTitle` and `ChartSubtitle` directly inside `Chart`.
7. Keep axis titles, labels, grids, and ticks inside their owning axis.
8. Use `ChartAxisLabel` instead of EJ2 `labelStyle` configuration.
9. Use `ChartAxes` and named `ChartAxis` for additional axes.
10. Use `palettes`, series `fill`, or `colorField` for color assignment.
11. Use root `pointRender` and return a color for computed point styling.
12. Use only documented theme names.
13. Give percentage-height charts a parent with a resolved height.
14. Use `ChartLegend.align`, not `alignment`.
15. Do not use undocumented legend columns.
16. Apply only series properties relevant to the selected type.
17. Avoid unsupported animation easing properties.
18. Keep labels, markers, gradients, and shadows restrained.
19. Verify contrast and add non-color cues.
20. Avoid undocumented internal CSS selectors.
21. Test light, dark, responsive, RTL, print, and export states.
22. Do not mix EJ2, Cartesian, and pie configuration families.
23. Ensure every imported symbol is used.
24. Emit valid, unescaped TSX.
