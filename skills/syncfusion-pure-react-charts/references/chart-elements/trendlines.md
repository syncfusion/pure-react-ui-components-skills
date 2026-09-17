# Trendlines Reference

Use trendlines to reveal general patterns, smooth short-term variation, or extend a fitted model beyond the observed data. Configure trendlines inside the series they analyze.

## Required component hierarchy

Place `ChartTrendline` inside `ChartTrendlineCollection`, and place the collection inside the owning `ChartSeries`.

```tsx
import {
  Chart,
  ChartSeries,
  ChartSeriesCollection,
  ChartTrendline,
  ChartTrendlineCollection,
} from "@syncfusion/react-charts";

<Chart>
  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="x"
      yField="y"
      type="Scatter"
      name="Observed"
    >
      <ChartTrendlineCollection>
        <ChartTrendline
          type="Linear"
          name="Linear trend"
        />
      </ChartTrendlineCollection>
    </ChartSeries>
  </ChartSeriesCollection>
</Chart>
```

Do not place `ChartTrendline` directly under `Chart`, `ChartSeriesCollection`, or `ChartSeries` without its collection.

## Supported trendline types

Use exact values from the public `TrendlineTypes` union:

- `Linear`
- `Exponential`
- `Polynomial`
- `Power`
- `Logarithmic`
- `MovingAverage`

The documented default is `Linear`.

```tsx
<ChartTrendline type="Linear" />
```

Use exact casing. Do not emit values such as `Moving Average`, `movingAverage`, or `Poly`.

## Verified trendline properties

The official `ChartTrendlineProps` API documents:

- `type`: fitted model; default `Linear`
- `name`: legend display name; default `""`
- `stroke`: line color; default `""`
- `width`: line width
- `dashArray`: dash pattern; default `""`
- `opacity`: value from `0` through `1`; default `1`
- `period`: moving-average period; default `2`
- `polynomialOrder`: polynomial degree; default `2`
- `forwardForecast`: forward extension in data periods; default `0`
- `backwardForecast`: backward extension in data periods; default `0`
- `intercept`: forced Y-intercept; default `null`
- `enableTooltip`: trendline tooltip state; default `true`
- `legendShape`: trendline legend symbol; default `SeriesType`
- `animation`: default `{ enable: true, duration: 1000, delay: 0 }`
- `accessibility`: trendline accessibility settings

A trendline is computed only when its parent series is visible. 

## Linear trendline

Use `Linear` for data that follows an approximately constant rate of increase or decrease.

```tsx
<ChartTrendlineCollection>
  <ChartTrendline
    type="Linear"
    name="Linear trend"
    stroke="#D32F2F"
    width={2}
  />
</ChartTrendlineCollection>
```

A linear trendline is a straight best-fit line. turn49search205

## Exponential trendline

Use `Exponential` for accelerating growth or decay.

```tsx
<ChartTrendline
  type="Exponential"
  name="Exponential fit"
/>
```

Use this model only when the source values support an exponential relationship. turn49search206

## Logarithmic trendline

Use `Logarithmic` when change is rapid initially and then levels off.

```tsx
<ChartTrendline
  type="Logarithmic"
  name="Logarithmic fit"
/>
```

Ensure the model's numeric-domain requirements are satisfied. In particular, logarithmic fitting requires compatible positive X-domain values. turn49search205

## Polynomial trendline

Use `Polynomial` for curved data with direction changes. Configure the degree with `polynomialOrder`.

```tsx
<ChartTrendline
  type="Polynomial"
  name="Quadratic trend"
  polynomialOrder={2}
/>
```

The documented default polynomial order is `2`. Use a sensible positive order relative to the number of source points. Avoid unnecessary high-order fits that follow noise rather than the underlying pattern. turn49search205

## Power trendline

Use `Power` for a power-law relationship.

```tsx
<ChartTrendline
  type="Power"
  name="Power fit"
/>
```

Ensure the data satisfies the model's numeric-domain requirements before using a power fit. turn49search206

## Moving-average trendline

Use `MovingAverage` to smooth short-term variation. Configure the averaging window with `period`.

```tsx
<ChartTrendline
  type="MovingAverage"
  name="6-period moving average"
  period={6}
/>
```

