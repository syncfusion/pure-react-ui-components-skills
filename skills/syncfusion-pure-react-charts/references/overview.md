# Overview Reference

Use this high-level reference to choose the chart family and establish the Pure React Chart architecture before adding detailed styling or interaction. Start with the analytical question and data shape, then select the simplest chart type that communicates the data accurately.

## Core rules

- Pick the chart family that matches the data shape and analytical task.
- Use Cartesian charts for trends, comparisons, distributions, ranges, and relationships.
- Use pie or donut charts only for a small part-to-whole dataset with one meaningful total.
- Use financial, polar, radar, histogram, Pareto, waterfall, and other specialized families only when their semantics match the data.
- Prefer one clear chart over several unrelated metrics forced into one plot.
- Choose the chart type before adding labels, markers, annotations, secondary axes, or interactions.
- Do not mix EJ2 components with `@syncfusion/react-charts` Pure React components.

## Selection workflow

Choose a chart in this order:

1. Identify the question: trend, comparison, composition, distribution, relationship, range, ranking, or change contribution.
2. Identify the X domain: category, numeric, DateTime, or logarithmic.
3. Identify the Y measure and unit.
4. Determine whether series share a common scale.
5. Determine whether the total is meaningful for part-to-whole analysis.
6. Check whether the data requires a specialized statistical, financial, or radial representation.
7. Select the simplest compatible series type.
8. Add supporting elements only when they improve interpretation.

## Cartesian charts

Cartesian charts use horizontal and vertical axes. They are the default choice for most trend and comparison tasks.

### Line and spline

Use a line chart for change across an ordered numeric or time domain.

```tsx
<ChartSeries
  dataSource={data}
  xField="date"
  yField="value"
  type="Line"
/>
```

Use `Spline` only when a smooth curve is appropriate and does not imply unsupported intermediate precision.

Good uses:

- monthly revenue
- sensor readings
- indexed performance
- rates over time

Avoid line charts for unordered categories unless the connecting path has a meaningful sequence.

### Column and bar

Use column charts for comparisons across categories and bar charts when category labels are long or ranking is the primary task.

```tsx
<ChartSeries
  dataSource={data}
  xField="category"
  yField="value"
  type="Column"
/>
```

```tsx
<ChartSeries
  dataSource={data}
  xField="category"
  yField="value"
  type="Bar"
/>
```

For comparisons, start a numeric value axis at zero unless a truncated range is analytically justified and clearly communicated.

### Stacked series

Use stacked columns, bars, or areas to show composition across categories or time while preserving the total.

```tsx
<ChartSeries
  dataSource={data}
  xField="quarter"
  yField="productA"
  type="StackingColumn"
  stackingGroup="sales"
/>
```

Use 100% stacked variants when relative share matters more than absolute total. Avoid stacks with too many segments because middle segments are difficult to compare.

### Area

Use area charts when both trend and magnitude relative to a baseline matter.

```tsx
<ChartSeries
  dataSource={data}
  xField="date"
  yField="volume"
  type="Area"
/>
```

Avoid overlapping several opaque area series. Use stacking, transparency, or separate panes when overlap hides values.

### Scatter and bubble

Use scatter plots for relationships between two numeric variables.

```tsx
<ChartSeries
  dataSource={data}
  xField="height"
  yField="weight"
  type="Scatter"
/>
```

Use bubble charts only when a third numeric variable is meaningfully encoded by size.

```tsx
<ChartSeries
  dataSource={data}
  xField="income"
  yField="lifeExpectancy"
  sizeField="population"
  type="Bubble"
/>
```

Do not map a category string to `sizeField`.

### Range charts

Use range area or range column charts when each observation contains a lower and upper bound.

```tsx
<ChartSeries
  dataSource={data}
  xField="month"
  low="minimum"
  high="maximum"
  type="RangeColumn"
/>
```

Good uses include temperature ranges, confidence bands, and planned-versus-allowed intervals.

### Step charts

Use step line or step area charts when values remain constant until a discrete change occurs.

```tsx
<ChartSeries
  dataSource={data}
  xField="time"
  yField="stateValue"
  type="StepLine"
  step="Left"
/>
```

