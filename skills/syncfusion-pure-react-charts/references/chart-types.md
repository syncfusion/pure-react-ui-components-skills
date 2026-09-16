# Chart Types Reference

Use this file to select the correct chart family, exact series `type`, required data mappings, and component hierarchy. For type-specific options, also read the linked detailed reference.

## Non-negotiable implementation rules

- Use `Chart`, `ChartSeriesCollection`, and `ChartSeries` for Cartesian, financial, histogram, Pareto, waterfall, polar, and radar requests unless the detailed family reference specifies otherwise.
- Use the separate `PieChart`, `PieChartSeriesCollection`, and `PieChartSeries` family for pie and doughnut requests.
- Place every `ChartSeries` inside `ChartSeriesCollection`.
- Use real TSX. Never emit `&lt;`, `&gt;`, or `=&gt;` inside code fences.
- Do not use invented secondary-axis tags such as `ChartSecondaryYAxis`. Define additional axes with `ChartAxes`, give each axis a unique `name`, and map the series with `xAxisName` or `yAxisName`.
- Do not configure marker or data-label child components as guessed object props. When requested, place `ChartMarker` inside `ChartSeries` and `ChartDataLabel` inside `ChartMarker`.
- Use only type literals and values verified below or in the linked dedicated reference.

## Base Cartesian hierarchy

```tsx
import {
  Chart,
  ChartPrimaryXAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { month: "Jan", value: 35 },
  { month: "Feb", value: 42 },
  { month: "Mar", value: 38 },
];

export default function App() {
  return (
    <Chart>
      <ChartPrimaryXAxis valueType="Category" />
      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="value"
          type="Line"
          name="Value"
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Public Cartesian series type literals

Use exact casing. The public `ChartSeriesType` union documents:

- `Line`
- `MultiColoredLine`
- `MultiColoredArea`
- `Column`
- `Bar`
- `Area`
- `StackingColumn`
- `StackingColumn100`
- `StackingBar`
- `StackingBar100`
- `StackingLine`
- `StackingLine100`
- `StepLine`
- `StepArea`
- `StackingStepArea`
- `Spline`
- `SplineArea`
- `SplineRangeArea`
- `Scatter`
- `Bubble`
- `Candle`
- `Hilo`
- `HiloOpenClose`
- `RangeArea`
- `RangeStepArea`
- `RangeColumn`
- `StackingArea`
- `StackingArea100`
- `Waterfall`
- `Histogram`
- `Pareto`
- `BoxAndWhisker`
- Polar variants: `PolarLine`, `PolarArea`, `PolarColumn`, `PolarSpline`, `PolarSplineArea`, `PolarScatter`, `PolarRangeColumn`, `PolarStackingArea`, `PolarStackingColumn`
- Radar variants: `RadarLine`, `RadarArea`, `RadarColumn`, `RadarSpline`, `RadarSplineArea`, `RadarScatter`, `RadarRangeColumn`, `RadarStackingArea`, `RadarStackingColumn`

Do not emit `OHLC`, `Ohlc`, or `HiLo` as `type` values. Use `HiloOpenClose` for OHLC and `Hilo` for high-low series.

## Type selector

- Trend: `Line`, `Spline`, `StepLine`, `MultiColoredLine`, `StackingLine`, `StackingLine100`
- Magnitude or cumulative area: `Area`, `SplineArea`, `StepArea`, `MultiColoredArea`, `StackingArea`, `StackingArea100`, `StackingStepArea`
- Category comparison: `Column`, `Bar`, `RangeColumn`
- Part-to-whole across series: `StackingColumn`, `StackingColumn100`, `StackingBar`, `StackingBar100`
- Value range: `RangeArea`, `RangeStepArea`, `SplineRangeArea`, `RangeColumn`
- Correlation: `Scatter`, `Bubble`
- Distribution: `Histogram`
- Statistical distribution by category: `BoxAndWhisker`
- Ranked contribution and cumulative percentage: `Pareto`
- Sequential gains and losses: `Waterfall`
- Financial range or OHLC: `Hilo`, `HiloOpenClose`, `Candle`
- Part-to-whole by slices: use the `PieChart` family; see [pie-and-donut.md](pie-and-donut.md)
- Circular coordinates: use the `Polar*` and `Radar*` series type literals listed above

## Line family

Configure `Line`, `Spline`, and `StepLine` with the mappings and styling shown below.

### Line

Required mappings: `dataSource`, `xField`, and `yField`.

```tsx
<ChartSeries
  dataSource={data}
  xField="month"
  yField="value"
  type="Line"
  name="Sales"
  width={2}
  dashArray="5,5"
