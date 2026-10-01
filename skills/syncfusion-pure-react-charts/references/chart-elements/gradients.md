# Gradient Fills Reference

Use gradient fills when a series, trendline, or indicator should be painted with a linear or radial color transition instead of a flat `fill`. Gradients are declared with JSX children, not with a CSS string or an SVG `<defs>` block.

## Components

| Component | Purpose |
| --- | --- |
| `ChartLinearGradient` | Defines a linear gradient along the vector `(x1, y1)` → `(x2, y2)` |
| `ChartRadialGradient` | Defines a radial gradient centered at `(cx, cy)` with radius `r` and optional focal point `(fx, fy)` |
| `ChartGradientColorStop` | One color stop inside either gradient |

All three are exported by `@syncfusion/react-charts` and render nothing on their own.

## Supported hosts

Place exactly one gradient as a direct child of the element it paints:

- `ChartSeries`: paints the series fill (columns, bars, areas, bubbles, scatter points) or the stroke of line-type series
- `ChartTrendline`: paints the trendline stroke
- `ChartIndicator`: paints the indicator line

If more than one gradient child is present, only the first `ChartLinearGradient` or `ChartRadialGradient` is used. Do not place a gradient under `Chart`, `ChartSeriesCollection`, `ChartMarker`, or `ChartLegend`.

## Linear gradient

```tsx
import {
  Chart,
  ChartGradientColorStop,
  ChartLinearGradient,
  ChartPrimaryXAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { month: "Jan", sales: 35 },
  { month: "Feb", sales: 28 },
  { month: "Mar", sales: 34 },
  { month: "Apr", sales: 32 },
  { month: "May", sales: 40 },
];

export default function GradientArea() {
  return (
    <Chart>
      <ChartPrimaryXAxis valueType="Category" />
      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="sales"
          type="SplineArea"
          name="Sales"
        >
          {/* Vertical gradient: top (y1 = 0) to bottom (y2 = 1) */}
          <ChartLinearGradient x1={0} y1={0} x2={0} y2={1}>
            <ChartGradientColorStop offset={0} color="#3B82F6" opacity={0.9} />
            <ChartGradientColorStop offset={100} color="#3B82F6" opacity={0.05} />
          </ChartLinearGradient>
        </ChartSeries>
      </ChartSeriesCollection>
    </Chart>
  );
}
```

### `ChartLinearGradient` props

- `x1: number | string`, default `0`
- `y1: number | string`, default `0`
- `x2: number | string`, default `1`
- `y2: number | string`, default `0`

Coordinates are relative to the painted shape's bounding box: `0` is the start edge and `1` the end edge (percentage strings such as `"50%"` are also accepted). The defaults produce a left-to-right gradient. Use `x1={0} y1={0} x2={0} y2={1}` for top-to-bottom.

## Radial gradient

```tsx
<ChartSeries
  dataSource={bubbleData}
  xField="x"
  yField="y"
  sizeField="size"
  type="Bubble"
  name="Markets"
>
  <ChartRadialGradient cx="50%" cy="50%" r="50%" fx="30%" fy="30%">
    <ChartGradientColorStop offset={0} color="#FFFFFF" />
    <ChartGradientColorStop offset={100} color="#7C3AED" />
  </ChartRadialGradient>
</ChartSeries>
```

### `ChartRadialGradient` props

- `cx: number | string`, default `"50%"`
- `cy: number | string`, default `"50%"`
- `r: number | string`, default `"50%"`
- `fx: number | string`, defaults to `cx`
- `fy: number | string`, defaults to `cy`

Moving the focal point (`fx`, `fy`) away from the center gives bubbles and columns a lit, three-dimensional look.

## Color stops

### `ChartGradientColorStop` props

- `color: string`: required; any CSS color. Values beginning with `javascript:`, `data:`, or `url(` are rejected.
- `offset: number | string`: position along the gradient from `0` to `100` (a value such as `"50%"` is also accepted); out-of-range values are clamped
- `opacity: number`, default `1`, clamped to `0`–`1`
- `lighten: number`, default `0`, range `0`–`1`: lightens the stop color
- `brighten: number`, default `0`, range `-1`–`1`: brightens (positive) or darkens (negative) the stop color

Use `lighten` and `brighten` to derive related tints from one brand color without hard-coding several hex values:

```tsx
<ChartLinearGradient x1={0} y1={0} x2={0} y2={1}>
  <ChartGradientColorStop offset={0} color="#0F766E" brighten={0.4} />
  <ChartGradientColorStop offset={50} color="#0F766E" />
  <ChartGradientColorStop offset={100} color="#0F766E" brighten={-0.3} />
</ChartLinearGradient>
```

Provide at least two stops, ordered by increasing `offset`.

## Trendline and indicator gradients

```tsx
<ChartSeries dataSource={data} xField="x" yField="y" type="Scatter" name="Samples">
  <ChartTrendlineCollection>
    <ChartTrendline type="Linear" width={3} name="Trend">
      <ChartLinearGradient>
        <ChartGradientColorStop offset={0} color="#F59E0B" />
        <ChartGradientColorStop offset={100} color="#DC2626" />
      </ChartLinearGradient>
    </ChartTrendline>
  </ChartTrendlineCollection>
</ChartSeries>
```

```tsx
<ChartIndicatorCollection>
  <ChartIndicator type="Sma" seriesName="Price" period={14} width={2}>
    <ChartLinearGradient>
      <ChartGradientColorStop offset={0} color="#22C55E" />
      <ChartGradientColorStop offset={100} color="#16A34A" />
    </ChartLinearGradient>
  </ChartIndicator>
</ChartIndicatorCollection>
```

## Accessibility and legibility

- Keep enough contrast between the lightest stop and the chart background so the shape edge stays visible.
- Do not encode data meaning only in the gradient; gradients are decorative. Use range color mapping ([range-color-mapping.md](./range-color-mapping.md)) when color must represent value bands.
- Fading area fills to low opacity at the baseline keeps overlapping series readable.

## Common errors

Incorrect: a CSS gradient string as `fill`.

```tsx
<ChartSeries fill="linear-gradient(#3B82F6, #FFFFFF)" />
```

Incorrect: SVG primitives inside the chart.

```tsx
<ChartSeries>
  <linearGradient id="g"><stop offset="0" stopColor="#3B82F6" /></linearGradient>
</ChartSeries>
```

Incorrect: a gradient placed outside its host.

```tsx
<Chart>
  <ChartLinearGradient>{/* ... */}</ChartLinearGradient>
</Chart>
```

## Validation checklist

1. The gradient is a direct child of `ChartSeries`, `ChartTrendline`, or `ChartIndicator`.
2. Only one gradient per host.
3. Every `ChartGradientColorStop` has a `color`.
4. Stop offsets use the `0`–`100` scale; coordinates use the `0`–`1` bounding-box scale.
5. `ChartLinearGradient`, `ChartRadialGradient`, and `ChartGradientColorStop` are imported and used.