Choose `Left`, `Right`, or `Center` according to when the change becomes effective.

## Part-to-whole charts

Use pie or donut charts only when:

- all slices represent parts of one meaningful total
- values are non-negative
- the number of slices is small enough to compare
- the total and time period are consistent

Prefer a bar chart when precise comparison or ranking matters.

Do not combine unrelated measures, multiple totals, or a long tail of tiny categories in one pie or donut.

Pie and donut components may use a separate accumulation-chart API rather than `ChartSeries`. Read the dedicated pie or accumulation-chart reference before generating code. Do not assume Cartesian component hierarchy applies to every chart family.

## Financial charts

Use `Candle`, `Hilo`, or `HiloOpenClose` when the data contains the required financial fields.

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

Add technical indicators only when the requested analysis calls for them. Keep volume or indicator series in a separate pane when a shared scale would be misleading.

## Polar and radar charts

Use polar charts for data that is inherently angular or cyclical. Use radar charts for comparing multivariate profiles across a common set of dimensions.

```tsx
<ChartSeries
  dataSource={data}
  xField="direction"
  yField="speed"
  type="PolarLine"
/>
```

```tsx
<ChartSeries
  dataSource={data}
  xField="dimension"
  yField="score"
  type="RadarArea"
/>
```

Avoid radar charts when exact comparisons matter or when axes use different units. A grouped bar chart is usually easier to compare precisely.

## Histogram

Use a histogram to show the frequency distribution of a continuous numeric variable.

```tsx
<ChartSeries
  dataSource={data}
  yField="value"
  type="Histogram"
/>
```

Do not use a histogram for categorical counts. Use a column or bar chart for categories.

## Pareto

Use a Pareto chart when categories should be ranked by contribution and paired with cumulative percentage to identify the few categories responsible for most of an outcome.

```tsx
<ChartSeries
  dataSource={data}
  xField="cause"
  yField="count"
  type="Pareto"
/>
```

Sort or validate the category order required by the implementation and explain the cumulative measure clearly.

## Waterfall

Use a waterfall chart to show how positive and negative contributions move a starting value toward an ending value.

```tsx
<ChartSeries
  dataSource={data}
  xField="stage"
  yField="change"
  type="Waterfall"
/>
```

Good uses include profit bridges, budget variance, and inventory reconciliation. Do not use waterfall for unrelated category magnitudes.

## Box-and-whisker

Use a box-and-whisker chart to compare distributions through quartiles, median, whiskers, and outliers.

```tsx
<ChartSeries
  dataSource={data}
  xField="group"
  yField="values"
  type="BoxAndWhisker"
/>
```

Do not replace distribution data with a single average when spread and outliers are important.

## Chart architecture

The standard Cartesian Pure React hierarchy is:

```text
Chart
├── ChartTitle
├── ChartPrimaryXAxis
│   ├── ChartAxisTitle
│   └── ChartAxisLabel
├── ChartPrimaryYAxis
│   ├── ChartAxisTitle
│   └── ChartAxisLabel
├── Optional root elements
│   ├── ChartLegend
│   ├── ChartTooltip
│   ├── ChartCrosshair
│   ├── ChartSelection
│   ├── ChartHighlight
│   └── ChartZoomSettings
└── ChartSeriesCollection
    └── ChartSeries
        ├── ChartMarker
        │   └── ChartDataLabel
        ├── ChartTrendlineCollection
        ├── ChartErrorBar
        └── Other series-owned elements
```

Every `ChartSeries` must be inside `ChartSeriesCollection`.

## Minimal architecture example

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
  ChartTitle,
  ChartTooltip,
} from "@syncfusion/react-charts";

const data = [
  { month: "Jan", sales: 100 },
  { month: "Feb", sales: 120 },
  { month: "Mar", sales: 110 },
  { month: "Apr", sales: 145 },
];

