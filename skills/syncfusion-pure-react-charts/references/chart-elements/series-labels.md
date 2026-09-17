# Series Labels Reference

Use series labels to identify an entire series directly inside the plot area. A series label is different from a data label because it identifies the series as a whole rather than displaying a label for every data point.

Series labels are useful when multiple line or area series must be identified without repeatedly checking the legend.

## Required component hierarchy

Place `ChartSeriesLabel` inside the `ChartSeries` that it identifies. Place every `ChartSeries` inside `ChartSeriesCollection`.

```tsx
import {
  Chart,
  ChartSeries,
  ChartSeriesCollection,
  ChartSeriesLabel,
} from "@syncfusion/react-charts";

<Chart>
  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="month"
      yField="sales"
      type="Line"
      name="Sales"
    >
      <ChartSeriesLabel visible={true} />
    </ChartSeries>
  </ChartSeriesCollection>
</Chart>
```

Do not place `ChartSeriesLabel` directly under `Chart` or `ChartSeriesCollection`.

Incorrect:

```tsx
<Chart>
  <ChartSeriesLabel visible={true} />
</Chart>
```

Correct:

```tsx
<ChartSeries>
  <ChartSeriesLabel visible={true} />
</ChartSeries>
```

## Basic series label

Set `visible={true}` to display the inline series label.

```tsx
<ChartSeries
  dataSource={data}
  xField="month"
  yField="value"
  type="Line"
  name="Revenue"
>
  <ChartSeriesLabel visible={true} />
</ChartSeries>
```

When custom label content is not supplied, keep the series `name` meaningful because the series label identifies the owning series.

## Custom label content

Use the series-label content API to replace the default series name when a different inline label is required.

```tsx
<ChartSeries
  dataSource={data}
  xField="month"
  yField="value"
  type="Line"
  name="Revenue"
>
  <ChartSeriesLabel
    visible={true}
    text="Net Revenue"
  />
</ChartSeries>
```

Use concise text that clearly identifies the series. Do not use a long sentence as a series label.

## Template content

Use the series-label template when the label requires custom React content.

```tsx
<ChartSeries
  dataSource={data}
  xField="month"
  yField="value"
  type="Line"
  name="Revenue"
>
  <ChartSeriesLabel
    visible={true}
    template={(props) => (
      <span>{props.series.name}</span>
    )}
  />
</ChartSeries>
```

Keep the template output small and presentation-focused. Avoid interactive controls, complex layouts, or large blocks of text inside the series label.

## Series-label appearance

Configure the series-label background, border, font, and opacity through `ChartSeriesLabel`.

```tsx
<ChartSeriesLabel
  visible={true}
  background="#E3F2FD"
  border={{
    color: "#1565C0",
    width: 1,
  }}
  font={{
    fontFamily: "Arial",
    fontSize: "12px",
    fontStyle: "Normal",
    fontWeight: "Bold",
    color: "#0D47A1",
    opacity: 1,
  }}
  opacity={1}
/>
```

Use exact font-property names:

- `fontFamily`
- `fontSize`
- `fontStyle`
- `fontWeight`
- `color`
- `opacity`

Use `fontSize`, not `size`. Use `fontWeight`, not `bold`.

Incorrect:

```tsx
font={{
  size: "12px",
  bold: true,
}}
```

Correct:

```tsx
font={{
  fontSize: "12px",
  fontWeight: "Bold",
}}
```

## Overlap handling

When multiple series labels are close together, use the series-label overlap-control property.

```tsx
<ChartSeriesLabel
  visible={true}
  showOverlapText={false}
/>
```

Set `showOverlapText={false}` when overlapping labels should be hidden. Enable overlapping text only when the chart design explicitly requires every label to remain visible and the result stays readable.

## Positioning

Series labels are positioned within the chart near their owning series. Use only the position properties documented by the current `ChartSeriesLabel` API.

Do not manually calculate SVG coordinates or render an external absolutely positioned label when the series-label feature supports the requested placement.

When several labels are difficult to read:

- Reduce the label text length.
- Use distinct label colors matching the series.
- Disable overlapping text.
- Increase the chart's available plot space.
- Adjust only the documented series-label position settings.

## Rotation

Use the series-label rotation API only when angled text improves readability.

```tsx
<ChartSeriesLabel
  visible={true}
  enableRotation={true}
  rotationAngle={45}
/>
```

Do not use an unverified `angle` property. Keep rotation moderate so the label remains readable.

## Multiple series labels

Configure a separate `ChartSeriesLabel` inside every series that requires an inline label.

```tsx
<ChartSeriesCollection>
  <ChartSeries
    dataSource={salesData}
    xField="month"
    yField="value"
    type="Line"
    name="Sales"
    fill="#1565C0"
  >
    <ChartSeriesLabel
      visible={true}
      text="Sales"
      background="#E3F2FD"
      border={{
        color: "#1565C0",
        width: 1,
      }}
      font={{
        fontSize: "12px",
        fontWeight: "Bold",
        color: "#0D47A1",
      }}
      showOverlapText={false}
    />
  </ChartSeries>

  <ChartSeries
    dataSource={revenueData}
    xField="month"
    yField="value"
    type="Line"
    name="Revenue"
    fill="#E65100"
  >
    <ChartSeriesLabel
      visible={true}
      text="Revenue"
      background="#FFF3E0"
      border={{
        color: "#E65100",
        width: 1,
      }}
      font={{
        fontSize: "12px",
        fontWeight: "Bold",
        color: "#BF360C",
      }}
      showOverlapText={false}
    />
  </ChartSeries>
</ChartSeriesCollection>
```

