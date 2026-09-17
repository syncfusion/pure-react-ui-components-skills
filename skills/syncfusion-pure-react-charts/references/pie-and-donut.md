# Pie and Donut Charts Reference

Use the Pure React pie-chart component family exported by `@syncfusion/react-charts` for pie and donut visualizations. Build slices declaratively from a bound data source, and do not reproduce pie geometry with custom SVG, Canvas, paths, stroke offsets, or manually calculated angles.

## Core rules

- Use `PieChart` as the root component.
- Place `PieChartSeries` inside `PieChartSeriesCollection`.
- Bind category and numeric fields through `xField` and `yField`.
- Place `PieChartDataLabel` inside its owning series.
- Place `PieChartTitle`, `PieChartSubtitle`, `PieChartLegend`, and `PieChartTooltip` directly inside `PieChart`.
- Create a donut by setting the series `innerRadius` above `0%`.
- Use pie or donut only for part-to-whole data with one meaningful total.
- Prefer a bar chart when exact ranking or comparison is the primary task.
- Do not use Cartesian `Chart`, axes, Cartesian series, trendlines, or striplines for pie or donut requests.

## Component hierarchy

```text
PieChart
├── PieChartTitle
├── PieChartSubtitle
├── PieChartLegend
├── PieChartTooltip
└── PieChartSeriesCollection
    └── PieChartSeries
        └── PieChartDataLabel
```

Keep every `PieChartSeries` inside `PieChartSeriesCollection`.

## Minimal pie chart

```tsx
import {
  PieChart,
  PieChartDataLabel,
  PieChartLegend,
  PieChartSeries,
  PieChartSeriesCollection,
  PieChartSubtitle,
  PieChartTitle,
  PieChartTooltip,
} from "@syncfusion/react-charts";

interface BrowserShare {
  browser: string;
  share: number;
}

const data: BrowserShare[] = [
  { browser: "Chrome", share: 63 },
  { browser: "Safari", share: 19 },
  { browser: "Edge", share: 8 },
  { browser: "Firefox", share: 6 },
  { browser: "Other", share: 4 },
];

export default function BrowserSharePie() {
  return (
    <PieChart
      width="100%"
      height="420px"
      accessibility={{
        ariaLabel: "Browser usage share",
        role: "img",
        focusable: true,
        tabIndex: 0,
      }}
    >
      <PieChartTitle text="Usage by browser" />
      <PieChartSubtitle text="Share of total usage" />

      <PieChartLegend
        visible={true}
        position="Bottom"
      />

      <PieChartTooltip
        enable={true}
        format="${point.x}: ${point.y}%"
      />

      <PieChartSeriesCollection>
        <PieChartSeries
          dataSource={data}
          xField="browser"
          yField="share"
          name="Browsers"
        >
          <PieChartDataLabel
            visible={true}
            position="Outside"
            format="{value}%"
          />
        </PieChartSeries>
      </PieChartSeriesCollection>
    </PieChart>
  );
}
```

Data labels and tooltips are disabled by default. Enable them only when required.

## Minimal donut chart

Use the same component family and set `innerRadius` on `PieChartSeries`.

```tsx
export default function BrowserShareDonut() {
  return (
    <PieChart width="100%" height="420px">
      <PieChartTitle text="Usage by browser" />

      <PieChartLegend
        visible={true}
        position="Bottom"
      />

      <PieChartTooltip
        enable={true}
        format="${point.x}: ${point.y}%"
      />

      <PieChartSeriesCollection>
        <PieChartSeries
          dataSource={data}
          xField="browser"
          yField="share"
          name="Browsers"
          innerRadius="58%"
          radius="82%"
        >
          <PieChartDataLabel
            visible={true}
            position="Outside"
            format="{value}%"
          />
        </PieChartSeries>
      </PieChartSeriesCollection>
    </PieChart>
  );
}
```

`innerRadius="0%"` renders a pie. Any meaningful value above `0%` creates a donut opening. Keep `innerRadius` smaller than `radius`.

## Data requirements

