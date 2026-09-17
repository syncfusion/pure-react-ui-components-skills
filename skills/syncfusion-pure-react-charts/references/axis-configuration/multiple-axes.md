# Multiple Axes Reference

Use multiple axes when chart series represent different units, ranges, or scale types that cannot be interpreted accurately on one shared axis.

## Component hierarchy

Keep `ChartPrimaryXAxis` and `ChartPrimaryYAxis` for the primary axes. Define every additional axis as a named `ChartAxis` inside `ChartAxes`.

Map a series to an additional axis by setting the series `xAxisName` or `yAxisName` to the exact value of the axis `name`.

```tsx
import {
  Chart,
  ChartAxes,
  ChartAxis,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

<Chart>
  <ChartPrimaryXAxis valueType="Category" />
  <ChartPrimaryYAxis>
    <ChartAxisTitle text="Revenue" />
  </ChartPrimaryYAxis>

  <ChartAxes>
    <ChartAxis name="growthAxis" opposedPosition={true}>
      <ChartAxisTitle text="Growth (%)" />
      <ChartAxisLabel format="{value}%" />
    </ChartAxis>
  </ChartAxes>

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="month"
      yField="revenue"
      type="Column"
      name="Revenue"
    />
    <ChartSeries
      dataSource={data}
      xField="month"
      yField="growth"
      type="Line"
      name="Growth"
      yAxisName="growthAxis"
    />
  </ChartSeriesCollection>
</Chart>
```

Do not use invented component tags such as `ChartSecondaryXAxis` or `ChartSecondaryYAxis`.

## Axis naming and series mapping

Every additional axis requires a unique, non-empty `name`.

```tsx
<ChartAxes>
  <ChartAxis name="temperatureAxis" />
  <ChartAxis name="rainfallAxis" opposedPosition={true} />
</ChartAxes>
```

Map series using matching names:

```tsx
<ChartSeries yAxisName="temperatureAxis" />
<ChartSeries yAxisName="rainfallAxis" />
```

Mapping rules:

- `yAxisName` maps a series to a named Y-axis.
- `xAxisName` maps a series to a named X-axis.
- The name comparison is case-sensitive.
- A missing or mismatched name prevents the series from using the intended additional axis.
- A series that does not specify an axis name uses the applicable primary axis.
- Do not assign a primary-axis alias such as `"PrimaryAxis"` unless an axis with that exact public name has been explicitly defined and verified.

## Additional Y-axis

Use an additional Y-axis when series share an X domain but use different measurement units or numeric ranges.

```tsx
const data = [
  { month: "Jan", revenue: 42000, growth: 8.4 },
  { month: "Feb", revenue: 51000, growth: 12.1 },
  { month: "Mar", revenue: 47000, growth: 6.8 },
  { month: "Apr", revenue: 59000, growth: 14.5 },
];

<Chart>
  <ChartPrimaryXAxis valueType="Category">
    <ChartAxisTitle text="Month" />
  </ChartPrimaryXAxis>

  <ChartPrimaryYAxis
    minimum={0}
    maximum={70000}
    interval={10000}
  >
    <ChartAxisTitle text="Revenue" />
    <ChartAxisLabel format="C0" />
  </ChartPrimaryYAxis>

  <ChartAxes>
    <ChartAxis
      name="growthAxis"
      minimum={0}
      maximum={20}
      interval={5}
      opposedPosition={true}
    >
      <ChartAxisTitle text="Growth (%)" />
      <ChartAxisLabel format="{value}%" />
    </ChartAxis>
  </ChartAxes>

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="month"
      yField="revenue"
      type="Column"
      name="Revenue"
    />
    <ChartSeries
      dataSource={data}
      xField="month"
      yField="growth"
      type="Line"
      name="Growth"
      yAxisName="growthAxis"
    />
  </ChartSeriesCollection>
</Chart>
```

Use `opposedPosition={true}` to render the additional Y-axis on the side opposite the primary Y-axis.

## Additional X-axis

Use a named X-axis when a series requires a different X scale or value type.