The documented default period is `2`. Use a positive integer and ensure the source series contains enough points for the selected period. Forecast and polynomial settings do not apply to a moving-average trendline unless the official API explicitly says otherwise. turn49search205

## Multiple trendlines on one series

Add multiple `ChartTrendline` children to one `ChartTrendlineCollection`.

```tsx
<ChartSeries
  dataSource={data}
  xField="x"
  yField="y"
  type="Scatter"
  name="Observed"
>
  <ChartTrendlineCollection>
    <ChartTrendline
      type="Linear"
      name="Linear"
      stroke="#1565C0"
      width={2}
    />
    <ChartTrendline
      type="Polynomial"
      name="Polynomial"
      polynomialOrder={2}
      stroke="#E65100"
      width={2}
      dashArray="5,5"
    />
  </ChartTrendlineCollection>
</ChartSeries>
```

Each trendline can have independent model and appearance settings. 

## Appearance

Use `stroke`, `width`, `dashArray`, and `opacity`.

```tsx
<ChartTrendline
  type="Linear"
  stroke="#FF5733"
  width={2}
  dashArray="5,5"
  opacity={0.9}
/>
```

Important corrections:

- Use `stroke`, not `fill`, for the trendline color.
- Keep opacity between `0` and `1`.
- Use a comma-separated SVG-style dash pattern.

Incorrect:

```tsx
<ChartTrendline
  fill="#FF5733"
/>
```

Correct:

```tsx
<ChartTrendline
  stroke="#FF5733"
/>
```

The official API describes the trendline color through `stroke`. 

## Trendline markers and data labels

Use `ChartMarker` inside `ChartTrendline` when trendline points need markers. Place `ChartDataLabel` inside that marker when calculated trend values need labels.

```tsx
<ChartTrendline
  type="Linear"
  name="Linear trend"
>
  <ChartMarker
    visible={true}
    shape="Circle"
    width={7}
    height={7}
  >
    <ChartDataLabel
      visible={true}
      position="Top"
      format="{value}"
    />
  </ChartMarker>
</ChartTrendline>
```

Do not pass an object-style `marker={{ ... }}` prop when the Pure React component hierarchy provides the `ChartMarker` child. The official feature documentation describes `ChartMarker` with `ChartDataLabel` for trendline labels. 

## Forecast extensions

Use `forwardForecast` and `backwardForecast` to extend a fitted trendline by a number of data periods.

```tsx
<ChartTrendline
  type="Linear"
  name="Trend and forecast"
  forwardForecast={12}
  backwardForecast={3}
/>
```

Both defaults are `0`. Forecast values extend the fitted model, not the source data itself. Use non-negative values appropriate to the axis spacing and analytical context. turn49search206

Do not present a forecast extension as a guaranteed future outcome. It is a projection of the selected mathematical model.

## Fixed intercept

Use `intercept` to force the fitted trendline through a specific Y-axis intercept.

```tsx
<ChartTrendline
  type="Linear"
  intercept={0}
/>
```

The documented default is `null`, which allows the model to calculate the intercept. Configure a fixed intercept only when the analytical requirement justifies it. 

## Tooltip

Trendline tooltips are enabled by default through `enableTooltip={true}`.

```tsx
<ChartTrendline
  type="Linear"
  enableTooltip={true}
/>
```

Set `enableTooltip={false}` when trendline hover information is not required. 

## Legend name and shape

Use `name` for the trendline's legend text and `legendShape` for its legend symbol.

```tsx
<ChartTrendline
  type="Linear"
  name="Linear trend"
  legendShape="HorizontalLine"
/>
```

The documented `legendShape` default is `SeriesType`. With that default, the trendline uses a straight-line symbol in the legend styled with the trendline stroke. Use only values from the public `LegendShape` union. 

## Animation

Configure trendline animation through the `animation` object.

```tsx
<ChartTrendline
  type="Linear"
  animation={{
    enable: true,
    duration: 500,
    delay: 0,
  }}
/>
```

The documented default is `{ enable: true, duration: 1000, delay: 0 }`. Disable animation only when the request or surrounding chart behavior requires it. 

## Complete example

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartDataLabel,
  ChartLegend,
  ChartMarker,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
  ChartTrendline,
  ChartTrendlineCollection,
} from "@syncfusion/react-charts";

