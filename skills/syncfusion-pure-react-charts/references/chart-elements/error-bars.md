# Error Bars Reference

Use error bars to show uncertainty or variability around Cartesian chart points. Configure `ChartErrorBar` as a child of the owning `ChartSeries`.

## Required hierarchy

```tsx
import {
  Chart,
  ChartErrorBar,
  ChartPrimaryXAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { category: "A", value: 100 },
  { category: "B", value: 120 },
];

<Chart>
  <ChartPrimaryXAxis valueType="Category" />

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="category"
      yField="value"
      type="Line"
    >
      <ChartErrorBar
        visible={true}
        type="Custom"
        verticalError={10}
      />
    </ChartSeries>
  </ChartSeriesCollection>
</Chart>
```

Do not place `ChartErrorBar` directly under `Chart` or `ChartSeriesCollection`.

## Verified properties

The Pure React `ChartErrorBarProps` API documents:

- `visible: boolean`, default `false`
- `type: "Percentage" | "StandardDeviation" | "StandardError" | "Custom"`, default `"Custom"`
- `verticalError: number | string`, default `1`
- `horizontalError: number | string`, default `0`
- `width: number`, default `1`
- `color: string`, default `""`
- `errorBarColorField: string`, default `""`
- `errorBarCap: ChartErrorBarCapProps`, default `{ width: 1, length: 10, color: "", opacity: 1 }`

Use only properties owned by `ChartErrorBar`.

## Important corrections

The original draft used APIs that are not part of the verified Pure React error-bar contract:

- Do not use `errorBarValue`; use `verticalError` or `horizontalError`.
- Do not use `mode="Vertical"`, `mode="Horizontal"`, or `mode="Both"`; direction follows the configured vertical and horizontal error values.
- Do not use `type="Fixed"`; the verified Pure React type union contains `Percentage`, `StandardDeviation`, `StandardError`, and `Custom`.
- Do not use `fill`; use `color` for the error-bar stroke.
- Do not use `dashArray` directly on `ChartErrorBar` unless the current public API adds it.
- Do not use `horizontalLineWidth`; customize end caps through `errorBarCap.length` and `errorBarCap.width`.
- Do not map error values through series `high` and `low`; those fields belong to range and financial series.

## Custom error bars

Use `type="Custom"` to provide explicit vertical and horizontal errors.

### Vertical error

```tsx
<ChartErrorBar
  visible={true}
  type="Custom"
  verticalError={10}
  horizontalError={0}
/>
```

### Horizontal error

```tsx
<ChartErrorBar
  visible={true}
  type="Custom"
  verticalError={0}
  horizontalError={5}
/>
```

### Both directions

```tsx
<ChartErrorBar
  visible={true}
  type="Custom"
  verticalError={10}
  horizontalError={5}
/>
```

Use numeric values when every point shares the same error magnitude.

## Map errors from data fields

Both `verticalError` and `horizontalError` accept a number or string. Use a string to map the error magnitude from a data-source field.

```tsx
const data = [
  { x: 10, y: 100, xError: 1.5, yError: 8 },
  { x: 20, y: 120, xError: 2, yError: 12 },
  { x: 30, y: 108, xError: 1, yError: 6 },
];

<ChartSeries
  dataSource={data}
  xField="x"
  yField="y"
  type="Scatter"
>
  <ChartErrorBar
    visible={true}
    type="Custom"
    horizontalError="xError"
    verticalError="yError"
  />
</ChartSeries>
```

Every mapped error field must exist in the applicable data objects and contain finite, non-negative numeric values.

Do not use this incorrect range-series pattern:

```tsx
<ChartSeries
  high="highError"
  low="lowError"
>
  <ChartErrorBar type="Custom" />
</ChartSeries>
```

## Percentage error bars

Use `type="Percentage"` and provide the error magnitude through the appropriate error property.

```tsx
<ChartErrorBar
  visible={true}
  type="Percentage"
  verticalError={10}
/>
```

The error is calculated as a percentage of each point value, so its rendered magnitude varies with the point value.

For horizontal percentage error:

```tsx
<ChartErrorBar
  visible={true}
  type="Percentage"
  horizontalError={5}
  verticalError={0}
/>
```

## Standard-deviation error bars

Use `type="StandardDeviation"` for standard-deviation-based variability.

```tsx
<ChartErrorBar
  visible={true}
  type="StandardDeviation"
  verticalError={1}
/>
```

Use the vertical or horizontal error value as required by the selected statistical configuration. Keep the data numeric and ensure the series has enough valid values for a meaningful statistical result.

## Standard-error bars

Use `type="StandardError"` for standard error of the mean.

```tsx
<ChartErrorBar
  visible={true}
  type="StandardError"
  verticalError={1}
/>
```

Do not describe standard error as identical to standard deviation. They communicate different statistical quantities.

## Appearance

Use `color` and `width` for the error-bar stroke.