```tsx
<Chart>
  <ChartPrimaryXAxis valueType="Category" />

  <ChartAxes>
    <ChartAxis
      name="dateAxis"
      valueType="DateTime"
      opposedPosition={true}
      interval={1}
      intervalType="Months"
    >
      <ChartAxisTitle text="Date" />
      <ChartAxisLabel format="MMM" />
    </ChartAxis>
  </ChartAxes>

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={categoryData}
      xField="category"
      yField="value"
      type="Column"
      name="Category Values"
    />
    <ChartSeries
      dataSource={timeData}
      xField="date"
      yField="value"
      type="Line"
      name="Timeline"
      xAxisName="dateAxis"
    />
  </ChartSeriesCollection>
</Chart>
```

The mapped `xField` values must be compatible with the selected X-axis `valueType`.

## More than two axes

`ChartAxes` may contain multiple named axes. Keep each name unique and map each series deliberately.

```tsx
<ChartAxes>
  <ChartAxis
    name="temperatureAxis"
    opposedPosition={false}
  >
    <ChartAxisTitle text="Temperature (°C)" />
    <ChartAxisLabel format="{value}°C" />
  </ChartAxis>

  <ChartAxis
    name="humidityAxis"
    opposedPosition={true}
  >
    <ChartAxisTitle text="Humidity (%)" />
    <ChartAxisLabel format="{value}%" />
  </ChartAxis>
</ChartAxes>

<ChartSeriesCollection>
  <ChartSeries
    dataSource={weatherData}
    xField="time"
    yField="pressure"
    type="Line"
    name="Pressure"
  />
  <ChartSeries
    dataSource={weatherData}
    xField="time"
    yField="temperature"
    type="Line"
    name="Temperature"
    yAxisName="temperatureAxis"
  />
  <ChartSeries
    dataSource={weatherData}
    xField="time"
    yField="humidity"
    type="Line"
    name="Humidity"
    yAxisName="humidityAxis"
  />
</ChartSeriesCollection>
```

Avoid unnecessary axes. Multiple scales can make comparisons misleading when axis ranges, units, and colors are unclear.

## Additional-axis properties

A named `ChartAxis` supports the same core axis configuration model used by the primary axes. Relevant properties include:

- `name`: unique axis identifier; default `""`
- `valueType`: `Double`, `DateTime`, `Category`, or `Logarithmic`
- `minimum`: explicit minimum value
- `maximum`: explicit maximum value
- `interval`: interval between labels or ticks
- `intervalType`: DateTime interval unit
- `desiredIntervals`: requested approximate interval count
- `rangePadding`: axis range-padding mode
- `opposedPosition`: render on the opposite side
- `inverted`: reverse the axis direction; default `false`
- `rowIndex`: pane row assignment
- `columnIndex`: pane column assignment; default `0`
- `crossAt`: axis-crossing configuration
- `lineStyle`: axis-line appearance
- `minorTicksPerInterval`: minor intervals; default `0`
- `maxLabelDensity`: maximum labels per 100 pixels; default `3`

Place child configuration such as `ChartAxisTitle`, `ChartAxisLabel`, grid lines, tick lines, and striplines inside the named axis.

```tsx
<ChartAxis
  name="secondaryScale"
  valueType="Double"
  opposedPosition={true}
  inverted={false}
  lineStyle={{ color: "#E67E22", width: 1, dashArray: "" }}
>
  <ChartAxisTitle text="Secondary scale" color="#E67E22" />
  <ChartAxisLabel color="#E67E22" format="{value}" />
  <ChartMajorGridLines width={0} />
  <ChartMajorTickLines width={1} height={6} color="#E67E22" />
</ChartAxis>
```

## Multiple axes with different value types

Each axis must match its series data.

```tsx
<ChartAxes>
  <ChartAxis
    name="logAxis"
    valueType="Logarithmic"
    logBase={10}
    minimum={1}
    maximum={10000}
    opposedPosition={true}
  >
    <ChartAxisTitle text="Logarithmic value" />
  </ChartAxis>
</ChartAxes>
```

A series mapped to `logAxis` must contain positive numeric Y values. A series mapped to a DateTime X-axis must contain date-compatible X values.

## Multiple axes and panes

Use `rowIndex` for a vertical axis assigned to a chart row and `columnIndex` for a horizontal axis assigned to a chart column.

```tsx
<ChartAxis
  name="volumeAxis"
  rowIndex={1}
>
  <ChartAxisTitle text="Volume" />
</ChartAxis>
```

The corresponding row or column must exist in the chart's pane collection. For complete pane hierarchy, use the dedicated multiple-panes reference.

