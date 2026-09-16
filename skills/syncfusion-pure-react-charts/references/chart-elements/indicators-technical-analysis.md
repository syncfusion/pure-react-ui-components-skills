# Indicators (Technical Analysis) Reference

Use technical indicators to derive market-analysis lines or bands from a financial source series. Configure indicators in the chart-level `ChartIndicatorCollection`, not as children of `ChartSeries`. Each indicator binds to its source series through `seriesName`, which must match the source series `name`.

## Required component hierarchy

```tsx
import {
  Chart,
  ChartIndicator,
  ChartIndicatorCollection,
  ChartPrimaryXAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

<Chart>
  <ChartPrimaryXAxis valueType="DateTime" />

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="date"
      open="open"
      high="high"
      low="low"
      close="close"
      type="Candle"
      name="Price"
    />
  </ChartSeriesCollection>

  <ChartIndicatorCollection>
    <ChartIndicator
      type="BollingerBands"
      field="Close"
      period={20}
      standardDeviation={2}
      seriesName="Price"
    />
  </ChartIndicatorCollection>
</Chart>
```

Do not nest `ChartIndicator` inside `ChartSeries`.

Incorrect:

```tsx
<ChartSeries type="Candle">
  <ChartIndicator type="BollingerBands" />
</ChartSeries>
```

Correct:

```tsx
<ChartSeriesCollection>
  <ChartSeries type="Candle" dataSource={data} name="Price" />
</ChartSeriesCollection>

<ChartIndicatorCollection>
  <ChartIndicator type="BollingerBands" seriesName="Price" />
</ChartIndicatorCollection>
```

## Source-series binding

An indicator reads its input data from the series whose `name` matches the indicator's `seriesName`. Indicators do not have their own `dataSource`, `xField`, `open`, `high`, `low`, `close`, or `volume` properties — financial field mappings belong to the source series.

```tsx
const data = [
  {
    date: new Date(2026, 0, 2),
    open: 101,
    high: 108,
    low: 98,
    close: 106,
    volume: 145000,
  },
];

<ChartSeriesCollection>
  <ChartSeries
    dataSource={data}
    xField="date"
    open="open"
    high="high"
    low="low"
    close="close"
    volume="volume"
    type="Candle"
    name="Price"
  />
</ChartSeriesCollection>

<ChartIndicatorCollection>
  <ChartIndicator type="Sma" field="Close" period={14} seriesName="Price" />
</ChartIndicatorCollection>
```

Every field mapped on the source series must exist in its data objects. Use numeric values for price and volume fields and date-compatible values for a DateTime X-axis.

## Indicator type values

Use exact type names from the `IndicatorsType` union:

- `Sma`
- `Ema`
- `Tma`
- `Momentum`
- `Atr`
- `AccumulationDistribution`
- `BollingerBands`
- `Macd`
- `Rsi`
- `Stochastic`

Use exact casing. Do not emit `RSI`, `MACD`, or `EMA` as component `type` values.

Display names and type literals differ:

- RSI display name → `type="Rsi"`
- MACD display name → `type="Macd"`
- EMA display name → `type="Ema"`
- SMA display name → `type="Sma"`
- TMA display name → `type="Tma"`
- ATR display name → `type="Atr"`

## Verified common properties

The official `ChartIndicatorProps` API documents common indicator settings including:

- `accessibility`
- `animation`
- `dashArray`
- `field`
- `fill`
- `period`
- `seriesName`
- `type`
- `width`
- `visible`
- `xAxisName`
- `yAxisName`

The documented defaults include:

- `animation`: `{ enable: true, duration: 1000, delay: 0 }`
- `dashArray`: `""`
- `field`: `"Close"`
- `fill`: `""`
- `period`: `14`

Do not apply every property to every indicator. Use only the settings meaningful to the selected indicator.

## Financial data field

`field` selects the financial value used for price-based calculations. The documented default is `Close`.

```tsx
<ChartIndicator
  type="Rsi"
  field="Close"
  seriesName="Price"
/>
```

Use exact values from the `FinancialDataField` union: `Open`, `High`, `Low`, `Close`.

Do not use a mapped JavaScript property name such as `"close"` for `field`. `close="close"` on the series maps the data property; `field="Close"` on the indicator selects the financial calculation field.