/>
```

`fill`, `width`, and `opacity` customize the series. `dashArray` controls the dash pattern.

### Spline

```tsx
<ChartSeries
  dataSource={data}
  xField="month"
  yField="value"
  type="Spline"
  splineType="Natural"
/>
```

Documented `splineType` values are `Natural`, `Monotonic`, `Cardinal`, and `Clamped`. `cardinalSplineTension` applies to cardinal splines.

### Step line

```tsx
<ChartSeries
  dataSource={data}
  xField="month"
  yField="value"
  type="StepLine"
  step="Center"
  noRisers={false}
/>
```

Documented `step` values are `Left`, `Center`, and `Right`. Set `noRisers={true}` to omit vertical risers.

### Multi-colored line

The mapped color field must exist in every applicable data object.

```tsx
const data = [
  { x: 1, y: 10, color: "#2E86DE" },
  { x: 2, y: 18, color: "#E67E22" },
];

<ChartSeries
  dataSource={data}
  xField="x"
  yField="y"
  colorField="color"
  type="MultiColoredLine"
/>
```

### Empty points

The source documents `Gap`, `Zero`, `Average`, and `Drop` modes for Cartesian empty points.

```tsx
<ChartSeries
  dataSource={data}
  xField="x"
  yField="y"
  type="Line"
  emptyPointSettings={{ mode: "Gap", fill: "#BDBDBD" }}
/>
```

## Area family

Configure `Area`, `SplineArea`, `StepArea`, and `RangeArea` with the mappings and styling shown below.

### Area variants

```tsx
<ChartSeries dataSource={data} xField="x" yField="y" type="Area" />
<ChartSeries dataSource={data} xField="x" yField="y" type="SplineArea" />
<ChartSeries dataSource={data} xField="x" yField="y" type="StepArea" />
<ChartSeries dataSource={data} xField="x" yField="y" type="MultiColoredArea" colorField="color" />
```

`SplineArea` supports `splineType`. `StepArea` supports the documented step-series options. Area appearance uses series properties such as `fill`, `border`, and `opacity` where documented.

### Stacking area

All participating series must use the same stacking type and compatible X values.

```tsx
<ChartSeriesCollection>
  <ChartSeries dataSource={productA} xField="month" yField="value" type="StackingArea" name="A" />
  <ChartSeries dataSource={productB} xField="month" yField="value" type="StackingArea" name="B" />
</ChartSeriesCollection>
```

Use `StackingArea100` for normalized percentage contribution.

### Range and spline-range area

Do not use `yField` as one range boundary. Map both `high` and `low`.

```tsx
const rangeData = [
  { day: "Mon", low: 18, high: 29 },
  { day: "Tue", low: 20, high: 31 },
];

<ChartSeries
  dataSource={rangeData}
  xField="day"
  low="low"
  high="high"
  type="RangeArea"
/>
```

Use `type="SplineRangeArea"` for smooth range boundaries.

## Column and bar family

Configure `Column`, `Bar`, `RangeColumn`, `StackingColumn`, and `StackingBar` with the mappings and styling shown below.

### Column and bar

```tsx
<ChartSeries dataSource={data} xField="category" yField="value" type="Column" />
<ChartSeries dataSource={data} xField="category" yField="value" type="Bar" />
```

Supported column/bar-specific configuration documented by the source includes:

- `columnFacet="Cylinder"` for cylindrical column or bar rendering. Do not emit `Pyramid` unless the current API explicitly lists it.
- `columnSpacing`: a relative spacing value from 0 to 1.
- `columnWidth`: a relative width value from 0 to 1.
- `columnWidthInPixel`: fixed width in pixels.
- `groupName`: clusters series sharing the same group.
- `fill`, `opacity`, `border`, and `cornerRadius` for appearance.
- `colorField` for data-driven point colors.
- Root `Chart.enableSideBySidePlacement`, whose documented default is `true`.

### Stacking column and bar

```tsx
<ChartSeriesCollection>
  <ChartSeries dataSource={north} xField="quarter" yField="value" type="StackingColumn" stackingGroup="Sales" name="North" />
  <ChartSeries dataSource={south} xField="quarter" yField="value" type="StackingColumn" stackingGroup="Sales" name="South" />
