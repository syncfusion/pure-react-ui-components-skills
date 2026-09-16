# Axis Labels Reference

Use this reference for axis-label collision handling, rotation, placement, edge-label behavior, formatting, styling, trimming, wrapping, density, and multi-level labels in Syncfusion Pure React Charts.

## Component hierarchy

Place `ChartAxisLabel` inside the axis it configures.

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

<Chart>
  <ChartPrimaryXAxis valueType="Category">
    <ChartAxisLabel />
  </ChartPrimaryXAxis>

  <ChartPrimaryYAxis valueType="Double">
    <ChartAxisLabel format="{value}" />
  </ChartPrimaryYAxis>

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="month"
      yField="sales"
      type="Column"
    />
  </ChartSeriesCollection>
</Chart>
```

Do not place `ChartAxisLabel` directly under `Chart`. Keep the label component inside `ChartPrimaryXAxis`, `ChartPrimaryYAxis`, or a named `ChartAxis` inside `ChartAxes`.

## Core label properties

The official `ChartAxisLabelProps` API documents these implementation properties:

- `align`: `"Left" | "Center" | "Right"`; default `"Center"`
- `border`: `{ color, width, dashArray }`; default width is `0`
- `color`: CSS color string; default `""`
- `edgeLabelPlacement`: `"None" | "Hide" | "Shift"`; default `"Shift"`
- `enableTrim`: boolean; default `false`
- `enableWrap`: boolean; default `false`
- `fontFamily`: string; default `""`
- `fontSize`: string; default `""`
- `fontStyle`: string; default `""`
- `fontWeight`: string; default `""`
- `format`: string; default `""`
- `formatter`: `(value: number, text: string) => string | boolean`; default `null`
- `intersectMode`: controls overlapping labels
- `maxLabelWidth`: maximum label width used by trimming and wrapping; source default `34`
- `position`: `"Inside" | "Outside"`; default is outside placement
- `rotationAngle`: number; default `0`

Use only properties owned by `ChartAxisLabel`. Axis range, intervals, value type, category indexing, and label density belong to the parent axis.

## Smart overlap handling

Use `intersectMode` when labels collide.

Supported source values:

- `None`: render labels without collision handling
- `Hide`: hide overlapping labels
- `Trim`: truncate overlapping labels
- `Wrap`: wrap labels to multiple lines
- `MultipleRows`: place labels on multiple rows
- `Rotate45`: rotate labels by 45 degrees
- `Rotate90`: rotate labels by 90 degrees

```tsx
<ChartPrimaryXAxis valueType="Category">
  <ChartAxisLabel intersectMode="Rotate45" />
</ChartPrimaryXAxis>
```

Choose one collision strategy. Do not combine an automatic rotation mode with a conflicting manual `rotationAngle` unless the requested design requires manual rotation and the runtime behavior has been verified.

### Recommended selection

- Use `Hide` when every label is not required.
- Use `Trim` when labels can be recognized from shortened text.
- Use `Wrap` for long labels when extra vertical space is acceptable.
- Use `MultipleRows` when preserving complete category text is more important than compact height.
- Use `Rotate45` or `Rotate90` when angled text remains readable.
- Use `None` only when overlap is acceptable or independently prevented.

## Manual rotation

Use `rotationAngle` for a specific angle. The source documents a default of `0` and supports values from 0 through 360 degrees.

```tsx
<ChartPrimaryXAxis valueType="Category">
  <ChartAxisLabel rotationAngle={45} />
</ChartPrimaryXAxis>
```

Manual rotation changes label orientation only. It does not change the axis orientation or reverse the axis range.

## Label position

Use `position` to place labels relative to the axis line.

- `Outside`: place labels outside the axis line
- `Inside`: place labels inside the plot area relative to the axis line

```tsx
<ChartPrimaryYAxis>
  <ChartAxisLabel position="Inside" />
</ChartPrimaryYAxis>
```

When labels are inside, verify that labels do not obscure data points, columns, markers, annotations, or other plot-area content.

## Edge-label handling

Use `edgeLabelPlacement` to control labels at the minimum and maximum ends of the axis.

- `None`: apply no special edge handling
- `Hide`: hide an overflowing edge label
- `Shift`: move the edge label inside the available axis bounds

The documented default is `Shift`.

```tsx
<ChartPrimaryXAxis valueType="Category">
  <ChartAxisLabel edgeLabelPlacement="Shift" />
</ChartPrimaryXAxis>
```

Use `Hide` when shifting would create a collision with adjacent labels. Use `Shift` when preserving the first and last labels is important.

## Label formatting

Use `format` on `ChartAxisLabel`.

### Standard numeric formats

The source documents global formats such as:

- `N` or `N2`: number formatting, optionally with decimal precision
- `C` or `C2`: currency formatting, optionally with decimal precision
- `P` or `P2`: percentage formatting, optionally with decimal precision

```tsx
<ChartPrimaryYAxis valueType="Double">
  <ChartAxisLabel format="C0" />
</ChartPrimaryYAxis>
```

### Custom format

Use the `{value}` placeholder to add units or surrounding text.

```tsx
<ChartPrimaryYAxis valueType="Double">
  <ChartAxisLabel format="{value}°C" />
