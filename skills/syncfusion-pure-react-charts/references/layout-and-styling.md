# Layout and Styling Reference

Use this guidance for responsive Pure React Chart layout, predictable sizing, consistent spacing, theme-driven colors, and readable visual density. Establish the chart container first, then refine the chart margin, title, legend, axes, series, markers, labels, and annotations.

## Core rules

- Give the chart a predictable width and height.
- When using percentage height, give the parent an explicit height.
- Prefer a supported chart theme and a small application palette over unrelated one-off colors.
- Keep spacing consistent across the chart, legend, controls, and surrounding content.
- Preserve plot-area space by limiting dense labels, markers, annotations, and legends.
- Test narrow widths, translated labels, browser zoom, and large datasets.
- Keep visual styling secondary to accurate data communication.

## Predictable container size

`Chart.width` and `Chart.height` accept CSS length strings such as pixels and percentages. A `100%` height depends on a parent with a resolved height. 

```tsx
<div className="chart-panel">
  <Chart width="100%" height="100%">
    {/* chart children */}
  </Chart>
</div>
```

```css
.chart-panel {
  width: 100%;
  height: 420px;
  min-width: 0;
}
```

`min-width: 0` is useful when the chart sits in a CSS Grid or Flexbox item because it allows the item to shrink instead of overflowing.

For a simple fixed-height responsive chart:

```tsx
<Chart width="100%" height="420px">
  {/* chart children */}
</Chart>
```

Avoid relying on intrinsic content height for production dashboards.

## Responsive page layout

Let CSS control the surrounding layout while the chart fills its assigned panel.

```tsx
<div className="dashboard-grid">
  <section className="chart-card">
    <Chart width="100%" height="360px">
      {/* chart children */}
    </Chart>
  </section>

  <section className="chart-card">
    <Chart width="100%" height="360px">
      {/* chart children */}
    </Chart>
  </section>
</div>
```

```css
.dashboard-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 24px;
}

.chart-card {
  min-width: 0;
  padding: 16px;
  border: 1px solid #d1d1d1;
  border-radius: 8px;
  background: #ffffff;
}

@media (max-width: 720px) {
  .dashboard-grid {
    grid-template-columns: 1fr;
  }

  .chart-card {
    padding: 12px;
  }
}
```

Use CSS media queries or a media-query hook for responsive layout. Avoid reading `window.innerWidth` directly during rendering because it is not reactive and can cause server-rendering issues.

## Chart margin

Use the root `margin` object to control space between the chart boundary and the chart area. The current API documents `top`, `right`, `bottom`, and `left`, each defaulting to `10` pixels. turn73view456

```tsx
<Chart
  margin={{
    top: 16,
    right: 16,
    bottom: 16,
    left: 16,
  }}
>
  {/* chart children */}
</Chart>
```

Do not use chart margin as a substitute for page-level card padding. Use CSS padding around the chart for page composition and `Chart.margin` for internal chart breathing room.

## Theme-driven styling

Set a supported chart `theme` and use a restrained palette when application branding requires custom series colors. The root Chart API currently documents themes including `Material`, `MaterialDark`, `Tailwind`, `TailwindDark`, `Bootstrap`, and `BootstrapDark`. 

```tsx
<Chart
  theme="Tailwind"
  palettes={[
    "#005A9C",
    "#107C10",
    "#A4262C",
    "#5C2D91",
  ]}
  background="#FFFFFF"
>
  {/* chart children */}
</Chart>
```

Import a compatible Syncfusion stylesheet once at application level. Do not mix light and dark theme resources in the same view unless the product explicitly manages theme switching.

A palette assigns colors sequentially to series. It does not guarantee sufficient contrast or color-vision accessibility. Verify series, labels, focus outlines, highlights, and tooltips against their actual backgrounds.

## Light and dark application themes

Derive chart colors from the same application theme state used by surrounding UI.

```tsx
const dark = colorMode === "dark";

<Chart
  theme={dark ? "TailwindDark" : "Tailwind"}
  background={dark ? "#1F1F1F" : "#FFFFFF"}
  border={{
    color: dark ? "#666666" : "#D1D1D1",
    width: 1,
    dashArray: "",
  }}
  focusOutline={{
    color: dark ? "#75B6FF" : "#005FCC",
    width: 2,
    offset: 2,
  }}
>
  {/* chart children */}
</Chart>
```

Use the same mode for title, axes, legend, tooltip, and annotation text. Avoid switching only the plot background while leaving labels with colors designed for the opposite theme.

## Title hierarchy

Use a concise `ChartTitle` for the visualization and normal page headings for the surrounding section.

```tsx
<section aria-labelledby="revenue-heading">
  <h2 id="revenue-heading">Revenue dashboard</h2>

  <Chart>
    <ChartTitle
      text="Monthly revenue"
      fontSize="18px"
      fontWeight="Bold"
      color="#242424"
      overflow="Wrap"
    />
  </Chart>
</section>
```

Keep long explanations outside the chart. This preserves plot height and improves accessibility.

## Axis spacing and labels

Use axis child components for title and label styling.

