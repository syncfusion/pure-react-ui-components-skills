# Data Labels Reference

Use data labels to display a point's value or a mapped label near the rendered point. Keep data labels inside the series marker hierarchy.

## Required component hierarchy

Place `ChartDataLabel` inside `ChartMarker`, and place `ChartMarker` inside `ChartSeries`.

```tsx
import {
  Chart,
  ChartDataLabel,
  ChartMarker,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

<Chart>
  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="category"
      yField="value"
      type="Column"
    >
      <ChartMarker>
        <ChartDataLabel visible={true} />
      </ChartMarker>
    </ChartSeries>
  </ChartSeriesCollection>
</Chart>
```

Do not place `ChartDataLabel` directly under `ChartSeries`, `ChartSeriesCollection`, or `Chart`.

Incorrect:

```tsx
<ChartSeries dataSource={data} type="Column">
  <ChartDataLabel visible={true} />
</ChartSeries>
```

Correct:

```tsx
<ChartSeries dataSource={data} type="Column">
  <ChartMarker>
    <ChartDataLabel visible={true} />
  </ChartMarker>
</ChartSeries>
```

## Verified data-label properties

The official `ChartDataLabelProps` API documents these properties:

- `border`: label border configuration; default `{ color: "", width: 1 }`
- `borderRadius`: `{ x, y }`; default `{ x: 5, y: 5 }`
- `enableRotation`: boolean; default `false`
- `fill`: CSS color string; default `"transparent"`
- `font`: `ChartFontProps`; default includes `color: ""`, `fontFamily: ""`, `fontSize: ""`, `fontStyle: "Normal"`, `fontWeight: ""`, and `opacity: 1`
- `format`: string or `null`; default `""`
- `formatter`: `(index: number, text: string) => string | boolean`; default `null`
- `intersectMode`: `"None" | "Hide" | "Rotate90"`; default `"Hide"`
- `labelField`: string or `null`; default `""`
- `margin`: `{ left, right, top, bottom }`; default `5` on each side
- `opacity`: number; default `1`
- `position`: data-label position; default `"Auto"`
- `rotationAngle`: rotation angle used when rotation is enabled
- `visible`: boolean controlling label rendering

Use exact property names. In particular, use `fontSize` inside `font`, not `size`.

## Basic labels

```tsx
<ChartSeries
  dataSource={data}
  xField="category"
  yField="value"
  type="Column"
>
  <ChartMarker>
    <ChartDataLabel visible={true} />
  </ChartMarker>
</ChartSeries>
```

A label is generated for each eligible point when `visible={true}`.

## Label position

The official API documents these position values:

- `Auto`
- `Outer`
- `Top`
- `Bottom`
- `Middle`

```tsx
<ChartMarker>
  <ChartDataLabel
    visible={true}
    position="Top"
  />
</ChartMarker>
```

Do not use `Left` or `Right` unless the current public `LabelPosition` union explicitly includes those values. They are not listed in the verified `ChartDataLabelProps` API result.

Position support can vary by series geometry. Use the position supported by the selected series family, and prefer `Auto` when no specific placement is requested.

## Standard and custom formatting

Use `format` to apply a global numeric format or a custom `{value}` pattern.

### Currency

```tsx
<ChartDataLabel
  visible={true}
  format="C0"
/>
```

### Number

```tsx
<ChartDataLabel
  visible={true}
  format="N2"
/>
```

### Percentage

```tsx
<ChartDataLabel
  visible={true}
  format="P1"
/>
```

### Custom unit

```tsx
<ChartDataLabel
  visible={true}
  format="{value}°C"
/>
```

Do not use tooltip-style placeholders such as `${point.x}` or `${point.y}` in `ChartDataLabel.format`. The verified data-label API documents global formats and the `{value}` placeholder.

Incorrect:

```tsx
<ChartDataLabel
  visible={true}
  format="${point.x}: ${point.y}"
/>
```

Correct:

```tsx
<ChartDataLabel
  visible={true}
  format="{value} units"
/>
```

## Map label text from the data source

Use `labelField` when labels should come from a field other than the numeric Y value.

