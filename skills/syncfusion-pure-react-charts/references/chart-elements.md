# Chart Elements Reference

Use this umbrella reference to route chart-element requests to the correct detailed page. Each element has a specific owner in the Pure React component hierarchy, so read the matching topic before combining elements.

## Table of contents

1. [Annotations](./chart-elements/annotations.md)
2. [Data labels](./chart-elements/data-labels.md)
3. [Markers](./chart-elements/markers.md)
4. [Trendlines](./chart-elements/trendlines.md)
5. [Striplines](./chart-elements/striplines.md)
6. [Legend](./chart-elements/legend.md)
7. [Series labels](./chart-elements/series-labels.md)
8. [Last value labels](./chart-elements/last-value-labels.md)
9. [Error bars](./chart-elements/error-bars.md)
10. [Indicators](./chart-elements/indicators-technical-analysis.md)

## Ownership summary

Use the correct owner for each element:

- Chart-level: annotation collection and legend
- Series-level: markers, trendline collection, series labels, last-value labels, error bars, and indicators
- Marker-level: data labels for marker-based Cartesian series
- Axis-level: striplines

Do not move a child component to the root simply because it affects the complete chart.

## Base composition pattern

```tsx
import {
  Chart,
  ChartDataLabel,
  ChartLegend,
  ChartMarker,
  ChartPrimaryXAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

<Chart>
  <ChartPrimaryXAxis valueType="Category" />
  <ChartLegend visible={true} />

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="month"
      yField="sales"
      type="Line"
      name="Sales"
    >
      <ChartMarker
        visible={true}
        shape="Circle"
        width={8}
        height={8}
      >
        <ChartDataLabel
          visible={true}
          position="Top"
          format="{value}"
        />
      </ChartMarker>
    </ChartSeries>
  </ChartSeriesCollection>
</Chart>
```

Every `ChartSeries` must remain inside `ChartSeriesCollection`.

## Annotations

Use annotations for explanatory text or content anchored to an axis point or pixel position.

```tsx
<ChartAnnotationCollection>
  <ChartAnnotation
    x="Apr"
    y={72}
    coordinateUnit="Point"
    content="Peak"
  />
</ChartAnnotationCollection>
```

`Point` positioning follows axis values. `Pixel` positioning uses chart-relative pixel coordinates. When an annotation targets named axes, map `xAxisName` and `yAxisName` correctly.

Read [Annotations](./chart-elements/annotations.md) for collection hierarchy, alignment, accessibility, and content rules.

## Data labels

Use data labels to show values or mapped text close to points.

```tsx
<ChartMarker visible={true}>
  <ChartDataLabel
    visible={true}
    position="Top"
    format="{value}"
    intersectMode="Hide"
  />
</ChartMarker>
```

Data labels default to hidden. Use `font.fontSize`, not `font.size`, and enable rotation before setting a rotation angle.

Read [Data labels](./chart-elements/data-labels.md) for formatters, templates, overlap handling, mapped label fields, and appearance.

## Markers

Use markers to make individual points visible in line, spline, area, or scatter-style charts.

```tsx
<ChartMarker
  visible={true}
  shape="Diamond"
  width={8}
  height={8}
  fill="#1976D2"
/>
```

Use direct `width` and `height` properties. Do not use a guessed `size` object.

Read [Markers](./chart-elements/markers.md) for shapes, borders, images, offsets, opacity, and nested data labels.

## Trendlines

Place `ChartTrendline` inside `ChartTrendlineCollection`, then place the collection inside its parent series.

```tsx
<ChartSeries
  dataSource={data}
  xField="x"
  yField="y"
  type="Scatter"
>
  <ChartTrendlineCollection>
    <ChartTrendline
      type="Linear"
      name="Linear trend"
      stroke="#C62828"
      width={2}
    />
  </ChartTrendlineCollection>
</ChartSeries>
```

Verified trendline types include `Linear`, `Exponential`, `Polynomial`, `Power`, `Logarithmic`, and `MovingAverage`. Use `stroke` for the trendline color.

Read [Trendlines](./chart-elements/trendlines.md) for forecasts, polynomial order, moving-average period, intercepts, tooltips, and accessibility.

## Striplines

Striplines belong to an axis because their range is interpreted using that axis scale.

```tsx
<ChartPrimaryYAxis valueType="Double">
  <ChartStripLines>
    <ChartStripLine
      visible={true}
      range={{ start: 100, end: 120 }}
      style={{ color: '#FFE5E5' }}
      text={{ content: 'Target' }}
    />
  </ChartStripLines>
</ChartPrimaryYAxis>
```

Read [Striplines](./chart-elements/striplines.md) for ranges, lines, labels, orientation, and named-axis placement.

## Legend

Place `ChartLegend` directly inside `Chart`.

```tsx
<ChartLegend
  visible={true}
  position="Bottom"
  align="Center"
  toggleVisibility={true}
/>
```