## Simple moving average

```tsx
<ChartIndicatorCollection>
  <ChartIndicator
    type="Sma"
    field="Close"
    period={10}
    fill="#1565C0"
    width={2}
    seriesName="Price"
  />
</ChartIndicatorCollection>
```

`period` is the rolling look-back period. Ensure the source data contains enough points for a meaningful result.

## Exponential moving average

```tsx
<ChartIndicatorCollection>
  <ChartIndicator
    type="Ema"
    field="Close"
    period={10}
    fill="#2E7D32"
    width={2}
    seriesName="Price"
  />
</ChartIndicatorCollection>
```

## Triangular moving average

```tsx
<ChartIndicatorCollection>
  <ChartIndicator
    type="Tma"
    field="Close"
    period={10}
    fill="#6A1B9A"
    width={2}
    seriesName="Price"
  />
</ChartIndicatorCollection>
```

Do not use uppercase `SMA`, `EMA`, or `TMA` literals.

## Bollinger Bands

Bollinger Bands require a period, a standard-deviation multiplier, and financial price data on the source series.

```tsx
<ChartIndicatorCollection>
  <ChartIndicator
    type="BollingerBands"
    field="Close"
    period={20}
    standardDeviation={2}
    bandColor="rgba(33, 150, 243, 0.18)"
    fill="#1565C0"
    width={2}
    seriesName="Price"
  />
</ChartIndicatorCollection>
```

The official API documents:

- `bandColor`: fill color for the Bollinger range band; default `rgba(211,211,211,0.25)`
- `standardDeviation`: multiplier used to calculate the upper and lower bands; default `2`
- `period`: look-back period; default `14`
- `upperLine`: upper-line connector settings
- `lowerLine`: lower-line connector settings
- `periodLine`: central or period-line connector settings

Do not use `fill` as the range-band color. Use `bandColor` for the band and line-specific props for line appearance.

```tsx
<ChartIndicator
  type="BollingerBands"
  seriesName="Price"
  bandColor="rgba(33, 150, 243, 0.18)"
  upperLine={{ color: "#1565C0", width: 1, dashArray: "" }}
  lowerLine={{ color: "#1565C0", width: 1, dashArray: "" }}
  periodLine={{ color: "#0D47A1", width: 2, dashArray: "" }}
/>
```

## Relative Strength Index

Use `type="Rsi"`, not `RSI`.

```tsx
<ChartIndicatorCollection>
  <ChartIndicator
    type="Rsi"
    field="Close"
    period={14}
    overBought={70}
    overSold={30}
    fill="#7B1FA2"
    yAxisName="rsiAxis"
    seriesName="Price"
  />
</ChartIndicatorCollection>
```

The official API documents defaults of `overBought={80}` and `overSold={20}`. Configure different thresholds only when the requested analysis requires them.

RSI commonly uses a separate Y-axis because its scale differs from price. Define the axis through `ChartAxes` and ensure `yAxisName` matches its `name` exactly.

## Moving Average Convergence Divergence

Use `type="Macd"`, not `MACD`.

```tsx
<ChartIndicatorCollection>
  <ChartIndicator
    type="Macd"
    field="Close"
    fastPeriod={26}
    slowPeriod={12}
    macdType="Both"
    macdPositiveColor="#2E7D32"
    macdNegativeColor="#C62828"
    yAxisName="macdAxis"
    seriesName="Price"
  />
</ChartIndicatorCollection>
```

The official API documents:

- `fastPeriod`, with documented default `26`
- `slowPeriod`, with documented default `12`
- `macdType`, with documented default `Both`
- `macdLine`, with default color `#ff9933` and width `2`
- `macdPositiveColor`, default `#2ecd71`
- `macdNegativeColor`, default `#e74c3d`

Use only values from the public `MacdType` union (`Line`, `Histogram`, `Both`) for `macdType`. There is no `signalPeriod` property; the signal line is derived from the configured periods.

## Stochastic oscillator

Use `type="Stochastic"`.

```tsx
<ChartIndicatorCollection>
  <ChartIndicator
    type="Stochastic"
    field="Close"
    period={14}
    kPeriod={14}
    dPeriod={3}
    overBought={80}
    overSold={20}
    yAxisName="stochasticAxis"
    seriesName="Price"
  />
</ChartIndicatorCollection>
```