</ChartSeriesCollection>
```

Use exact corresponding types:

- `StackingColumn`
- `StackingColumn100`
- `StackingBar`
- `StackingBar100`

Series with the same `stackingGroup` stack together. Different groups form independent stacks.

### Range column

```tsx
<ChartSeries
  dataSource={rangeData}
  xField="day"
  low="low"
  high="high"
  type="RangeColumn"
/>
```

## Scatter and bubble

Configure `Scatter` and `Bubble` with the mappings and styling shown below.

### Scatter

Configure marker dimensions with the `ChartMarker` child, not an invented `size` object on `ChartSeries`.

```tsx
<ChartSeries dataSource={data} xField="x" yField="y" type="Scatter">
  <ChartMarker visible={true} shape="Circle" width={8} height={8} />
</ChartSeries>
```

### Bubble

The bubble requires X, Y, and size data. Use the exact size-field mapping documented by the current `ChartSeriesProps` API. Do not pass a number-size object intended for marker dimensions.

```tsx
const bubbleData = [
  { x: 10, y: 20, size: 12 },
  { x: 15, y: 28, size: 20 },
];

<ChartSeries
  dataSource={bubbleData}
  xField="x"
  yField="y"
  size="size"
  type="Bubble"
/>
```

Bubble-specific source options include `colorField`, `minRadius`, `maxRadius`, `fill`, `width`, and `opacity` where documented.

## Financial family

Configure `Candle`, `Hilo`, and `HiloOpenClose` with the mappings and styling shown below.

Use date-compatible X values and a `DateTime` axis when plotting a real timeline.

### Candle

```tsx
<ChartSeries
  dataSource={financialData}
  xField="date"
  open="open"
  high="high"
  low="low"
  close="close"
  type="Candle"
/>
```

Source-supported options include `enableSolidCandles`, `bullFillColor`, and `bearFillColor`.

### Hilo

```tsx
<ChartSeries
  dataSource={financialData}
  xField="date"
  high="high"
  low="low"
  type="Hilo"
/>
```

### OHLC

The display name is OHLC, but the exact series literal is `HiloOpenClose`.

```tsx
<ChartSeries
  dataSource={financialData}
  xField="date"
  open="open"
  high="high"
  low="low"
  close="close"
  type="HiloOpenClose"
/>
```

## Histogram

Histogram input is a set of numeric observations. The source places bin configuration under `histogramSettings`, rather than using an unverified root `binInterval` series prop.

```tsx
const observations = [
  { value: 15 },
  { value: 18 },
  { value: 23 },
  { value: 31 },
];

<ChartSeries
  dataSource={observations}
  yField="value"
  type="Histogram"
  histogramSettings={{
    binInterval: 10,
    showNormalDistribution: true,
  }}
/>
```

Documented histogram settings include `binInterval`, `showNormalDistribution`, `normalCurveColor`, and `normalCurveDashArray`.

## Pareto

Use the native `Pareto` series. Do not rebuild it as an invented column-plus-line combination unless the user explicitly requests a custom combination chart.

```tsx
<ChartSeries
  dataSource={data}
  xField="category"
  yField="value"
  type="Pareto"
  paretoOptions={{
    showAxis: true,
    fill: "#E67E22",
    width: 2,
    dashArray: "0",
  }}
/>
```

Documented Pareto options include `showAxis` with default `true`, `fill`, `width` with default `1`, and `dashArray` with default `"0"`.

## Waterfall

```tsx
<ChartSeries
  dataSource={data}
  xField="category"
  yField="value"
  type="Waterfall"
  waterfallSettings={{
    intermediateSumIndexes: [3],
    sumIndexes: [6],
    positiveColor: "#2E7D32",
    negativeColor: "#C62828",
    connectorLine: {
      visible: true,
      strokeWidth: 1,
      strokeOpacity: 1,
    },
  }}
/>
```

Waterfall settings documented by the source include `intermediateSumIndexes`, `sumIndexes`, `positiveColor`, `negativeColor`, and `connectorLine`. Connector options include `visible`, `strokeColor`, `strokeWidth`, `strokeOpacity`, and `dashArray`.

## Box-and-Whisker

Each category maps to an array of numeric Y values. The series computes quartiles and outliers.

```tsx
const distributions = [
  { category: "A", values: [12, 15, 16, 17, 22, 31] },
  { category: "B", values: [10, 14, 18, 21, 24, 29] },
];