A pie or donut source should contain one category and one finite numeric value per slice.

```tsx
interface SlicePoint {
  category: string;
  value: number;
}
```

Validate that:

- category labels are non-empty
- values are finite numbers
- values are non-negative
- the total is greater than zero
- every slice belongs to the same total and period

```tsx
function isValidPieData(value: unknown): value is SlicePoint[] {
  if (!Array.isArray(value) || value.length === 0) {
    return false;
  }

  const validPoints = value.every((item) => {
    if (typeof item !== "object" || item === null) {
      return false;
    }

    const point = item as Record<string, unknown>;
    return (
      typeof point.category === "string" &&
      point.category.trim().length > 0 &&
      typeof point.value === "number" &&
      Number.isFinite(point.value) &&
      point.value >= 0
    );
  });

  if (!validPoints) {
    return false;
  }

  return value.some(
    (item) => (item as SlicePoint).value > 0,
  );
}
```

Do not plot formatted strings such as `"42%"` or `"₹1,250"` as `yField` values. Keep slice values numeric and format only displayed text.

## Part-to-whole suitability

Use pie or donut when all of these are true:

1. The values form one meaningful total.
2. The user needs a broad composition view.
3. There are relatively few slices.
4. Negative values are not present.
5. Slice order and labeling are understandable.

Use a sorted bar chart instead when:

- users must compare close values precisely
- there are many categories
- labels are long
- values do not form one total
- ranking is the main task
- the chart contains a long tail of tiny slices

## Slice colors

Use a mapped color field or the series `palettes` property. Do not calculate slice geometry or build custom paths.

```tsx
const data = [
  { category: "Product A", value: 42, color: "#005A9C" },
  { category: "Product B", value: 27, color: "#107C10" },
  { category: "Product C", value: 18, color: "#A4262C" },
  { category: "Other", value: 13, color: "#5C2D91" },
];

<PieChartSeries
  dataSource={data}
  xField="category"
  yField="value"
  colorField="color"
/>
```

Or:

```tsx
<PieChartSeries
  dataSource={data}
  xField="category"
  yField="value"
  palettes={[
    "#005A9C",
    "#107C10",
    "#A4262C",
    "#5C2D91",
  ]}
/>
```

A palette does not automatically guarantee accessible contrast. Use labels, patterns, or another non-color cue when slice distinction is essential.

## Patterns

Use `applyPattern` when slices need an additional visual distinction beyond color.

```tsx
<PieChartSeries
  dataSource={data}
  xField="category"
  yField="value"
  applyPattern={true}
/>
```

Test the resulting patterns at the chart's actual size and contrast settings.

## Data labels

Place `PieChartDataLabel` inside the series.

```tsx
<PieChartSeries
  dataSource={data}
  xField="category"
  yField="value"
>
  <PieChartDataLabel
    visible={true}
    position="Outside"
    name="category"
    maxLabelWidth={120}
    connectorStyle={{
      type: "Curve",
      width: 1,
      color: "#666666",
      dashArray: "",
      length: "8%",
    }}
  />
</PieChartSeries>
```

Use:

- `Inside` when slices are large and text contrast is sufficient
- `Outside` when labels need more room or slices are smaller
- `name` to map a data-source field as label content
- `format` for numeric display formats
- `maxLabelWidth` for constrained labels
- `connectorStyle` for outside-label connectors

Data labels default to hidden, and their default position is `Inside`.

## Smart labels

`PieChart.smartLabels` defaults to `true` and helps position labels to reduce overlap.

```tsx
<PieChart smartLabels={true}>
  {/* chart children */}
</PieChart>
```

Smart labels cannot make an excessive number of slices readable. Group small values or switch to a bar chart when the composition is too dense.

## Group small slices

Use `groupTo` and `groupMode` to combine smaller points into an `Others` slice.

```tsx
<PieChartSeries
  dataSource={data}
  xField="category"
  yField="value"
  groupMode="Value"
  groupTo="3"
/>
```

`Value` groups according to Y values. `Point` groups according to point index. Confirm the installed package's threshold interpretation and test the result with the actual dataset.

