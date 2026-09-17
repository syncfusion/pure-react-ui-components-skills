# Category Axis Reference

Use a Category axis for discrete string labels such as product names, regions, months treated as categories, departments, stages, or other ordered groups.

## Component hierarchy

Set `valueType="Category"` on the axis and place label configuration inside that axis.

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartPrimaryXAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

<Chart>
  <ChartPrimaryXAxis valueType="Category">
    <ChartAxisLabel />
  </ChartPrimaryXAxis>

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="category"
      yField="value"
      type="Column"
    />
  </ChartSeriesCollection>
</Chart>
```

The field mapped by `xField` must exist in the data objects. String and other discrete values are rendered as category labels.

## When to use Category

Use `Category` when:

- X values are discrete labels.
- Equal spacing between labels is intended.
- The source order should control the category order.
- Date-like labels should be treated as discrete categories rather than placed according to elapsed time.

Do not use `Category` when actual time gaps must affect point spacing. Use a `DateTime` axis with date-compatible values for continuous time-based positioning.

## Basic Category axis

```tsx
const data = [
  { category: "North", value: 42 },
  { category: "South", value: 35 },
  { category: "East", value: 51 },
  { category: "West", value: 46 },
];

<ChartPrimaryXAxis valueType="Category" />

<ChartSeriesCollection>
  <ChartSeries
    dataSource={data}
    xField="category"
    yField="value"
    type="Column"
    name="Sales"
  />
</ChartSeriesCollection>
```

Categories follow the order supplied by the series data. Sort or transform the data before binding when a different category order is required.

## Label placement

Use `placement` on `ChartAxisLabel` to control whether category labels appear between or directly on tick marks.

Supported values:

- `BetweenTicks`: place each label between adjacent ticks; this is the documented default behavior for Category axes.
- `OnTicks`: place each label directly on a tick.

```tsx
<ChartPrimaryXAxis valueType="Category">
  <ChartAxisLabel placement="OnTicks" />
</ChartPrimaryXAxis>
```

Use `BetweenTicks` for column-style categories when each label should align with a category band. Use `OnTicks` when labels must align exactly with tick positions or point locations.

Do not confuse `placement` with `position`:

- `placement` controls `BetweenTicks` versus `OnTicks`.
- `position` controls `Inside` versus `Outside` relative to the axis line.

```tsx
<ChartPrimaryXAxis valueType="Category">
  <ChartAxisLabel
    placement="BetweenTicks"
    position="Outside"
  />
</ChartPrimaryXAxis>
```

## Visible category range

For a Category axis, `minimum` and `maximum` refer to category index positions rather than category text.

- `minimum`: starting visible category index
- `maximum`: ending visible category index
- `interval`: spacing between displayed category labels

```tsx
<ChartPrimaryXAxis
  valueType="Category"
  minimum={1}
  maximum={5}
  interval={2}
/>
```

This configuration limits the visible axis range to indexes `1` through `5` and displays labels using an interval of `2`.

### Range validation

- Use zero-based positions that correspond to the category sequence.
- Ensure `minimum` is not greater than `maximum`.
- Ensure the selected range contains categories present in the bound data.
- Use a positive `interval`.
- Do not pass category strings as `minimum` or `maximum` unless the current Pure React API explicitly supports that form.

## Indexed Category axis

The documented default for `indexed` is `false`.

When `indexed={false}`, points from series that share the same category value align to the same category position.

When `indexed={true}`, points are placed by their data-item index instead of being merged only by matching category values.

```tsx
<ChartPrimaryXAxis
  valueType="Category"
  indexed={true}
/>
```

Use an indexed Category axis when duplicate or repeated category labels must remain separate according to their item positions.

### Indexed axis example

```tsx
const firstSeries = [
  { category: "Jan", value: 35 },
  { category: "Feb", value: 42 },
];

const secondSeries = [
  { category: "Jan", value: 28 },
  { category: "Mar", value: 48 },
];