```tsx
<ChartPrimaryXAxis valueType="Category">
  <ChartAxisTitle
    text="Month"
    fontSize="13px"
    padding={8}
  />
  <ChartAxisLabel
    fontSize="12px"
    color="#424242"
    edgeLabelPlacement="Shift"
    intersectAction="Trim"
    maxLabelWidth={80}
  />
</ChartPrimaryXAxis>
```

For translated or long category labels, prefer wrapping or multiple rows before adding aggressive rotation.

```tsx
<ChartAxisLabel
  enableWrap={true}
  maxLabelWidth={90}
  intersectAction="Wrap"
/>
```

Use label trimming only when the full text remains available elsewhere, such as a tooltip, legend, or accessible data table.

## Legend layout

The current legend API supports positions `Auto`, `Top`, `Left`, `Bottom`, `Right`, and `Custom`, plus width, height, padding, item spacing, paging, and label-width controls. 

```tsx
<ChartLegend
  visible={true}
  position="Bottom"
  align="Center"
  padding={8}
  itemPadding={12}
  shapeWidth={10}
  shapeHeight={10}
  shapePadding={8}
  maxLabelWidth={160}
/>
```

Use `Bottom` for many dashboard and mobile layouts because it preserves horizontal plot width. Use side legends only when the chart panel is wide enough.

When many legend items cannot fit, `enablePages={true}` allows legend paging. The property defaults to `true`. 

```tsx
<ChartLegend
  visible={true}
  position="Bottom"
  enablePages={true}
/>
```

Do not fix legend width or height unless the panel dimensions are predictable and the result has been tested with the longest series names.

## Series spacing

For column and bar series, use `columnWidth` and `columnSpacing` sparingly to balance density and readability.

```tsx
<ChartSeries
  dataSource={data}
  xField="month"
  yField="sales"
  type="Column"
  columnWidth={0.65}
  columnSpacing={0.15}
/>
```

Avoid extremely thin columns or large gaps that make comparisons difficult. Test grouped and stacked series separately because their spacing needs differ.

## Markers

Markers are hidden by default, and their documented default width and height are `5` pixels. Use direct `width` and `height` properties. 

```tsx
<ChartMarker
  visible={true}
  shape="Circle"
  width={7}
  height={7}
  border={{
    color: "#FFFFFF",
    width: 1,
    dashArray: "",
  }}
/>
```

Use markers when individual points need emphasis. For dense line series, hide markers or show them only where they add meaning.

Do not use an unsupported `size={{ width, height }}` object.

## Data labels

Data labels default to hidden. Their overlap strategy defaults to `Hide`, and they support position, formatting, wrapping through templates, rotation, margins, and font styling. 

```tsx
<ChartMarker visible={true}>
  <ChartDataLabel
    visible={true}
    position="Top"
    format="{value}"
    intersectMode="Hide"
    margin={{
      top: 4,
      right: 4,
      bottom: 4,
      left: 4,
    }}
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
```

Use `fontSize`, not `size`, inside the data-label font object. Turn on rotation only when needed:

```tsx
<ChartDataLabel
  visible={true}
  enableRotation={true}
  rotationAngle={-45}
/>
```

For dense charts, label key values, the latest value, or exceptional points rather than every point.

## Annotations

Keep annotations concise and reserve them for important thresholds, events, or explanations.

```tsx
<ChartAnnotationCollection>
  <ChartAnnotation
    x="Apr"
    y={72}
    coordinateUnit="Point"
    content="Peak"
    hAlign="Center"
    vAlign="Top"
  />
</ChartAnnotationCollection>
```

Avoid placing several annotations over the same plot region. Test annotation positions after resizing and with translated content.

## Plot-area density

Prioritize the series and axes. Add supporting elements only when each one answers a specific question.

For crowded charts:

- reduce the number of visible data labels
- remove markers from dense continuous lines
- shorten or wrap category labels
- move the legend to the bottom or enable paging
- reduce annotations to significant events
- use tooltips for secondary details
- aggregate or downsample data using a domain-appropriate method
- split unrelated metrics into separate panes or charts

Do not shrink text until everything fits. Reduce information density first.

## Responsive chart component

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
  ChartTitle,
  ChartTooltip,
} from "@syncfusion/react-charts";

const data = [
  { month: "Jan", sales: 35 },
  { month: "Feb", sales: 32 },
  { month: "Mar", sales: 34 },
  { month: "Apr", sales: 40 },
  { month: "May", sales: 38 },
  { month: "Jun", sales: 43 },
];