Group only when combining small categories does not hide information users must inspect individually.

## Legend

Place `PieChartLegend` directly inside `PieChart`.

```tsx
<PieChartLegend
  visible={true}
  position="Bottom"
  align="Center"
  enablePages={true}
  maxLabelWidth={160}
  toggleVisibility={true}
/>
```

The legend defaults to visible, and `toggleVisibility` defaults to `true`. Supported positions include `Auto`, `Top`, `Left`, `Bottom`, `Right`, and `Custom`.

Use a bottom legend for many responsive layouts. If the legend becomes longer than the chart itself, reconsider the chart family or group small slices.

## Tooltip

Place `PieChartTooltip` directly inside `PieChart`.

```tsx
<PieChartTooltip
  enable={true}
  format="${point.x}: ${point.y}"
  followPointer={true}
/>
```

Use a formatter or JSX template when the built-in format cannot produce the required localized content.

```tsx
<PieChartTooltip
  enable={true}
  template={(props) => (
    <div style={{ padding: "8px" }}>
      <strong>{String(props.x)}</strong>
      <div>{numberFormatter.format(Number(props.y))}</div>
    </div>
  )}
/>
```

Do not make the tooltip the only place where essential slice meaning is available.

## Exploded slices

Use package properties for exploded slices.

```tsx
<PieChartSeries
  dataSource={data}
  xField="category"
  yField="value"
  explode={true}
  explodeIndex={0}
  explodeOffset="12%"
/>
```

Use explosion sparingly. Exploding every slice usually weakens the part-to-whole structure.

## Start and end angles

Use package-provided angle properties only when a partial-circle or rotated composition is explicitly required.

```tsx
<PieChartSeries
  dataSource={data}
  xField="category"
  yField="value"
  startAngle={-90}
  endAngle={270}
/>
```

Do not calculate individual slice angles. The chart computes slice geometry from `yField` values.

## Radius and center

Use root `center` and series `radius` or `innerRadius` for layout adjustments.

```tsx
<PieChart
  center={{ x: "50%", y: "48%" }}
>
  <PieChartSeriesCollection>
    <PieChartSeries
      dataSource={data}
      xField="category"
      yField="value"
      radius="80%"
      innerRadius="55%"
    />
  </PieChartSeriesCollection>
</PieChart>
```

Do not use absolute-positioned slice elements to imitate center or radius changes.

## Responsive layout

Give the chart a predictable container height.

```tsx
<div className="pie-card__canvas">
  <PieChart width="100%" height="100%">
    {/* chart children */}
  </PieChart>
</div>
```

```css
.pie-card__canvas {
  width: 100%;
  height: 420px;
  min-width: 0;
}

@media (max-width: 640px) {
  .pie-card__canvas {
    height: 340px;
  }
}
```

When `height="100%"` is used, the parent must have a resolved height.

## Accessibility

Configure accessibility on the root, series, and legend where needed.

```tsx
<PieChart
  accessibility={{
    ariaLabel:
      "Browser usage share. Five slices. A data table follows.",
    role: "region",
    focusable: true,
    tabIndex: 0,
  }}
  focusOutline={{
    color: "#005FCC",
    width: 2,
    offset: 2,
  }}
>
  <PieChartLegend
    accessibility={{
      ariaLabel: "Browser categories",
      focusable: true,
      tabIndex: 0,
    }}
  />

  <PieChartSeriesCollection>
    <PieChartSeries
      dataSource={data}
      xField="browser"
      yField="share"
      accessibility={{
        ariaLabel: "Browser usage slices",
        role: "img",
        focusable: true,
        tabIndex: 0,
      }}
    />
  </PieChartSeriesCollection>
</PieChart>
```

Also provide a text summary or synchronized data table for precise values.

```tsx
<table>
  <caption>Browser usage share</caption>
  <thead>
    <tr>
      <th scope="col">Browser</th>
      <th scope="col">Share</th>
    </tr>
  </thead>
  <tbody>
    {data.map((item) => (
      <tr key={item.browser}>
        <th scope="row">{item.browser}</th>
        <td>{item.share}%</td>
      </tr>
    ))}
  </tbody>
</table>
```