The official API documents:

- `kPeriod`, default `14`
- `dPeriod`, default `3`
- `overBought`, default `80`
- `overSold`, default `20`

The high, low, and close inputs come from the source series financial field mappings.

## Average True Range

Use `type="Atr"`.

```tsx
<ChartIndicatorCollection>
  <ChartIndicator
    type="Atr"
    field="Close"
    period={14}
    yAxisName="atrAxis"
    seriesName="Price"
  />
</ChartIndicatorCollection>
```

ATR typically uses a separate numeric scale from price.

## Momentum

```tsx
<ChartIndicatorCollection>
  <ChartIndicator
    type="Momentum"
    field="Close"
    period={14}
    yAxisName="momentumAxis"
    seriesName="Price"
  />
</ChartIndicatorCollection>
```

## Accumulation Distribution

Requires high, low, close, and volume mappings on the source series.

```tsx
<ChartSeries
  dataSource={data}
  xField="date"
  high="high"
  low="low"
  close="close"
  volume="volume"
  type="Candle"
  name="Price"
/>

<ChartIndicatorCollection>
  <ChartIndicator
    type="AccumulationDistribution"
    yAxisName="adAxis"
    seriesName="Price"
  />
</ChartIndicatorCollection>
```

Every source data object must contain finite numeric high, low, close, and volume values.

## Separate indicator axis

Define additional axes inside `ChartAxes`, then map indicators by `yAxisName`.

```tsx
<ChartAxes>
  <ChartAxis
    name="rsiAxis"
    minimum={0}
    maximum={100}
    interval={20}
    opposedPosition={true}
  >
    <ChartAxisTitle text="RSI" />
    <ChartAxisLabel format="{value}" />
  </ChartAxis>
</ChartAxes>

<ChartIndicatorCollection>
  <ChartIndicator
    type="Rsi"
    seriesName="Price"
    yAxisName="rsiAxis"
  />
</ChartIndicatorCollection>
```

Do not invent `ChartSecondaryYAxis`. The axis `name` and indicator `yAxisName` must match exactly.

## Multiple indicators

Add multiple `ChartIndicator` children to one `ChartIndicatorCollection`.

```tsx
<ChartIndicatorCollection>
  <ChartIndicator
    type="Ema"
    field="Close"
    period={9}
    fill="#E53935"
    seriesName="Price"
  />
  <ChartIndicator
    type="BollingerBands"
    field="Close"
    period={20}
    standardDeviation={2}
    bandColor="rgba(30, 136, 229, 0.16)"
    seriesName="Price"
  />
</ChartIndicatorCollection>
```

Do not place multiple indicators directly inside the source series.

## Complete example

```tsx
import {
  Chart,
  ChartAxes,
  ChartAxis,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartIndicator,
  ChartIndicatorCollection,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { date: new Date(2026, 0, 2), open: 101, high: 108, low: 98, close: 106, volume: 145000 },
  { date: new Date(2026, 0, 3), open: 106, high: 111, low: 103, close: 109, volume: 163000 },
  { date: new Date(2026, 0, 4), open: 109, high: 112, low: 104, close: 105, volume: 151000 },
  { date: new Date(2026, 0, 5), open: 105, high: 114, low: 102, close: 112, volume: 179000 },
  { date: new Date(2026, 0, 6), open: 112, high: 118, low: 109, close: 116, volume: 188000 },
];

export default function TechnicalIndicatorsChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis valueType="DateTime">
        <ChartAxisTitle text="Date" />
        <ChartAxisLabel format="yMd" edgeLabelPlacement="Shift" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double">
        <ChartAxisTitle text="Price" />
        <ChartAxisLabel format="C0" />
      </ChartPrimaryYAxis>

      <ChartAxes>
        <ChartAxis
          name="rsiAxis"
          valueType="Double"
          minimum={0}
          maximum={100}
          interval={20}
          opposedPosition={true}
        >
          <ChartAxisTitle text="RSI" />
          <ChartAxisLabel format="{value}" />
        </ChartAxis>
      </ChartAxes>

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="date"
          open="open"
          high="high"
          low="low"
          close="close"
          type="Candle"
          name="Price"
        />
      </ChartSeriesCollection>

      <ChartIndicatorCollection>
        <ChartIndicator
          type="Ema"
          field="Close"
          period={3}
          fill="#1565C0"
          width={2}
          seriesName="Price"
        />
        <ChartIndicator
          type="Rsi"
          field="Close"
          period={3}
          overBought={70}
          overSold={30}
          fill="#7B1FA2"
          yAxisName="rsiAxis"
          seriesName="Price"
        />
      </ChartIndicatorCollection>
    </Chart>
  );
}
```