Use a consistent visual relationship between each series and its label. Matching the series-label text or border color to the series color improves identification.

## Line-series example

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
  ChartSeriesLabel,
} from "@syncfusion/react-charts";

const data = [
  { month: "Jan", sales: 35 },
  { month: "Feb", sales: 42 },
  { month: "Mar", sales: 38 },
  { month: "Apr", sales: 51 },
  { month: "May", sales: 47 },
];

export default function SeriesLabelChart() {
  return (
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
          type="Line"
          name="Sales"
          fill="#1565C0"
          width={2}
        >
          <ChartSeriesLabel
            visible={true}
            text="Sales"
            background="#E3F2FD"
            border={{
              color: "#1565C0",
              width: 1,
            }}
            font={{
              fontFamily: "Arial",
              fontSize: "12px",
              fontStyle: "Normal",
              fontWeight: "Bold",
              color: "#0D47A1",
              opacity: 1,
            }}
            opacity={1}
            showOverlapText={false}
          />
        </ChartSeries>
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Multi-series example

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
  ChartSeriesLabel,
} from "@syncfusion/react-charts";

const data = [
  { month: "Jan", sales: 35, revenue: 42 },
  { month: "Feb", sales: 42, revenue: 48 },
  { month: "Mar", sales: 38, revenue: 45 },
  { month: "Apr", sales: 51, revenue: 59 },
  { month: "May", sales: 47, revenue: 63 },
];

export default function MultiSeriesLabelsChart() {
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

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="sales"
          type="Line"
          name="Sales"
          fill="#1565C0"
          width={2}
        >
          <ChartSeriesLabel
            visible={true}
            background="#E3F2FD"
            border={{
              color: "#1565C0",
              width: 1,
            }}
            font={{
              fontSize: "12px",
              fontWeight: "Bold",
              color: "#0D47A1",
            }}
            showOverlapText={false}
          />
        </ChartSeries>

        <ChartSeries
          dataSource={data}
          xField="month"
          yField="revenue"
          type="Line"
          name="Revenue"
          fill="#E65100"
          width={2}
        >
          <ChartSeriesLabel
            visible={true}
            background="#FFF3E0"
            border={{
              color: "#E65100",
              width: 1,
            }}
            font={{
              fontSize: "12px",
              fontWeight: "Bold",
              color: "#BF360C",
            }}
            showOverlapText={false}
          />
        </ChartSeries>
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Series compatibility

Use series labels with the series families supported by the official series-label feature. Series labels are especially appropriate for line and area series because the inline label can identify the series near its path.

Do not attach `ChartSeriesLabel` to every chart type automatically. Confirm support for the selected series type before generating the final sample.

## Common errors

### Wrong hierarchy

Incorrect:

```tsx
<Chart>
  <ChartSeriesLabel visible={true} />
</Chart>
```

Correct:

```tsx
<ChartSeries>
  <ChartSeriesLabel visible={true} />
</ChartSeries>
```

### Missing series name

Incorrect when the label should use the default series name:

```tsx
<ChartSeries>
  <ChartSeriesLabel visible={true} />
</ChartSeries>
```

Correct:

```tsx
<ChartSeries name="Sales">
  <ChartSeriesLabel visible={true} />
</ChartSeries>
```

### Wrong font properties

Incorrect:

```tsx
<ChartSeriesLabel
  font={{
    size: "12px",
    bold: true,
  }}
/>
```

Correct:

```tsx
<ChartSeriesLabel
  font={{
    fontSize: "12px",
    fontWeight: "Bold",
  }}
/>
```

### Using a data-label component

Do not replace an entire-series label with `ChartDataLabel`. Data labels identify individual points, while `ChartSeriesLabel` identifies the complete series.

### Using a last-value-label component

Do not replace a series label with `ChartLastValueLabel` when the requirement is to display the series name. A last value label emphasizes the latest visible numeric value.

### Rendering custom external text

Do not render an absolutely positioned HTML element or custom SVG text when `ChartSeriesLabel` supports the requested inline-series labeling behavior.

## Validation checklist

Before returning a series-label implementation:

1. Import `ChartSeriesLabel` from `@syncfusion/react-charts`.
2. Place `ChartSeriesLabel` inside the owning `ChartSeries`.
3. Place the series inside `ChartSeriesCollection`.
4. Set `visible={true}` when the inline series label is requested.
5. Give the owning series a meaningful `name` when the default label should use the series name.
6. Use `text` only when custom label text is required.
7. Keep template content concise when a template is used.
8. Use `background`, `border`, `font`, and `opacity` only according to the series-label API.
9. Use `fontSize`, not `size`.
10. Use `fontWeight`, not `bold`.
11. Use `showOverlapText` deliberately when multiple labels may collide.
12. Use only documented position and rotation properties.
13. Confirm that the selected series type supports series labels.
14. Do not replace the feature with data labels, last value labels, legends, or annotations.
15. Ensure every imported symbol is used.
16. Emit valid, unescaped TSX.