</ChartPrimaryYAxis>
```

Do not use a numeric format that conflicts with the source data. Percentage formats expect percentage-compatible values, and currency output follows the active locale.

## Custom formatter

Use `formatter` when a format string is insufficient. The callback receives the numeric value and current label text and returns a string or `boolean`.

```tsx
const formatAxisLabel = (value: number, text: string): string => {
  return value >= 1000 ? `${value / 1000}K` : text;
};

<ChartPrimaryYAxis valueType="Double">
  <ChartAxisLabel formatter={formatAxisLabel} />
</ChartPrimaryYAxis>
```

Keep the formatter deterministic and fast because it executes for individual axis labels.

## Label styling

Use the label component's font and border properties.

```tsx
<ChartPrimaryXAxis valueType="Category">
  <ChartAxisLabel
    align="Center"
    color="#333333"
    fontFamily="Arial"
    fontSize="12px"
    fontStyle="Normal"
    fontWeight="600"
    border={{
      color: "#D0D0D0",
      width: 1,
      dashArray: "",
    }}
  />
</ChartPrimaryXAxis>
```

Use valid CSS color strings. `fontSize` is a string value such as `"12px"`, not a numeric value.

## Trimming long labels

Set `enableTrim={true}` and provide `maxLabelWidth`.

```tsx
<ChartPrimaryXAxis valueType="Category">
  <ChartAxisLabel
    enableTrim={true}
    maxLabelWidth={80}
  />
</ChartPrimaryXAxis>
```

The source documents a default `maxLabelWidth` of `34` pixels. Increase it when the chart has enough room and preserving more text is useful.

`enableTrim` shortens content that exceeds `maxLabelWidth`. Do not describe trimming as a replacement for tooltip or accessible text unless the dedicated accessibility and tooltip references document that behavior.

## Wrapping long labels

Set `enableWrap={true}` and provide `maxLabelWidth`.

```tsx
<ChartPrimaryXAxis valueType="Category">
  <ChartAxisLabel
    enableWrap={true}
    maxLabelWidth={90}
  />
</ChartPrimaryXAxis>
```

Do not enable both trimming and wrapping without an explicit design requirement and verified behavior. Choose the overflow strategy that matches the requested layout.

## Controlling label density

`maxLabelDensity` belongs to the parent axis, not `ChartAxisLabel`. It limits the maximum number of labels rendered per 100 pixels of axis length. The documented default is `3`.

```tsx
<ChartPrimaryXAxis
  valueType="Category"
  maxLabelDensity={2}
>
  <ChartAxisLabel intersectMode="Hide" />
</ChartPrimaryXAxis>
```

Explicit axis range settings such as `minimum`, `maximum`, and `interval` control the calculated range and can take precedence over automatic label-density behavior.

## Multi-level labels

Use multi-level labels for hierarchical groups that span sections of a single axis. Each level contains one or more categories, and each category defines `start`, `end`, and `text`.

The official API documents the following multi-level label properties:

- `alignment`: `"Left" | "Center" | "Right"`; default `"Center"`
- `border`: `{ color, width, dashArray }`; default width is `0`
- `categories`: multi-level label category collection
- `overflow`: `"Trim" | "Wrap" | "None"`; default `"Wrap"`
- `textStyle`: font settings for the level

Each multi-level category supports:

- `start`: `string | number | Date | null`
- `end`: `string | number | Date | null`
- `text`: string; default `""`
- `maximumTextWidth`: number or `null`

Use the exact multi-level label collection and child component names documented by the package's current Pure React API before generating JSX. Do not invent a collection wrapper from interface names alone.

Conceptual configuration shape:

```tsx
const level = {
  alignment: "Center",
  overflow: "Wrap",
  categories: [
    { start: 0, end: 2, text: "First Half" },
    { start: 3, end: 5, text: "Second Half" },
  ],
};
```

Ensure `start` and `end` values are compatible with the parent axis type.

## Complete label example

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { category: "Consumer Electronics", value: 1250 },
  { category: "Home Appliances", value: 980 },
  { category: "Office Equipment", value: 760 },
  { category: "Sports Accessories", value: 640 },
];

export default function AxisLabelsChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis
        valueType="Category"
        maxLabelDensity={3}
      >
        <ChartAxisLabel
          intersectMode="Rotate45"
          edgeLabelPlacement="Shift"
          enableTrim={true}
          maxLabelWidth={110}
          color="#333333"
          fontFamily="Arial"
          fontSize="12px"
          fontWeight="500"
        />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double">
        <ChartAxisLabel
          format="C0"
          color="#333333"
          fontSize="12px"
        />
      </ChartPrimaryYAxis>

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="category"
          yField="value"
          type="Column"
          name="Sales"
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Validation checklist

Before returning an axis-label implementation:

1. Import `ChartAxisLabel` from `@syncfusion/react-charts`.
2. Place `ChartAxisLabel` inside the axis it configures.
3. Match the parent axis `valueType` to the bound data.
4. Use only documented `intersectMode` values.
5. Use only `Inside` or `Outside` for `position`.
6. Use only `None`, `Hide`, or `Shift` for `edgeLabelPlacement`.
7. Use a string value for `fontSize`.
8. Put `maxLabelDensity` on the parent axis.
9. Provide `maxLabelWidth` when trim or wrap behavior requires a specific width.
10. Keep formatter return values compatible with `string | boolean`.
11. Ensure multi-level category boundaries match the axis data type.
12. Emit valid, unescaped TSX.
