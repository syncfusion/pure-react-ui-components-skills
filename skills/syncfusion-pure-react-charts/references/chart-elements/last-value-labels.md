# Last Value Labels Reference

Use a last value label to display the value of the last visible data point for a series at the corresponding axis edge. This is useful for live charts and other views where the latest visible value needs emphasis.

## Required component hierarchy

Place `ChartLastValueLabel` inside the owning `ChartSeries`, and place the series inside `ChartSeriesCollection`.

```tsx
import {
  Chart,
  ChartLastValueLabel,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

<Chart>
  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="time"
      yField="value"
      type="Line"
      name="Current value"
    >
      <ChartLastValueLabel enable={true} />
    </ChartSeries>
  </ChartSeriesCollection>
</Chart>
```

Do not place `ChartLastValueLabel` directly under `Chart` or `ChartSeriesCollection`.

## Verified properties

The official `ChartLastValueLabelProps` API documents:

- `enable: boolean`, default `false`
- `background: string`, default `""`
- `border: ChartBorderProps`, default `{ color: "#D3D3D3", width: 1, dashArray: "" }`
- `font: ChartFontProps`, default `{ fontSize: "12px", fontFamily: "Roboto", fontWeight: "Normal", fontStyle: "Normal", opacity: 1, color: "" }`
- `lineStyle: ChartLineStyleProps`, default `{ width: 1, color: "", dashArray: "" }`
- `rx: number`, default `5`
- `ry: number`, default `5`

The label is axis-aligned and appears at the chart axis edge for the last visible point. 

## Important corrections

The original draft used several unsupported names:

- Use `enable`, not `visible`.
- Use `background`, not `fill`.
- Use `font.fontSize`, not `font.size`.
- Use `font.fontWeight`, not `font.bold`.
- Use `rx` and `ry` for rounded corners.
- Use `lineStyle` for the line behind or connecting to the label.
- Do not use `hAlign` or `vAlign`; these properties are not listed in the official `ChartLastValueLabelProps` API. 

Incorrect:

```tsx
<ChartLastValueLabel
  visible={true}
  fill="#4ECDC4"
  font={{ size: "12px", bold: true }}
  hAlign="Left"
  vAlign="Center"
/>
```

Correct:

```tsx
<ChartLastValueLabel
  enable={true}
  background="#4ECDC4"
  font={{
    fontSize: "12px",
    fontWeight: "Bold",
  }}
/>
```

## Basic usage

```tsx
<ChartSeries
  dataSource={data}
  xField="time"
  yField="value"
  type="Line"
>
  <ChartLastValueLabel enable={true} />
</ChartSeries>
```

The label tracks the last visible point, not necessarily the final item in the complete data array when zooming or filtering changes the visible range. 

## Background and border

Use `background` for the label background and `border` for its outline.

```tsx
<ChartLastValueLabel
  enable={true}
  background="#E3F2FD"
  border={{
    color: "#1565C0",
    width: 1,
    dashArray: "",
  }}
/>
```

The documented border default is:

```tsx
{
  color: "#D3D3D3",
  width: 1,
  dashArray: "",
}
```

Use a valid CSS color for `background` and `border.color`. Use an SVG-style comma-separated string for `dashArray`. 

## Font customization

Use the `font` object with exact `ChartFontProps` names.

```tsx
<ChartLastValueLabel
  enable={true}
  font={{
    fontSize: "12px",
    fontFamily: "Roboto",
    fontWeight: "Bold",
    fontStyle: "Normal",
    opacity: 1,
    color: "#1F1F1F",
  }}
/>
```

The official default font object is:

```tsx
{
  fontSize: "12px",
  fontFamily: "Roboto",
  fontWeight: "Normal",
  fontStyle: "Normal",
  opacity: 1,
  color: "",
}
```

Use a string such as `"12px"` for `fontSize`. Use `fontWeight: "Bold"` or another valid CSS font-weight value rather than a Boolean `bold` property. 

## Rounded corners

Use `rx` and `ry` to control horizontal and vertical corner radii.

```tsx
<ChartLastValueLabel
  enable={true}
  rx={6}
  ry={6}
/>
```

Both properties have a documented default of `5`. 

## Line styling

Use `lineStyle` for the line rendered behind or with the last value label.

```tsx
<ChartLastValueLabel
  enable={true}
  lineStyle={{
    width: 1,
    color: "#1565C0",
    dashArray: "4,2",
  }}
/>
```

The documented default is:

```tsx
{
  width: 1,
  color: "",
  dashArray: "",
}
```

Use `width={0}` inside `lineStyle` only when the current visual requirement is to suppress the line. 

## Multiple series

Configure a last value label separately for every series that needs one.

```tsx
<ChartSeriesCollection>
  <ChartSeries
    dataSource={temperatureData}
    xField="time"
    yField="value"
    type="Line"
    name="Temperature"
  >
    <ChartLastValueLabel
      enable={true}
      background="#E3F2FD"
      border={{ color: "#1565C0", width: 1, dashArray: "" }}
      font={{ fontSize: "12px", color: "#0D47A1" }}
      lineStyle={{ width: 1, color: "#1565C0", dashArray: "" }}
    />
  </ChartSeries>

  <ChartSeries
    dataSource={humidityData}
    xField="time"
    yField="value"
    type="Line"
    name="Humidity"
  >
    <ChartLastValueLabel
      enable={true}
      background="#E8F5E9"
      border={{ color: "#2E7D32", width: 1, dashArray: "" }}
      font={{ fontSize: "12px", color: "#1B5E20" }}
      lineStyle={{ width: 1, color: "#2E7D32", dashArray: "" }}
    />
  </ChartSeries>
</ChartSeriesCollection>
```