export default function ResponsiveChart() {
  return (
    <section className="chart-card" aria-labelledby="sales-title">
      <h2 id="sales-title" className="chart-card__heading">
        Sales overview
      </h2>

      <div className="chart-card__canvas">
        <Chart
          width="100%"
          height="100%"
          theme="Tailwind"
          background="#FFFFFF"
          margin={{
            top: 12,
            right: 12,
            bottom: 12,
            left: 12,
          }}
          accessibility={{
            ariaLabel: "Monthly sales from January through June",
            role: "img",
            focusable: true,
            tabIndex: 0,
          }}
          focusOutline={{
            color: "#005FCC",
            width: 2,
            offset: 2,
          }}
        >
          <ChartTitle
            text="Monthly sales"
            fontSize="18px"
            fontWeight="Bold"
            color="#242424"
          />

          <ChartPrimaryXAxis valueType="Category">
            <ChartAxisTitle text="Month" />
            <ChartAxisLabel
              fontSize="12px"
              color="#424242"
              edgeLabelPlacement="Shift"
              intersectAction="Trim"
              maxLabelWidth={72}
            />
          </ChartPrimaryXAxis>

          <ChartPrimaryYAxis valueType="Double" minimum={0}>
            <ChartAxisTitle text="Sales" />
            <ChartAxisLabel
              format="{value}"
              fontSize="12px"
              color="#424242"
            />
          </ChartPrimaryYAxis>

          <ChartLegend
            visible={true}
            position="Bottom"
            align="Center"
            padding={8}
            itemPadding={12}
          />

          <ChartTooltip enable={true} />

          <ChartSeriesCollection>
            <ChartSeries
              dataSource={data}
              xField="month"
              yField="sales"
              type="Column"
              name="Sales"
              fill="#005A9C"
              columnWidth={0.65}
              columnSpacing={0.15}
            />
          </ChartSeriesCollection>
        </Chart>
      </div>
    </section>
  );
}
```

```css
.chart-card {
  min-width: 0;
  padding: 16px;
  border: 1px solid #d1d1d1;
  border-radius: 8px;
  background: #ffffff;
}

.chart-card__heading {
  margin: 0 0 12px;
  color: #242424;
  font-size: 20px;
  line-height: 1.3;
}

.chart-card__canvas {
  width: 100%;
  height: 420px;
  min-width: 0;
}

@media (max-width: 720px) {
  .chart-card {
    padding: 12px;
  }

  .chart-card__canvas {
    height: 340px;
  }
}
```

## Empty and loading layout

Reserve a stable panel height while data loads so the page does not jump.

```tsx
<div className="chart-card__canvas">
  {status === "loading" ? (
    <div className="chart-state" role="status">
      Loading chart data...
    </div>
  ) : status === "empty" ? (
    <div className="chart-state">
      No data is available.
    </div>
  ) : (
    <Chart width="100%" height="100%">
      {/* chart children */}
    </Chart>
  )}
</div>
```

```css
.chart-state {
  display: grid;
  height: 100%;
  place-items: center;
  padding: 24px;
  color: #424242;
  text-align: center;
}
```

The root Chart API also exposes `noDataTemplate` for the all-series-empty state. 

## Accessibility and styling

- Keep title, axis, legend, tooltip, and data-label text readable against their backgrounds.
- Maintain a visible focus outline in both light and dark modes.
- Do not use color alone to distinguish series or state.
- Use marker shapes, dash patterns, labels, or selection patterns as secondary cues.
- Test layout at browser zoom and narrow widths.
- Keep touch controls and custom buttons large enough for the target accessibility level.
- Provide a data table or text summary when dense styling cannot make every value accessible.

## Common errors

### Percentage height without parent height

Incorrect:

```tsx
<div>
  <Chart height="100%" />
</div>
```

Correct:

```tsx
<div style={{ height: "420px" }}>
  <Chart height="100%" />
</div>
```

### Excessive fixed dimensions

Avoid a fixed width such as `1200px` inside a responsive card. Prefer `width="100%"` and constrain the surrounding layout through CSS.

### Mixed spacing systems

Do not combine arbitrary page padding, chart margins, axis padding, and annotation offsets without a consistent spacing scale.

### Overloaded plot area

Do not enable markers, every data label, several annotations, crosshair, and long legend labels by default. Add only what supports interpretation.

### Unsupported marker size

Incorrect:

```tsx
<ChartMarker size={{ width: 8, height: 8 }} />
```

Correct:

```tsx
<ChartMarker width={8} height={8} />
```

### Wrong data-label font property

Incorrect:

```tsx
font={{ size: "12px" }}
```

Correct:

```tsx
font={{ fontSize: "12px" }}
```

## Validation checklist

Before returning a layout or styling implementation:

1. Give the chart a resolved width and height.
2. Give the parent an explicit height when the chart uses percentage height.
3. Use `min-width: 0` in shrinking Grid or Flexbox panels.
4. Keep page padding separate from internal chart margin.
5. Use one compatible Syncfusion theme.
6. Derive chart colors from the application theme.
7. Verify text, graphics, focus, and state contrast.
8. Keep title and axis text concise.
9. Configure long labels with wrapping, trimming, multiple rows, or rotation.
10. Choose a legend position that preserves plot space.
11. Enable legend paging when many items may overflow.
12. Use series spacing properties conservatively.
13. Hide markers when dense points make them distracting.
14. Avoid labeling every point by default.
15. Keep annotations short and sparse.
16. Reserve stable space for loading and empty states.
17. Test mobile widths, browser zoom, and long translations.
18. Avoid direct `window.innerWidth` reads during render.
19. Do not use unsupported nested sizing or font properties.
20. Do not mix EJ2 styling objects with Pure React child components.
21. Ensure every imported symbol is used.
22. Emit valid, unescaped TSX.
