# Multiple Panes Reference

Use multiple panes when related series must render in separate plot regions while remaining in one chart with shared context.

## Pane model

A chart can be divided in two directions:

- Rows divide the chart area vertically into stacked panes.
- Columns divide the chart area horizontally into side-by-side panes.

Use `ChartRow` to define a row and `ChartColumn` to define a column. Assign axes to panes with `rowIndex` or `columnIndex`.

- Assign a vertical axis to a row with `rowIndex`.
- Assign a horizontal axis to a column with `columnIndex`.
- Index values are zero-based.
- The documented default `columnIndex` is `0`.
- An axis assigned to an index requires a corresponding row or column at that index.

Do not assign a pane index directly to `ChartSeries`. Series enter a pane through the axis to which the series is mapped.

## Row properties

The official row API documents:

- `height: string`, default `"100%"`
- `border: { color, width, dashArray }`, with default width `1`

`height` accepts percentage or pixel values.

```tsx
<ChartRow
  height="70%"
  border={{ color: "#D9D9D9", width: 1, dashArray: "" }}
/>
<ChartRow
  height="30%"
  border={{ color: "#D9D9D9", width: 1, dashArray: "" }}
/>
```

Ensure the row heights form a sensible complete layout. Avoid ambiguous mixtures of percentages and pixels unless the container height is fixed and the remaining space is intentional.

## Column properties

Use `width` to allocate each `ChartColumn`. The width can be a percentage or pixel string.

```tsx
<ChartColumn width="60%" />
<ChartColumn width="40%" />
```

Ensure column widths form a sensible complete layout. A fixed pixel width is appropriate only when the chart container provides enough predictable space.

## Vertical split with rows

For stacked panes, define rows and map each Y-axis to the correct `rowIndex`.

```tsx
import {
  Chart,
  ChartAxes,
  ChartAxis,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartRow,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

<Chart>
  <ChartRow height="70%" />
  <ChartRow height="30%" />

  <ChartPrimaryXAxis valueType="DateTime" />

  <ChartPrimaryYAxis rowIndex={0}>
    <ChartAxisTitle text="Price" />
    <ChartAxisLabel format="C0" />
  </ChartPrimaryYAxis>

  <ChartAxes>
    <ChartAxis name="volumeAxis" rowIndex={1}>
      <ChartAxisTitle text="Volume" />
      <ChartAxisLabel format="{value}" />
    </ChartAxis>
  </ChartAxes>

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="date"
      yField="price"
      type="Line"
      name="Price"
    />
    <ChartSeries
      dataSource={data}
      xField="date"
      yField="volume"
      type="Column"
      name="Volume"
      yAxisName="volumeAxis"
    />
  </ChartSeriesCollection>
</Chart>
```

The price series uses the primary Y-axis in row `0`. The volume series maps to `volumeAxis`, which is assigned to row `1`.

## Horizontal split with columns

For side-by-side panes, define columns and map each X-axis to the correct `columnIndex`.

```tsx
import {
  Chart,
  ChartAxes,
  ChartAxis,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartColumn,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

<Chart>
  <ChartColumn width="50%" />
  <ChartColumn width="50%" />

  <ChartPrimaryXAxis
    valueType="Category"
    columnIndex={0}
  >
    <ChartAxisTitle text="Region" />
  </ChartPrimaryXAxis>

  <ChartAxes>
    <ChartAxis
      name="productAxis"
      valueType="Category"
      columnIndex={1}
    >
      <ChartAxisTitle text="Product" />
      <ChartAxisLabel edgeLabelPlacement="Shift" />
    </ChartAxis>
  </ChartAxes>

  <ChartPrimaryYAxis valueType="Double" />

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={regionData}
      xField="region"
      yField="value"
      type="Column"
      name="Regional Sales"
    />
    <ChartSeries
      dataSource={productData}
      xField="product"
      yField="value"
      type="Column"
      name="Product Sales"
      xAxisName="productAxis"
    />
  </ChartSeriesCollection>
</Chart>
```

The first series uses the primary X-axis in column `0`. The second series maps to `productAxis`, which is assigned to column `1`.

## Pane assignment through named axes

Use the same naming contract as multiple-axis charts:

```tsx
<ChartAxes>
  <ChartAxis name="secondaryAxis" rowIndex={1} />
</ChartAxes>

<ChartSeries yAxisName="secondaryAxis" />
```

Rules:

- Every additional axis needs a unique `name`.
- The series `xAxisName` or `yAxisName` must match that name exactly.
- `rowIndex` belongs to an axis used vertically.
- `columnIndex` belongs to an axis used horizontally.
- A series without an additional-axis mapping uses the applicable primary axis and therefore the primary axis pane.

## Shared X-axis with stacked Y panes

A common monitoring layout uses one DateTime X-axis and separate Y-axis scales.

```tsx
const data = [
  { time: new Date(2026, 8, 7, 10, 0), cpu: 42, memory: 68 },
  { time: new Date(2026, 8, 7, 10, 5), cpu: 57, memory: 72 },
  { time: new Date(2026, 8, 7, 10, 10), cpu: 49, memory: 75 },
];

<Chart>
  <ChartRow height="50%" />
  <ChartRow height="50%" />

  <ChartPrimaryXAxis
    valueType="DateTime"
    interval={5}
    intervalType="Minutes"
  >
    <ChartAxisLabel format="hm" />
  </ChartPrimaryXAxis>

  <ChartPrimaryYAxis
    rowIndex={0}
    minimum={0}
    maximum={100}
  >
    <ChartAxisTitle text="CPU (%)" />
    <ChartAxisLabel format="{value}%" />
  </ChartPrimaryYAxis>

  <ChartAxes>
    <ChartAxis
      name="memoryAxis"
      rowIndex={1}
      minimum={0}
      maximum={100}
    >
      <ChartAxisTitle text="Memory (%)" />
      <ChartAxisLabel format="{value}%" />
    </ChartAxis>
  </ChartAxes>

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="time"
      yField="cpu"
      type="Line"
      name="CPU"
    />
    <ChartSeries
      dataSource={data}
      xField="time"
      yField="memory"
      type="Line"
      name="Memory"
      yAxisName="memoryAxis"
    />
  </ChartSeriesCollection>
</Chart>
```