Use visually distinct but readable label styles when multiple labels can appear on the same axis edge.

## Live-update pattern

Bind the live data array to the series. The last value label updates from the last visible point when the series data changes.

```tsx
import { useEffect, useState } from "react";
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartLastValueLabel,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

type LivePoint = {
  time: Date;
  value: number;
};

export default function LiveLastValueChart() {
  const [data, setData] = useState<LivePoint[]>([
    { time: new Date(), value: 42 },
  ]);

  useEffect(() => {
    const timer = window.setInterval(() => {
      setData((current) => [
        ...current,
        {
          time: new Date(),
          value: Math.round(35 + Math.random() * 20),
        },
      ]);
    }, 2000);

    return () => window.clearInterval(timer);
  }, []);

  return (
    <Chart>
      <ChartPrimaryXAxis valueType="DateTime">
        <ChartAxisTitle text="Time" />
        <ChartAxisLabel format="hms" edgeLabelPlacement="Shift" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double">
        <ChartAxisTitle text="Value" />
        <ChartAxisLabel format="{value}" />
      </ChartPrimaryYAxis>

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="time"
          yField="value"
          type="Line"
          name="Current value"
          width={2}
        >
          <ChartLastValueLabel
            enable={true}
            background="#E3F2FD"
            border={{
              color: "#1565C0",
              width: 1,
              dashArray: "",
            }}
            font={{
              fontSize: "12px",
              fontFamily: "Roboto",
              fontWeight: "Bold",
              fontStyle: "Normal",
              opacity: 1,
              color: "#0D47A1",
            }}
            lineStyle={{
              width: 1,
              color: "#1565C0",
              dashArray: "4,2",
            }}
            rx={5}
            ry={5}
          />
        </ChartSeries>
      </ChartSeriesCollection>
    </Chart>
  );
}
```

This example demonstrates state updates and cleanup. Choose the update source and retained history according to the application requirement rather than assuming an arbitrary fixed buffer size.

## Complete static example

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartLastValueLabel,
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
  { month: "May", sales: 47 },
];

export default function LastValueLabelChart() {
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
          width={2}
        >
          <ChartLastValueLabel
            enable={true}
            background="#E3F2FD"
            border={{
              color: "#1565C0",
              width: 1,
              dashArray: "",
            }}
            font={{
              fontSize: "12px",
              fontFamily: "Roboto",
              fontWeight: "Bold",
              fontStyle: "Normal",
              opacity: 1,
              color: "#0D47A1",
            }}
            lineStyle={{
              width: 1,
              color: "#1565C0",
              dashArray: "4,2",
            }}
            rx={5}
            ry={5}
          />
        </ChartSeries>
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Series compatibility

The official source topic describes last value labels for line and area charts. Use the component with a supported series family. Do not automatically attach it to column, bar, scatter, financial, histogram, Pareto, waterfall, pie, or other series unless the current public API or a verified source example confirms support.

For a supported real-time line chart, the label represents the final visible point on the axis edge. 

## Common errors

### Using `visible`

Incorrect:

```tsx
<ChartLastValueLabel visible={true} />
```

Correct:

```tsx
<ChartLastValueLabel enable={true} />
```

### Using `fill`

Incorrect:

```tsx
<ChartLastValueLabel fill="#4ECDC4" />
```

Correct:

```tsx
<ChartLastValueLabel background="#4ECDC4" />
```

### Using incorrect font properties

Incorrect:

```tsx
<ChartLastValueLabel
  font={{ size: "12px", bold: true }}
/>
```

Correct:

```tsx
<ChartLastValueLabel
  font={{
    fontSize: "12px",
    fontWeight: "Bold",
  }}
/>
```

### Using unsupported alignment properties

Incorrect:

```tsx
<ChartLastValueLabel
  hAlign="Left"
  vAlign="Center"
/>
```

The official API does not list `hAlign` or `vAlign` for this component. The label is axis-aligned by the component. 

### Wrong hierarchy

Incorrect:

```tsx
<Chart>
  <ChartLastValueLabel enable={true} />
</Chart>
```

Correct:

```tsx
<ChartSeries>
  <ChartLastValueLabel enable={true} />
</ChartSeries>
```

## Validation checklist

Before returning a last-value-label implementation:

1. Import `ChartLastValueLabel` from `@syncfusion/react-charts`.
2. Place it inside the owning `ChartSeries`.
3. Place the series inside `ChartSeriesCollection`.
4. Use `enable={true}`, not `visible={true}`.
5. Use `background`, not `fill`.
6. Use `border` with `color`, `width`, and `dashArray`.
7. Use `font.fontSize`, not `font.size`.
8. Use `font.fontWeight`, not `font.bold`.
9. Use `lineStyle` for line width, color, and dash pattern.
10. Use `rx` and `ry` for rounded corners.
11. Do not use unsupported `hAlign` or `vAlign` properties.
12. Use the component only with a source-verified compatible series family.
13. Ensure the series has valid `dataSource`, `xField`, and `yField` mappings.
14. Ensure every imported symbol is used.
15. Emit valid, unescaped TSX.