<Chart>
  <ChartPrimaryXAxis
    valueType="Category"
    indexed={true}
  >
    <ChartAxisLabel placement="OnTicks" />
  </ChartPrimaryXAxis>

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={firstSeries}
      xField="category"
      yField="value"
      type="Line"
      name="First"
    />
    <ChartSeries
      dataSource={secondSeries}
      xField="category"
      yField="value"
      type="Line"
      name="Second"
    />
  </ChartSeriesCollection>
</Chart>
```

Do not enable `indexed` automatically. Enable it only when index-based category placement is required.

## Handling dense or long categories

Keep collision handling inside `ChartAxisLabel`.

```tsx
<ChartPrimaryXAxis
  valueType="Category"
  maxLabelDensity={2}
>
  <ChartAxisLabel
    intersectMode="Rotate45"
    edgeLabelPlacement="Shift"
    enableTrim={true}
    maxLabelWidth={90}
  />
</ChartPrimaryXAxis>
```

Relevant label options include:

- `intersectMode`: `None`, `Hide`, `Trim`, `Wrap`, `MultipleRows`, `Rotate45`, or `Rotate90`
- `rotationAngle`: manual rotation angle
- `edgeLabelPlacement`: `None`, `Hide`, or `Shift`
- `enableTrim` and `maxLabelWidth`
- `enableWrap` and `maxLabelWidth`

`maxLabelDensity` belongs to the axis, not `ChartAxisLabel`. Its documented default is `3` labels per 100 pixels of axis length.

## Complete example

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { product: "Laptop Computers", sales: 48 },
  { product: "Mobile Phones", sales: 62 },
  { product: "Home Appliances", sales: 39 },
  { product: "Office Equipment", sales: 31 },
  { product: "Sports Accessories", sales: 44 },
];

export default function CategoryAxisChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis
        valueType="Category"
        indexed={false}
        interval={1}
        maxLabelDensity={3}
      >
        <ChartAxisTitle text="Product" />
        <ChartAxisLabel
          placement="BetweenTicks"
          position="Outside"
          intersectMode="Rotate45"
          edgeLabelPlacement="Shift"
          enableTrim={true}
          maxLabelWidth={100}
        />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double">
        <ChartAxisTitle text="Sales" />
        <ChartAxisLabel format="{value}" />
      </ChartPrimaryYAxis>

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="product"
          yField="sales"
          type="Column"
          name="Sales"
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Common errors

### Using the wrong value type

Incorrect:

```tsx
<ChartPrimaryXAxis valueType="DateTime" />
```

when `xField` maps arbitrary category names.

Correct:

```tsx
<ChartPrimaryXAxis valueType="Category" />
```

### Using label placement on the axis

Incorrect:

```tsx
<ChartPrimaryXAxis
  valueType="Category"
  placement="OnTicks"
/>
```

Correct:

```tsx
<ChartPrimaryXAxis valueType="Category">
  <ChartAxisLabel placement="OnTicks" />
</ChartPrimaryXAxis>
```

### Using `Numeric` as a value type

`Numeric` is not the documented numeric-axis literal. Use `Double` for numeric axes and `Category` for discrete category labels.

### Enabling indexed mode unnecessarily

Do not set `indexed={true}` simply because category values are strings. The ordinary Category axis already supports strings. Indexed mode changes point placement to use data indexes.

## Validation checklist

Before returning a Category-axis implementation:

1. Import `ChartPrimaryXAxis` and any used axis child components from `@syncfusion/react-charts`.
2. Set `valueType="Category"` on the intended axis.
3. Ensure `xField` maps to an existing discrete data property.
4. Preserve the intended category order in the bound data.
5. Use `ChartAxisLabel.placement` for `BetweenTicks` or `OnTicks`.
6. Do not confuse label `placement` with label `position`.
7. Treat `minimum` and `maximum` as category index positions.
8. Use a positive `interval` when explicitly configured.
9. Set `indexed={true}` only when index-based placement is required.
10. Put `maxLabelDensity` on the axis.
11. Put trimming, wrapping, rotation, and edge behavior on `ChartAxisLabel`.
12. Use a `DateTime` axis instead when elapsed-time spacing matters.
13. Emit valid, unescaped TSX.