Use compatible X values across panes when the purpose is synchronized time comparison.

## Multiple rows and multiple columns

A chart may define both rows and columns. Every axis assignment must still be explicit.

```tsx
<Chart>
  <ChartRow height="60%" />
  <ChartRow height="40%" />

  <ChartColumn width="65%" />
  <ChartColumn width="35%" />

  <ChartAxes>
    <ChartAxis
      name="bottomAxis"
      rowIndex={1}
    />
    <ChartAxis
      name="rightAxis"
      columnIndex={1}
    />
  </ChartAxes>
</Chart>
```

Do not create a complex grid unless each pane has a clear analytical purpose. Multiple panes share one chart canvas and should retain a coherent comparison context.

## Styling pane boundaries

Use the row `border` configuration when a visible separator is required.

```tsx
<ChartRow
  height="65%"
  border={{
    color: "#D9D9D9",
    width: 1,
    dashArray: "",
  }}
/>
```

Keep separators subtle enough that the data remains visually dominant.

For columns, use only the border or separator properties documented by the current `ChartColumn` API. Do not copy `ChartRow` properties to `ChartColumn` unless the column API confirms them.

## Complete two-pane financial example

```tsx
import {
  Chart,
  ChartAxes,
  ChartAxis,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartRow,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { date: new Date(2026, 7, 3), close: 112, volume: 185000 },
  { date: new Date(2026, 7, 4), close: 118, volume: 212000 },
  { date: new Date(2026, 7, 5), close: 115, volume: 198000 },
  { date: new Date(2026, 7, 6), close: 123, volume: 247000 },
  { date: new Date(2026, 7, 7), close: 127, volume: 231000 },
];

export default function MultiplePanesChart() {
  return (
    <Chart>
      <ChartRow
        height="70%"
        border={{ color: "#D9D9D9", width: 1, dashArray: "" }}
      />
      <ChartRow
        height="30%"
        border={{ color: "#D9D9D9", width: 1, dashArray: "" }}
      />

      <ChartPrimaryXAxis
        valueType="DateTime"
        interval={1}
        intervalType="Days"
      >
        <ChartAxisTitle text="Date" />
        <ChartAxisLabel format="yMd" edgeLabelPlacement="Shift" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis rowIndex={0} valueType="Double">
        <ChartAxisTitle text="Closing Price" />
        <ChartAxisLabel format="C0" />
      </ChartPrimaryYAxis>

      <ChartAxes>
        <ChartAxis
          name="volumeAxis"
          rowIndex={1}
          valueType="Double"
          minimum={0}
        >
          <ChartAxisTitle text="Volume" />
          <ChartAxisLabel format="{value}" />
        </ChartAxis>
      </ChartAxes>

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="date"
          yField="close"
          type="Line"
          name="Close"
          width={2}
        />
        <ChartSeries
          dataSource={data}
          xField="date"
          yField="volume"
          type="Column"
          name="Volume"
          yAxisName="volumeAxis"
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Common errors

### Assigning a pane directly to a series

Incorrect:

```tsx
<ChartSeries rowIndex={1} />
```

Correct:

```tsx
<ChartAxis name="volumeAxis" rowIndex={1} />
<ChartSeries yAxisName="volumeAxis" />
```

### Referring to a row that does not exist

Incorrect:

```tsx
<ChartRow height="100%" />
<ChartAxis name="secondaryAxis" rowIndex={1} />
```

Only row index `0` exists in this example. Add another `ChartRow` or use `rowIndex={0}`.

### Referring to a column that does not exist

Incorrect:

```tsx
<ChartColumn width="100%" />
<ChartAxis name="secondaryXAxis" columnIndex={1} />
```

Only column index `0` exists.

### Confusing row and column assignment

Use `rowIndex` for vertical-axis pane assignment and `columnIndex` for horizontal-axis pane assignment. Do not swap these properties.

### Inventing secondary axis tags

Incorrect:

```tsx
<ChartSecondaryYAxis rowIndex={1} />
```

Correct:

```tsx
<ChartAxes>
  <ChartAxis name="secondaryYAxis" rowIndex={1} />
</ChartAxes>
```

### Omitting the series-axis mapping

Defining an additional pane axis does not automatically move a series into that pane. Map the series with the matching axis name.

## Validation checklist

Before returning a multiple-pane implementation:

1. Import the required pane, axis, and series components from `@syncfusion/react-charts`.
2. Define one `ChartRow` for every row index used by a vertical axis.
3. Define one `ChartColumn` for every column index used by a horizontal axis.
4. Use valid percentage or pixel strings for row heights and column widths.
5. Use zero-based `rowIndex` and `columnIndex` values.
6. Assign vertical axes with `rowIndex`.
7. Assign horizontal axes with `columnIndex`.
8. Place additional axes inside `ChartAxes`.
9. Give every additional axis a unique name.
10. Match the series `xAxisName` or `yAxisName` exactly to the additional axis name.
11. Ensure every pane index refers to an existing row or column.
12. Ensure each axis value type matches its mapped data.
13. Keep shared X data compatible when panes are intended for synchronized comparison.
14. Do not assign pane indexes directly to series.
15. Do not invent secondary-axis component tags.
16. Emit valid, unescaped TSX.