Do not use panes merely to simulate additional axes. Use panes when series need separate plot regions; use additional axes when series share a plot region but require different scales.

## Axis crossing with named axes

Use `crossAt.axis` to identify the target named axis.

```tsx
<ChartAxis
  name="secondaryYAxis"
  crossAt={{
    value: 0,
    axis: "primaryXAxis",
    allowOverlap: false,
  }}
/>
```

The target name must identify the actual axis that should be crossed, and `value` must be compatible with that target axis.

## Complete multiple-axis example

```tsx
import {
  Chart,
  ChartAxes,
  ChartAxis,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartMajorGridLines,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { month: "Jan", revenue: 42000, growth: 8.4 },
  { month: "Feb", revenue: 51000, growth: 12.1 },
  { month: "Mar", revenue: 47000, growth: 6.8 },
  { month: "Apr", revenue: 59000, growth: 14.5 },
  { month: "May", revenue: 64000, growth: 16.2 },
];

export default function MultipleAxesChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis valueType="Category">
        <ChartAxisTitle text="Month" />
        <ChartAxisLabel edgeLabelPlacement="Shift" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis
        valueType="Double"
        minimum={0}
        maximum={70000}
        interval={10000}
      >
        <ChartAxisTitle text="Revenue" color="#2E86DE" />
        <ChartAxisLabel format="C0" color="#2E86DE" />
        <ChartMajorGridLines width={1} color="#E5E5E5" />
      </ChartPrimaryYAxis>

      <ChartAxes>
        <ChartAxis
          name="growthAxis"
          valueType="Double"
          minimum={0}
          maximum={20}
          interval={5}
          opposedPosition={true}
          lineStyle={{ color: "#E67E22", width: 1, dashArray: "" }}
        >
          <ChartAxisTitle text="Growth (%)" color="#E67E22" />
          <ChartAxisLabel format="{value}%" color="#E67E22" />
          <ChartMajorGridLines width={0} />
        </ChartAxis>
      </ChartAxes>

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="revenue"
          type="Column"
          name="Revenue"
          fill="#2E86DE"
        />
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="growth"
          type="Line"
          name="Growth"
          yAxisName="growthAxis"
          fill="#E67E22"
          width={2}
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Common errors

### Inventing a secondary-axis component

Incorrect:

```tsx
<ChartSecondaryYAxis />
```

Correct:

```tsx
<ChartAxes>
  <ChartAxis name="secondaryYAxis" />
</ChartAxes>
```

### Using `yAxisName` on the axis

Incorrect:

```tsx
<ChartAxis yAxisName="growthAxis" />
```

Correct:

```tsx
<ChartAxis name="growthAxis" />
<ChartSeries yAxisName="growthAxis" />
```

### Name mismatch

Incorrect:

```tsx
<ChartAxis name="growthAxis" />
<ChartSeries yAxisName="GrowthAxis" />
```

Correct:

```tsx
<ChartAxis name="growthAxis" />
<ChartSeries yAxisName="growthAxis" />
```

### Mapping incompatible data

Do not map negative or zero values to a named logarithmic axis. Do not map category strings to an axis configured as `DateTime`.

### Duplicating names

Do not give two additional axes the same `name`. Series mapping becomes ambiguous and the intended scale cannot be identified reliably.

## Validation checklist

Before returning a multiple-axis implementation:

1. Import `ChartAxes` and `ChartAxis` from `@syncfusion/react-charts`.
2. Keep primary axes in `ChartPrimaryXAxis` and `ChartPrimaryYAxis`.
3. Place every additional `ChartAxis` inside `ChartAxes`.
4. Give every additional axis a unique, non-empty `name`.
5. Map each intended series with the matching `xAxisName` or `yAxisName`.
6. Match axis and series names exactly, including casing.
7. Ensure each axis `valueType` matches the mapped data.
8. Ensure explicit minimum, maximum, and interval values fit the intended unit and range.
9. Use `opposedPosition` to separate axes visually when appropriate.
10. Keep axis titles and labels explicit about units.
11. Use consistent colors when associating a series with its axis.
12. Use `rowIndex` and `columnIndex` only with configured panes.
13. Do not invent secondary-axis component tags.
14. Ensure all imports are used and all used components are imported.
15. Emit valid, unescaped TSX.