```tsx
<ChartErrorBar
  visible={true}
  type="Custom"
  verticalError={10}
  color="#D32F2F"
  width={2}
/>
```

- `color` accepts a valid CSS color string.
- `width` controls the error-bar stroke width.

## End-cap appearance

Use `errorBarCap` to configure the caps at the ends of error bars.

```tsx
<ChartErrorBar
  visible={true}
  type="Custom"
  verticalError={10}
  errorBarCap={{
    width: 1,
    length: 10,
    color: "#D32F2F",
    opacity: 1,
  }}
/>
```

The documented default cap object is:

```tsx
{
  width: 1,
  length: 10,
  color: "",
  opacity: 1,
}
```

Use `opacity` from `0` through `1`. Use `length` to control the end-cap length, replacing the invalid `horizontalLineWidth` draft property.

## Data-driven error-bar colors

Use `errorBarColorField` to map the error-bar color from a data field.

```tsx
const data = [
  { category: "A", value: 100, error: 8, errorColor: "#2E7D32" },
  { category: "B", value: 120, error: 14, errorColor: "#C62828" },
];

<ChartSeries
  dataSource={data}
  xField="category"
  yField="value"
  type="Column"
>
  <ChartErrorBar
    visible={true}
    type="Custom"
    verticalError="error"
    errorBarColorField="errorColor"
  />
</ChartSeries>
```

The mapped color field must exist and contain valid CSS color values.

## Complete example

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartErrorBar,
  ChartMarker,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { item: "Printer", quantity: 850, error: 60 },
  { item: "Desktop", quantity: 920, error: 75 },
  { item: "Charger", quantity: 640, error: 45 },
  { item: "Mobile", quantity: 1100, error: 90 },
  { item: "Keyboard", quantity: 720, error: 55 },
];

export default function ErrorBarsChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis valueType="Category">
        <ChartAxisTitle text="Item" />
        <ChartAxisLabel edgeLabelPlacement="Shift" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double" minimum={0}>
        <ChartAxisTitle text="Quantity" />
        <ChartAxisLabel format="{value}" />
      </ChartPrimaryYAxis>

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="item"
          yField="quantity"
          type="Line"
          name="Quantity"
          width={2}
        >
          <ChartMarker visible={true} shape="Circle" width={8} height={8} />
          <ChartErrorBar
            visible={true}
            type="Custom"
            verticalError="error"
            horizontalError={0}
            color="#D32F2F"
            width={2}
            errorBarCap={{
              width: 1,
              length: 10,
              color: "#D32F2F",
              opacity: 1,
            }}
          />
        </ChartSeries>
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Common errors

### Using old or unsupported value properties

Incorrect:

```tsx
<ChartErrorBar errorBarValue={10} />
```

Correct:

```tsx
<ChartErrorBar verticalError={10} />
```

### Using an unsupported mode

Incorrect:

```tsx
<ChartErrorBar mode="Vertical" />
```

Correct:

```tsx
<ChartErrorBar
  verticalError={10}
  horizontalError={0}
/>
```

### Using an unsupported type

Incorrect for the verified Pure React API:

```tsx
<ChartErrorBar type="Fixed" />
```

Correct for explicit fixed magnitudes:

```tsx
<ChartErrorBar
  type="Custom"
  verticalError={10}
/>
```

### Using `fill`

Incorrect:

```tsx
<ChartErrorBar fill="#FF5733" />
```

Correct:

```tsx
<ChartErrorBar color="#FF5733" />
```

### Using invalid cap properties

Incorrect:

```tsx
<ChartErrorBar horizontalLineWidth={10} />
```

Correct:

```tsx
<ChartErrorBar
  errorBarCap={{
    width: 1,
    length: 10,
    color: "#FF5733",
    opacity: 1,
  }}
/>
```

### Using range-series field mappings

Do not map error magnitudes with `ChartSeries.high` or `ChartSeries.low`. Use `ChartErrorBar.verticalError` and `horizontalError` as numbers or field-name strings.

## Validation checklist

Before returning an error-bar implementation:

1. Import `ChartErrorBar` from `@syncfusion/react-charts`.
2. Place it inside the owning `ChartSeries`.
3. Set `visible={true}`.
4. Use only `Percentage`, `StandardDeviation`, `StandardError`, or `Custom` for `type`.
5. Use `verticalError` and `horizontalError`, not `errorBarValue`.
6. Do not use a `mode` property.
7. Use `Custom` for explicit fixed or mapped error magnitudes.
8. Ensure mapped error-field names exist in the series data.
9. Ensure error values are finite and non-negative.
10. Use `color`, not `fill`, for the error-bar stroke.
11. Use `errorBarCap` for cap width, length, color, and opacity.
12. Use `errorBarColorField` only with a valid mapped color field.
13. Do not use series `high` and `low` as error-bar mappings.
14. Ensure all imports are used.
15. Emit valid, unescaped TSX.