Legend visibility defaults to enabled, and clicking a legend item toggles its series by default. Give every series a meaningful `name`.

Read [Legend](./chart-elements/legend.md) for position, alignment, paging, sizing, titles, custom placement, accessibility, and click behavior.

## Series labels

Use series labels when a line or area series should identify itself directly near the rendered path.

```tsx
<ChartSeries type="Line" name="Revenue">
  <ChartSeriesLabel visible={true} />
</ChartSeries>
```

Read [Series labels](./chart-elements/series-labels.md) before adding templates, placement, or styling. Do not replace point-level data labels with series labels when every value must be shown.

## Last-value labels

Use a last-value label to emphasize the latest value of a supported line or area series.

```tsx
<ChartSeries type="Line">
  <ChartLastValueLabel visible={true} />
</ChartSeries>
```

Read [Last value labels](./chart-elements/last-value-labels.md) for supported series, styling, alignment, and real-time update guidance.

## Error bars

Place error-bar configuration inside its parent series.

```tsx
<ChartSeries
  dataSource={data}
  xField="category"
  yField="value"
  type="Column"
>
  <ChartErrorBar
    visible={true}
    type="Custom"
    verticalError={5}
  />
</ChartSeries>
```

Read [Error bars](./chart-elements/error-bars.md) for fixed, percentage, standard-deviation, standard-error, and custom error values. Keep the displayed uncertainty consistent with the underlying calculation.

## Indicators

Use technical indicators only with a compatible financial or numeric source series, bound through the indicator's `seriesName`.

```tsx
<ChartIndicatorCollection>
  <ChartIndicator
    type="Ema"
    field="Close"
    period={10}
    seriesName="Price"
  />
</ChartIndicatorCollection>
```

Read [Indicators](./chart-elements/indicators-technical-analysis.md) for exact indicator types, periods, fields, required financial mappings, and axis placement.

## Combining elements

Add only elements that improve interpretation. A typical analytical chart may combine a legend, marker, data label, tooltip, and one annotation, but adding every feature can obscure the data.

```tsx
<Chart>
  <ChartLegend visible={true} position="Bottom" />
  <ChartTooltip enable={true} />

  <ChartAnnotationCollection>
    <ChartAnnotation
      x="Apr"
      y={72}
      coordinateUnit="Point"
      content="Peak"
    />
  </ChartAnnotationCollection>

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="month"
      yField="value"
      type="Line"
      name="Value"
    >
      <ChartMarker visible={true} width={7} height={7}>
        <ChartDataLabel
          visible={true}
          position="Top"
        />
      </ChartMarker>
    </ChartSeries>
  </ChartSeriesCollection>
</Chart>
```

When combining elements, verify the hierarchy of every child independently.

## Performance guidance

Performance depends on point count, series type, device, animation, and update frequency. Profile the target application before applying arbitrary thresholds.

For dense charts:

- reduce unnecessary data labels
- hide markers when they do not add meaning
- limit annotations to important events
- avoid multiple expensive trendlines on rapidly changing data
- disable animation for frequent live updates when needed
- keep streamed history bounded
- aggregate or downsample data using a domain-appropriate method

Do not describe ordinary point filtering as virtualization.

## Accessibility guidance

Do not rely only on color to distinguish chart elements. Combine colors with labels, marker shapes, dash patterns, or selection patterns.

Add accessibility configuration where the element API supports it, including annotations, trendlines, legend configuration, and series descriptions. Keep visible titles and axis units clear, and provide a text summary or data table for complex charts.

## Routing rules

Use the detailed page that matches the request:

- Point- or pixel-positioned explanatory content: `annotations.md`
- Values displayed beside points: `data-labels.md`
- Point symbols and shapes: `markers.md`
- Regression, fitting, or moving averages: `trendlines.md`
- Thresholds and reference ranges: `striplines.md`
- Series identification and visibility toggling: `legend.md`
- Direct labels for complete series: `series-labels.md`
- Latest-value emphasis: `last-value-labels.md`
- Uncertainty and error ranges: `error-bars.md`
- Financial technical analysis: `indicators-technical-analysis.md`

## Validation checklist

Before returning a chart-element implementation:

1. Read the detailed reference for every requested element.
2. Import each component from `@syncfusion/react-charts`.
3. Keep every series inside `ChartSeriesCollection`.
4. Place chart-level components directly inside `Chart`.
5. Place series-owned elements inside `ChartSeries`.
6. Place data labels inside `ChartMarker` for marker-based Cartesian series.
7. Place striplines inside the owning axis.
8. Place trendlines inside `ChartTrendlineCollection`.
9. Match named axes exactly when an element uses axis coordinates.
10. Use exact property names, literal values, and casing.
11. Avoid unsupported nested object configurations copied from EJ2.
12. Avoid dense combinations that reduce readability.
13. Keep interactive and live charts performant.
14. Add non-color cues and useful accessibility descriptions.
15. Ensure every imported symbol is used.
16. Emit valid, unescaped TSX.