```tsx
const data = [
  { category: "A", value: 42, label: "Target" },
  { category: "B", value: 58, label: "Above target" },
];

<ChartSeries
  dataSource={data}
  xField="category"
  yField="value"
  type="Column"
>
  <ChartMarker>
    <ChartDataLabel
      visible={true}
      labelField="label"
    />
  </ChartMarker>
</ChartSeries>
```

The mapped field must exist in every applicable data object. Use a string-compatible field for label text.

## Formatter callback

Use `formatter` for conditional or computed label text. The callback receives the point index and current label text, and returns a string or boolean.

```tsx
const formatDataLabel = (
  index: number,
  text: string,
): string | boolean => {
  return index % 2 === 0 ? text : false;
};

<ChartMarker>
  <ChartDataLabel
    visible={true}
    formatter={formatDataLabel}
  />
</ChartMarker>
```

Use a returned string to replace the label text. Use the API-supported boolean return to control output according to the component's documented behavior.

Do not assume the formatter receives a point object. The verified signature is `(index: number, text: string) => string | boolean`.

## Conditional labels

Use `formatter` rather than an unverified JSX template callback when conditional output is required.

```tsx
const showSelectedLabels = (
  index: number,
  text: string,
): string | boolean => {
  return index === 0 || index === 3 ? text : false;
};

<ChartMarker>
  <ChartDataLabel
    visible={true}
    formatter={showSelectedLabels}
  />
</ChartMarker>
```

If the condition depends on point data, use the index to read from the same immutable data array:

```tsx
const formatHighValues = (
  index: number,
  text: string,
): string | boolean => {
  return data[index]?.value > 100 ? text : false;
};
```

Keep the formatter synchronized with the data source used by that series.

## Appearance

Use `fill`, `border`, `borderRadius`, `font`, `margin`, and `opacity`.

```tsx
<ChartMarker>
  <ChartDataLabel
    visible={true}
    fill="#FFFFFF"
    border={{
      color: "#666666",
      width: 1,
      dashArray: "",
    }}
    borderRadius={{
      x: 5,
      y: 5,
    }}
    font={{
      fontFamily: "Arial",
      fontSize: "12px",
      fontStyle: "Normal",
      fontWeight: "Bold",
      color: "#222222",
      opacity: 1,
    }}
    margin={{
      left: 5,
      right: 5,
      top: 5,
      bottom: 5,
    }}
    opacity={1}
  />
</ChartMarker>
```

Important corrections:

- Use `fontSize`, not `size`.
- Use a string such as `"12px"` for `fontSize`.
- Use a numeric value from `0` to `1` for opacity.
- Use valid CSS color strings.
- Do not put text appearance directly on `ChartSeries` when it belongs to `ChartDataLabel.font`.

## Rotation

The data-label API separates rotation activation from the rotation angle. Set `enableRotation={true}` and provide `rotationAngle`.

```tsx
<ChartMarker>
  <ChartDataLabel
    visible={true}
    enableRotation={true}
    rotationAngle={45}
  />
</ChartMarker>
```

Do not use an unverified `angle` property.

Incorrect:

```tsx
<ChartDataLabel
  visible={true}
  angle={45}
/>
```

Correct:

```tsx
<ChartDataLabel
  visible={true}
  enableRotation={true}
  rotationAngle={45}
/>
```

When `enableRotation={false}`, the label remains in its default orientation regardless of the rotation angle.

## Overlap handling

Use `intersectMode` to control overlapping data labels.

Supported values:

- `None`: show labels even when they overlap
- `Hide`: hide overlapping labels
- `Rotate90`: rotate labels 90 degrees to reduce overlap

The documented default is `Hide`.

```tsx
<ChartMarker>
  <ChartDataLabel
    visible={true}
    intersectMode="Hide"
  />
</ChartMarker>
```

Do not use axis-label intersection values such as `Trim`, `Wrap`, `MultipleRows`, or `Rotate45` for chart data labels unless the current data-label union explicitly adds them. Axis labels and data labels use different intersection APIs.

## JSX templates

`ChartDataLabelProps` supports a `template` callback that receives `ChartDataLabelTemplateProps` and returns a JSX element. The template context exposes `x`, `y`, `label`, `seriesIndex`, and `pointIndex` — there is no `point` property.

```tsx
<ChartDataLabel
  visible={true}
  template={(props) => <strong>{props.y}</strong>}
/>
```

Use `labelField`, `format`, `formatter`, or `template` for label content customization.