const data = [
  { month: 1, sales: 12 },
  { month: 2, sales: 18 },
  { month: 3, sales: 17 },
  { month: 4, sales: 26 },
  { month: 5, sales: 31 },
  { month: 6, sales: 37 },
  { month: 7, sales: 41 },
  { month: 8, sales: 48 },
];

export default function TrendlineChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis
        valueType="Double"
        minimum={1}
        maximum={10}
        interval={1}
      >
        <ChartAxisTitle text="Month" />
        <ChartAxisLabel format="{value}" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double">
        <ChartAxisTitle text="Sales" />
        <ChartAxisLabel format="{value}" />
      </ChartPrimaryYAxis>

      <ChartLegend
        visible={true}
        position="Bottom"
      />

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="sales"
          type="Scatter"
          name="Observed sales"
        >
          <ChartMarker
            visible={true}
            shape="Circle"
            width={8}
            height={8}
            fill="#1565C0"
          />

          <ChartTrendlineCollection>
            <ChartTrendline
              type="Linear"
              name="Linear trend"
              stroke="#D32F2F"
              width={2}
              dashArray="5,5"
              opacity={1}
              forwardForecast={2}
              backwardForecast={0}
              enableTooltip={true}
              legendShape="HorizontalLine"
              animation={{
                enable: true,
                duration: 500,
                delay: 0,
              }}
            >
              <ChartMarker
                visible={true}
                shape="Circle"
                width={6}
                height={6}
              >
                <ChartDataLabel
                  visible={false}
                  position="Top"
                  format="{value}"
                />
              </ChartMarker>
            </ChartTrendline>
          </ChartTrendlineCollection>
        </ChartSeries>
      </ChartSeriesCollection>
    </Chart>
  );
}
```

Trendlines are supported for Cartesian series families including line, scatter, area, column, and supported financial charts. Verify compatibility with the selected series type rather than attaching a trendline to every chart family. turn49search207

## Common errors

### Missing collection wrapper

Incorrect:

```tsx
<ChartSeries>
  <ChartTrendline type="Linear" />
</ChartSeries>
```

Correct:

```tsx
<ChartSeries>
  <ChartTrendlineCollection>
    <ChartTrendline type="Linear" />
  </ChartTrendlineCollection>
</ChartSeries>
```

### Using `fill` for line color

Incorrect:

```tsx
<ChartTrendline fill="#FF5733" />
```

Correct:

```tsx
<ChartTrendline stroke="#FF5733" />
```

### Object-style marker configuration

Incorrect for the child-based Pure React pattern:

```tsx
<ChartTrendline
  marker={{ visible: true, shape: "Circle" }}
/>
```

Correct:

```tsx
<ChartTrendline>
  <ChartMarker visible={true} shape="Circle" />
</ChartTrendline>
```

### Applying properties to the wrong type

- `polynomialOrder` belongs to `Polynomial`.
- `period` belongs to `MovingAverage`.
- Forecast properties extend fitted models and should not be treated as observed data.

### Invalid model domains

Do not use logarithmic or power models with values that violate their mathematical domain requirements.

### Hidden parent series

A trendline is computed only when the parent series is visible. 

## Validation checklist

Before returning a trendline implementation:

1. Import `ChartTrendlineCollection` and `ChartTrendline` from `@syncfusion/react-charts`.
2. Place `ChartTrendlineCollection` inside the owning `ChartSeries`.
3. Place each `ChartTrendline` inside that collection.
4. Use only `Linear`, `Exponential`, `Polynomial`, `Power`, `Logarithmic`, or `MovingAverage`.
5. Use `stroke`, not `fill`, for line color.
6. Keep opacity between `0` and `1`.
7. Use a positive integer `polynomialOrder` only with `Polynomial`.
8. Use a positive integer `period` only with `MovingAverage`.
9. Ensure the source data has enough points for the selected model.
10. Ensure logarithmic and power models receive mathematically valid data.
11. Use non-negative, meaningful forecast periods.
12. Do not describe forecast extensions as guaranteed outcomes.
13. Use `ChartMarker` as a child for trendline markers.
14. Place `ChartDataLabel` inside the trendline marker when calculated-point labels are required.
15. Keep the parent series visible when the trendline must be computed.
16. Ensure every imported symbol is used.
17. Emit valid, unescaped TSX.