export default function SalesChart() {
  return (
    <Chart
      accessibility={{
        ariaLabel: "Monthly sales from January through April",
        role: "img",
        focusable: true,
        tabIndex: 0,
      }}
    >
      <ChartTitle text="Monthly sales" />

      <ChartPrimaryXAxis valueType="Category">
        <ChartAxisTitle text="Month" />
        <ChartAxisLabel edgeLabelPlacement="Shift" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double" minimum={0}>
        <ChartAxisTitle text="Sales" />
        <ChartAxisLabel format="{value}" />
      </ChartPrimaryYAxis>

      <ChartTooltip enable={true} />

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="sales"
          type="Column"
          name="Sales"
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Axis selection

Match the X-axis to the data:

- `Category`: discrete text labels
- `DateTime`: real dates and times
- `Double`: continuous numeric values
- `Logarithmic`: positive values spanning orders of magnitude

```tsx
<ChartPrimaryXAxis valueType="DateTime" />
```

Use multiple named axes only when series have genuinely different units or scales. If separate visual regions are clearer, use multiple panes or separate charts.

## Multiple series

Combine series when they share a meaningful X domain and comparison context.

```tsx
<ChartSeriesCollection>
  <ChartSeries
    dataSource={data}
    xField="month"
    yField="actual"
    type="Column"
    name="Actual"
  />
  <ChartSeries
    dataSource={data}
    xField="month"
    yField="target"
    type="Line"
    name="Target"
  />
</ChartSeriesCollection>
```

Avoid using a secondary axis merely to make unrelated lines fit. If scales differ, label every axis clearly and keep each series-axis mapping explicit.

## Supporting elements

Add elements according to the question:

- title and axis titles: establish purpose and units
- legend: identify multiple series
- tooltip: provide secondary point detail
- marker: emphasize individual line or scatter points
- data label: show selected important values
- annotation: explain a notable event
- stripline: show a threshold or range
- trendline: show a fitted or smoothed direction
- error bar: communicate uncertainty
- crosshair: support precise axis reading
- zoom and pan: explore dense or long ranges

Do not enable all supporting elements by default.

## Accessibility and interpretation

- Provide a meaningful root accessibility label.
- Keep titles, units, and visible labels clear.
- Do not rely on color alone to distinguish series or states.
- Provide a text summary or accessible data table for complex charts.
- Test keyboard access for interactive charts.
- Avoid chart types whose visual encoding makes the required comparison unnecessarily difficult.

## Common selection mistakes

### Pie for ranking

Use a sorted bar chart when users must compare many categories precisely.

### Line for unordered categories

Use columns, bars, or dots unless the category order and connecting path carry meaning.

### Multiple axes for unrelated metrics

Use separate panes or charts when the relationship is weak or the scales create a misleading visual comparison.

### Area chart with overlapping opaque series

Use line series, stacking, transparency, or separate panes.

### Radar chart with incompatible units

Normalize values only when analytically justified and clearly explain the normalization. Otherwise use grouped bars or small multiples.

### Histogram for categories

Use a bar or column chart for categorical frequencies.

### Specialized chart without specialized data

Do not use candlestick without OHLC values, bubble without a meaningful numeric size field, range charts without lower and upper bounds, or waterfall without additive contributions.

## Validation checklist

Before returning a chart recommendation or architecture:

1. Identify the analytical question.
2. Identify the data shape and units.
3. Choose the chart family before styling.
4. Use Cartesian charts for most trends and comparisons.
5. Use pie or donut only for a small, valid part-to-whole dataset.
6. Use specialized charts only when their semantics match the data.
7. Match the X-axis value type to the bound values.
8. Map every required data field explicitly.
9. Keep every Cartesian series inside `ChartSeriesCollection`.
10. Use a separate family-specific reference for pie and donut APIs.
11. Use additional axes only for genuinely different scales.
12. Prefer separate panes or charts when metrics are unrelated.
13. Add only supporting elements that improve interpretation.
14. Avoid visual encodings that imply unsupported precision or continuity.
15. Provide accessibility labels, units, and non-color cues.
16. Test dense data, long labels, mobile size, and keyboard interaction.
17. Do not mix EJ2 and Pure React APIs.
18. Ensure every imported symbol is used.
19. Emit valid, unescaped TSX.