## Stacked-series labels

Point-level data labels for stacked series use the same `ChartMarker` and `ChartDataLabel` hierarchy on each `ChartSeries`.

For stack-total labels, use the chart-level `ChartStackLabels` component with its `visible` prop, placed directly inside `Chart`.

```tsx
<Chart>
  <ChartStackLabels visible={true} />

  <ChartSeriesCollection>
    {/* stacked series */}
  </ChartSeriesCollection>
</Chart>
```

## Complete data-label example

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartDataLabel,
  ChartMarker,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { month: "Jan", sales: 35 },
  { month: "Feb", sales: 42 },
  { month: "Mar", sales: 38 },
  { month: "Apr", sales: 51 },
  { month: "May", sales: 47 },
];

const formatLabel = (
  index: number,
  text: string,
): string | boolean => {
  return index === 3 ? `${text} peak` : text;
};

export default function DataLabelsChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis valueType="Category">
        <ChartAxisTitle text="Month" />
        <ChartAxisLabel edgeLabelPlacement="Shift" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double">
        <ChartAxisTitle text="Sales" />
        <ChartAxisLabel format="{value}" />
      </ChartPrimaryYAxis>

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="sales"
          type="Column"
          name="Sales"
        >
          <ChartMarker>
            <ChartDataLabel
              visible={true}
              position="Top"
              format="{value}"
              formatter={formatLabel}
              fill="#FFFFFF"
              border={{
                color: "#666666",
                width: 1,
                dashArray: "",
              }}
              borderRadius={{ x: 5, y: 5 }}
              font={{
                fontFamily: "Arial",
                fontSize: "12px",
                fontStyle: "Normal",
                fontWeight: "Bold",
                color: "#222222",
                opacity: 1,
              }}
              margin={{
                left: 5,
                right: 5,
                top: 5,
                bottom: 5,
              }}
              intersectMode="Hide"
              opacity={1}
            />
          </ChartMarker>
        </ChartSeries>
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Common errors

### Wrong hierarchy

Incorrect:

```tsx
<ChartSeries>
  <ChartDataLabel visible={true} />
</ChartSeries>
```

Correct:

```tsx
<ChartSeries>
  <ChartMarker>
    <ChartDataLabel visible={true} />
  </ChartMarker>
</ChartSeries>
```

### Unsupported position

Incorrect without union verification:

```tsx
<ChartDataLabel position="Left" />
```

Use `Auto`, `Outer`, `Top`, `Bottom`, or `Middle` according to the verified API and selected series.

### Tooltip placeholders in data-label format

Incorrect:

```tsx
<ChartDataLabel format="${point.x}: ${point.y}" />
```

Correct:

```tsx
<ChartDataLabel format="{value}" />
```

### Wrong font property

Incorrect:

```tsx
font={{ size: "12px" }}
```

Correct:

```tsx
font={{ fontSize: "12px" }}
```

### Wrong rotation properties

Incorrect:

```tsx
<ChartDataLabel angle={45} />
```

Correct:

```tsx
<ChartDataLabel
  enableRotation={true}
  rotationAngle={45}
/>
```

### Assuming template support

Do not use a JSX `template` callback unless it appears in the current public `ChartDataLabelProps` API. Use `labelField`, `format`, or `formatter` instead.

## Validation checklist

Before returning a data-label implementation:

1. Import `ChartMarker` and `ChartDataLabel` from `@syncfusion/react-charts`.
2. Place `ChartDataLabel` inside `ChartMarker`.
3. Place `ChartMarker` inside the owning `ChartSeries`.
4. Set `visible={true}` when labels are requested.
5. Use only verified label positions.
6. Prefer `Auto` when the request does not require a particular position.
7. Use global formats or `{value}` with `format`.
8. Do not use tooltip placeholders in data-label formats.
9. Ensure `labelField` exists in the bound data.
10. Use the formatter signature `(index, text) => string | boolean`.
11. Use `fontSize`, not `size`, inside `font`.
12. Use `enableRotation` with `rotationAngle`.
13. Use only `None`, `Hide`, or `Rotate90` for `intersectMode`.
14. Do not assume JSX-template support.
15. Do not use an unverified standalone stack-label pattern.
16. Ensure every imported symbol is used.
17. Emit valid, unescaped TSX.