## Events

Use event props documented by `PieChart`, not Cartesian event types.

```tsx
import type {
  PieLegendClickEvent,
  PiePointClickEvent,
} from "@syncfusion/react-charts";

const handlePointClick = (
  event: PiePointClickEvent,
): void => {
  console.log(event.pointIndex, event.seriesIndex);
};

const handleLegendClick = (
  event: PieLegendClickEvent,
): void => {
  console.log(event.text);
};

<PieChart
  onPointClick={handlePointClick}
  onLegendClick={handleLegendClick}
>
  {/* pie chart children */}
</PieChart>
```

Do not type pie events with `PointClickEvent` or `LegendClickEvent` from the Cartesian chart family.

## Performance and density

- Keep slice counts low enough for labels and colors to remain distinguishable.
- Avoid expensive templates on a large number of slices.
- Disable animation when profiling shows it is costly or when reduced motion is required.
- Group small slices only when the loss of detail is acceptable.
- Avoid exploding many slices.
- Prefer a bar chart when many categories must remain individually readable.

## Common errors

### Cartesian root

Incorrect:

```tsx
<Chart>
  <ChartSeriesCollection>
    <ChartSeries type="Pie" />
  </ChartSeriesCollection>
</Chart>
```

Correct:

```tsx
<PieChart>
  <PieChartSeriesCollection>
    <PieChartSeries
      dataSource={data}
      xField="category"
      yField="value"
    />
  </PieChartSeriesCollection>
</PieChart>
```

### Manual slice geometry

Do not use custom SVG paths, Canvas arcs, conic gradients, stroke-dash offsets, or manually computed angles to recreate functionality already provided by `PieChart`.

### Missing series collection

Incorrect:

```tsx
<PieChart>
  <PieChartSeries dataSource={data} />
</PieChart>
```

Correct:

```tsx
<PieChart>
  <PieChartSeriesCollection>
    <PieChartSeries
      dataSource={data}
      xField="category"
      yField="value"
    />
  </PieChartSeriesCollection>
</PieChart>
```

### Donut without innerRadius

Use `innerRadius` on `PieChartSeries`. Do not use a separate Cartesian type or cover the center with an absolutely positioned circle.

### Labels outside the series

Incorrect:

```tsx
<PieChart>
  <PieChartDataLabel visible={true} />
</PieChart>
```

Correct:

```tsx
<PieChartSeries>
  <PieChartDataLabel visible={true} />
</PieChartSeries>
```

### Too many slices

Do not reduce text size until dozens of slices fit. Group minor points when appropriate or change to a sorted bar chart.

## Validation checklist

Before returning a pie or donut implementation:

1. Use `PieChart` as the root.
2. Import components from `@syncfusion/react-charts`.
3. Place each `PieChartSeries` inside `PieChartSeriesCollection`.
4. Bind the data array directly through `dataSource`.
5. Map `xField` to the category and `yField` to a finite numeric value.
6. Reject or normalize invalid, negative, or all-zero values.
7. Confirm the values represent one meaningful total.
8. Use `innerRadius` to create a donut.
9. Place data labels inside their owning series.
10. Place title, subtitle, legend, and tooltip directly inside `PieChart`.
11. Use `colorField`, `palettes`, or `applyPattern` instead of custom slice rendering.
12. Use package properties for explosion, angles, radius, and center.
13. Keep slice count low enough for meaningful comparison.
14. Prefer a bar chart when precise ranking is required.
15. Provide accessible names and a visible focus outline.
16. Do not rely only on color or hover.
17. Provide a text summary or table for precise values when needed.
18. Use pie-specific event types.
19. Give the chart a predictable responsive size.
20. Do not mix Cartesian, EJ2, or handcrafted pie geometry with the Pure React pie family.
21. Ensure every imported symbol is used.
22. Emit valid, compact, unescaped TSX.
