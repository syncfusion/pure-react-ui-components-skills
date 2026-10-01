# Range Color Mapping Reference

Use range color mapping to color each point by the value band it falls in (for example, red for low, amber for medium, and green for high) and to show those bands in the legend. It is configured once at the chart level with `ChartRangeColorCollection` and `ChartRangeColor`.

## Component hierarchy

Place `ChartRangeColorCollection` directly under `Chart`. Place one `ChartRangeColor` per band inside it.

```tsx
import {
  Chart,
  ChartLegend,
  ChartPrimaryXAxis,
  ChartRangeColor,
  ChartRangeColorCollection,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { month: "Jan", temp: 6 },
  { month: "Feb", temp: 9 },
  { month: "Mar", temp: 14 },
  { month: "Apr", temp: 19 },
  { month: "May", temp: 24 },
  { month: "Jun", temp: 29 },
  { month: "Jul", temp: 33 },
];

export default function TemperatureBands() {
  return (
    <Chart>
      <ChartPrimaryXAxis valueType="Category" />

      <ChartRangeColorCollection>
        <ChartRangeColor start={0} end={10} fill="#3B82F6" label="Cold (0–10 °C)" />
        <ChartRangeColor start={11} end={20} fill="#F59E0B" label="Mild (11–20 °C)" />
        <ChartRangeColor start={21} end={40} fill="#DC2626" label="Hot (21–40 °C)" />
      </ChartRangeColorCollection>

      <ChartLegend visible={true} mode="Range" />

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="temp"
          type="Column"
          name="Average temperature"
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

`ChartRangeColor` placed outside a `ChartRangeColorCollection` is ignored.

## `ChartRangeColor` props

- `start: number`, default `0`: inclusive lower bound of the band
- `end: number`, default `0`: inclusive upper bound of the band
- `fill: string`, default `""`: color applied to points in the band. Range mapping is only active when at least one band has a non-empty `fill`.
- `label: string`, default `""`: text shown for this band in a `Range` legend

## Which value is compared

By default each point's `yField` value is compared with the bands. To color by a different numeric field, set `colorField` on the series. The mapped value is then read from that field instead.

```tsx
<ChartSeries
  dataSource={data}
  xField="region"
  yField="revenue"
  colorField="growthPercent"
  type="Bar"
  name="Revenue"
/>
```

## Supported series

Range color mapping is applied only when **exactly one series is visible** and that series is a `Column`, `Bar`, `Scatter`, or `Bubble` series. For other types, or when several series are visible, points keep their normal series color and the legend falls back to `Series` mode.

## Legend modes

`ChartLegend.mode` accepts `"Series"`, `"Point"`, `"Range"`, and `"Gradient"`.

- `"Range"`: one legend item per `ChartRangeColor`, using its `label` and `fill`. Clicking an item toggles the points in that band.
- `"Gradient"`: a continuous color bar built from the band colors, suitable for heat-style scales.

Both modes require active range colors; without them the legend renders in `Series` mode.

```tsx
<ChartLegend visible={true} mode="Gradient" position="Bottom" />
```

## Design guidance

- Keep bands contiguous and non-overlapping so every value maps to exactly one color.
- Use a perceptually ordered palette (light to dark, or a diverging scale with a neutral midpoint).
- Always give each band a `label` that states its numeric range; do not rely on color alone.
- For decorative fills that do not carry meaning, use [gradients.md](./gradients.md) instead.

## Common errors

Incorrect: a root prop such as `rangeColorMapping` (it does not exist).

```tsx
<Chart rangeColorMapping={[{ start: 0, end: 10, colors: ["red"] }]} />
```

Incorrect: bands inside the series.

```tsx
<ChartSeries type="Column">
  <ChartRangeColor start={0} end={10} fill="#3B82F6" />
</ChartSeries>
```

Incorrect: expecting range colors on a multi-series or line chart. Use `colorField` with per-point colors, or `pointRender`, instead.

## Validation checklist

1. `ChartRangeColorCollection` is a direct child of `Chart` and contains only `ChartRangeColor` children.
2. Each band has `start`, `end`, `fill`, and a descriptive `label`.
3. Only one series is visible, of type `Column`, `Bar`, `Scatter`, or `Bubble`.
4. `colorField` is set when bands should compare a field other than `yField`.
5. `ChartLegend mode` is `"Range"` or `"Gradient"` when the bands should appear in the legend.