Small sample periods above only demonstrate configuration. For analytical use, provide sufficient historical data for the selected period and indicator calculation.

## Common errors

### Nesting indicators inside a series

Incorrect:

```tsx
<ChartSeries type="Candle">
  <ChartIndicator type="Ema" />
</ChartSeries>
```

Correct:

```tsx
<ChartSeriesCollection>
  <ChartSeries type="Candle" name="Price" />
</ChartSeriesCollection>

<ChartIndicatorCollection>
  <ChartIndicator type="Ema" seriesName="Price" />
</ChartIndicatorCollection>
```

### Using the wrong collection component

`ChartIndicators` is not exported by the package. Use `ChartIndicatorCollection`.

Incorrect:

```tsx
<ChartIndicators>
  <ChartIndicator />
</ChartIndicators>
```

Correct:

```tsx
<ChartIndicatorCollection>
  <ChartIndicator />
</ChartIndicatorCollection>
```

### Mapping data directly on the indicator

`ChartIndicator` has no `dataSource`, `xField`, `open`, `high`, `low`, `close`, or `volume` properties. Map the data on the source series and bind with `seriesName`.

Incorrect:

```tsx
<ChartIndicator dataSource={data} xField="date" close="close" />
```

Correct:

```tsx
<ChartSeries dataSource={data} xField="date" close="close" name="Price" />

<ChartIndicator type="Sma" seriesName="Price" />
```

### Wrong type casing

Incorrect:

```tsx
<ChartIndicator type="RSI" />
<ChartIndicator type="MACD" />
```

Correct:

```tsx
<ChartIndicator type="Rsi" />
<ChartIndicator type="Macd" />
```

### Confusing mapped field and calculation field

Incorrect:

```tsx
<ChartIndicator field="close" />
```

Correct:

```tsx
<ChartSeries close="close" name="Price" />

<ChartIndicator field="Close" seriesName="Price" />
```

### Missing named axis

Incorrect:

```tsx
<ChartIndicator type="Rsi" yAxisName="rsiAxis" />
```

when no `ChartAxis name="rsiAxis"` exists.

### Unmatched seriesName

The indicator's `seriesName` must exactly match the source series `name`, or the indicator will not render.

Incorrect:

```tsx
<ChartSeries name="Price" />
<ChartIndicator seriesName="price" />
```

Correct:

```tsx
<ChartSeries name="Price" />
<ChartIndicator seriesName="Price" />
```

## Validation checklist

Before returning a technical-indicator implementation:

1. Import `ChartIndicatorCollection` and `ChartIndicator` from `@syncfusion/react-charts`.
2. Place indicators inside the chart-level `ChartIndicatorCollection`.
3. Do not nest indicators inside `ChartSeries`.
4. Map the data and financial fields on the source series, not the indicator.
5. Bind the indicator to its source series with a `seriesName` that exactly matches the series `name`.
6. Ensure every source-series mapped field exists and contains compatible values.
7. Use exact indicator type casing, including `Rsi`, `Macd`, `Sma`, `Ema`, `Tma`, and `Atr`.
8. Use exact financial field values such as `Close` for `field`.
9. Use `bandColor` for the Bollinger range band.
10. Use `period`, `standardDeviation`, and line settings only where applicable.
11. Use `kPeriod` and `dPeriod` for Stochastic configuration.
12. Use MACD-specific periods, colors, lines, and `macdType` only for `Macd`.
13. Define additional axes through `ChartAxes` when the indicator needs a separate scale.
14. Match `xAxisName` and `yAxisName` exactly with axis names.
15. Ensure enough source points exist for the configured calculation period.
16. Ensure every imported symbol is used.
17. Emit valid, unescaped TSX.