<ChartSeries
  dataSource={distributions}
  xField="category"
  yField="values"
  type="BoxAndWhisker"
  boxAndWhiskerSettings={{
    boxPlotMode: "Normal",
    showMean: true,
    showOutliers: true,
    whiskerStyle: {
      stroke: "#555",
      width: 1,
      dashArray: "",
      capLength: 0.5,
    },
  }}
/>
```

Documented `boxPlotMode` values are `Normal`, `Exclusive`, and `Inclusive`. `showMean` and `showOutliers` default to `true`. Whisker styling uses `stroke`, `width`, `dashArray`, and `capLength`.

## Pie and doughnut

Read [pie-and-donut.md](pie-and-donut.md) before generating these charts. Do not use Cartesian `ChartSeries` for them.

```tsx
import {
  PieChart,
  PieChartDataLabel,
  PieChartLegend,
  PieChartSeries,
  PieChartSeriesCollection,
  PieChartTooltip,
} from "@syncfusion/react-charts";

const data = [
  { category: "A", value: 40 },
  { category: "B", value: 35 },
  { category: "C", value: 25 },
];

export default function App() {
  return (
    <PieChart>
      <PieChartSeriesCollection>
        <PieChartSeries dataSource={data} xField="category" yField="value">
          <PieChartDataLabel visible={true} position="Outside" />
        </PieChartSeries>
      </PieChartSeriesCollection>
      <PieChartLegend visible={true} position="Bottom" />
      <PieChartTooltip enable={true} />
    </PieChart>
  );
}
```

Create a doughnut by setting `innerRadius` on `PieChartSeries`:

```tsx
<PieChartSeries
  dataSource={data}
  xField="category"
  yField="value"
  innerRadius="50%"
/>
```

Pie-series source features include `radius`, `startAngle`, `endAngle`, `borderRadius`, grouping, empty-point settings, explode behavior, labels, selection, highlight, patterns, and annotations. Use only the exact options from the dedicated pie references and their linked API pages.

## Polar and radar

Use the exact specialized type literals listed in the union above (`PolarLine`, `PolarArea`, `RadarLine`, `RadarColumn`, and so on). Do not use generic `Polar` or `Radar` values — they do not exist in `ChartSeriesType`. Configure the circular axes with `coefficient`, `startAngle` (axis props), and `isClosedPath` (series prop) as documented in the axis reference.

## Combination charts

A combination chart uses multiple supported `ChartSeries` types in one `ChartSeriesCollection`.

```tsx
<ChartSeriesCollection>
  <ChartSeries dataSource={data} xField="month" yField="revenue" type="Column" name="Revenue" />
  <ChartSeries dataSource={data} xField="month" yField="growth" type="Line" name="Growth" yAxisName="growthAxis" />
</ChartSeriesCollection>
```

When scales differ, define a named additional axis through `ChartAxes` and map the series using the matching `yAxisName`. Read [multiple axes](axis-configuration/multiple-axes.md) before emitting the axis hierarchy.

## Shared series properties

Do not claim every property applies to every type. Apply properties only to the families documented by their detailed references.

Commonly documented `ChartSeries` properties include:

- Binding: `dataSource`, `xField`, `yField`
- Identity and axes: `name`, `xAxisName`, `yAxisName`
- Appearance: `fill`, `opacity`, `width`, `border`, `dashArray`
- Data-driven color: `colorField`
- Animation: `animation` with `enable`, `duration`, and `delay`
- Empty points: `emptyPointSettings`
- Financial mappings: `open`, `high`, `low`, `close`
- Range mappings: `high`, `low`
- Type-specific configuration: `splineType`, `step`, `noRisers`, `stackingGroup`, `groupName`, `columnFacet`, `columnSpacing`, `columnWidth`, `columnWidthInPixel`, `histogramSettings`, `paretoOptions`, `waterfallSettings`, and `boxAndWhiskerSettings`

Use child components for marker, data-label, trendline, and other feature structures where the dedicated reference documents child-based configuration.

## Final validation checklist

Before returning any chart sample:

1. Confirm the root chart family is correct.
2. Confirm the exact `type` literal and casing.
3. Confirm required mappings exist in the data.
4. Confirm range and financial series include every required field mapping.
5. Confirm `ChartSeries` is wrapped by `ChartSeriesCollection`.
6. Confirm feature children use the hierarchy from their dedicated references.
7. Confirm additional axes use `ChartAxes`, not invented secondary-axis tags.
8. Confirm every imported symbol is used and every used symbol is imported.
9. Confirm all JSX is real, balanced TSX.
10. Remove arbitrary performance thresholds and unverified claims.
