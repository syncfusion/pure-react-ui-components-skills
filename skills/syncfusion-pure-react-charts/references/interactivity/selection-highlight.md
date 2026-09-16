# Selection & Highlight Reference

Use `ChartSelection` for persistent click-based selection and `ChartHighlight` for temporary hover emphasis. Place both components directly inside `Chart`.

## Component hierarchy

```tsx
import {
  Chart,
  ChartHighlight,
  ChartSelection,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

<Chart>
  <ChartSelection mode="Point" />
  <ChartHighlight mode="Point" />

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

Do not place `ChartSelection` or `ChartHighlight` inside a series, axis, or series collection.

## Selection properties

`ChartSelectionProps` documents:

- `mode`: selection scope; default `None`
- `allowMultiSelection`: enables multiple selected items; default `false`
- `selectedDataIndexes`: points selected during initial rendering; default `[]`
- `pattern`: visual pattern for selected elements; default `None`

## Selection modes

```tsx
<ChartSelection mode="Point" />
<ChartSelection mode="Series" />
<ChartSelection mode="Cluster" />
<ChartSelection mode="None" />
```

Use exact mode values:

- `Point`: selects one data point
- `Series`: selects the entire series
- `Cluster`: selects related points sharing the same X value across series
- `None`: disables selection

Do not use lowercase values or unverified modes.

## Basic point selection

```tsx
<Chart>
  <ChartSelection mode="Point" />

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

Selection persists after the pointer leaves the selected element. Use highlight instead when only hover feedback is needed.

## Series selection

```tsx
<ChartSelection mode="Series" />
```

Use series selection when the complete series should be emphasized as one item.

## Cluster selection

```tsx
<ChartSelection mode="Cluster" />
```

Cluster selection is useful in multi-series charts where users compare all points at a shared X value.

```tsx
<ChartSeriesCollection>
  <ChartSeries
    dataSource={data}
    xField="month"
    yField="sales"
    type="Column"
    name="Sales"
  />
  <ChartSeries
    dataSource={data}
    xField="month"
    yField="revenue"
    type="Column"
    name="Revenue"
  />
</ChartSeriesCollection>
```

Keep X values compatible across the participating series so the cluster has a clear meaning.

## Multiple selection

Set `allowMultiSelection={true}` with `Point`, `Series`, or `Cluster` mode.

```tsx
<ChartSelection
  mode="Point"
  allowMultiSelection={true}
/>
```

The documented default is `false`. Do not enable multi-selection while leaving `mode="None"`.

Do not hard-code platform-specific keyboard instructions unless the current interaction documentation explicitly requires a modifier key. The component property itself controls whether multiple items may be selected.

## Preselected points

`selectedDataIndexes` requires an array of `ChartIndexesProps` objects. Each object identifies both the series and point.

```tsx
<ChartSelection
  mode="Point"
  selectedDataIndexes={[
    { seriesIndex: 0, pointIndex: 0 },
    { seriesIndex: 0, pointIndex: 2 },
    { seriesIndex: 0, pointIndex: 4 },
  ]}
/>
```

Do not pass a plain number array.

Incorrect:

```tsx
<ChartSelection
  mode="Point"
  selectedDataIndexes={[0, 2, 4]}
/>
```

`seriesIndex` and `pointIndex` are zero-based. Ensure every index exists in the rendered series and its data source.

### Preselect across multiple series

```tsx
<ChartSelection
  mode="Point"
  allowMultiSelection={true}
  selectedDataIndexes={[
    { seriesIndex: 0, pointIndex: 1 },
    { seriesIndex: 1, pointIndex: 1 },
  ]}
/>
```

## Selection patterns

Use exact values from the `SelectionPattern` union:

- `None`
- `Chessboard`
- `Dots`
- `DiagonalForward`
- `Crosshatch`
- `Pacman`
- `DiagonalBackward`
- `Grid`
- `Turquoise`
- `Star`
- `Triangle`
- `Circle`
- `Tile`
- `HorizontalDash`
- `VerticalDash`
- `Rectangle`
- `Box`
- `VerticalStripe`
- `HorizontalStripe`
- `Bubble`

```tsx
<ChartSelection
  mode="Point"
  pattern="Crosshatch"
/>
```

Use `Crosshatch`, not `CrossHatch`. Do not use unsupported names such as `Diagonals`, `LightBlue`, `Orange`, or `Pink`.

Patterns help distinguish selected elements without relying only on color.

## Highlight properties

`ChartHighlightProps` documents:

- `mode`: highlight scope; default `None`
- `fill`: hover color; default empty
- `pattern`: visual highlight pattern; default `None`

Highlight uses the same documented `None`, `Series`, `Point`, and `Cluster` modes as selection.

## Basic highlight

```tsx
<Chart>
  <ChartHighlight mode="Point" />

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

Highlight is temporary hover emphasis and does not replace persistent selection.

## Highlight customization

```tsx
<ChartHighlight
  mode="Point"
  fill="rgba(255, 0, 0, 0.35)"
  pattern="Dots"
/>
```

Use a valid CSS color for `fill`. Keep enough contrast between the normal and highlighted states.

## Series and cluster highlight

```tsx
<ChartHighlight mode="Series" />
```

Use series mode to emphasize the complete hovered series.

```tsx
<ChartHighlight mode="Cluster" />
```

Use cluster mode to emphasize related points at the same X value across series.

## Combined selection and highlight

Selection and highlight can use different modes and treatments.

```tsx
<Chart>
  <ChartSelection
    mode="Point"
    allowMultiSelection={true}
    pattern="Crosshatch"
  />

  <ChartHighlight
    mode="Series"
    fill="rgba(25, 118, 210, 0.25)"
    pattern="None"
  />

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

Keep the selected and highlighted states visually distinct. For example, use a pattern for persistent selection and a translucent fill for hover highlight.

## Selection change handling

The current Pure React `Chart` event table does not document `onSelectionChanged`. Do not generate that event based only on an older EJ2 example or the supplied draft.

Incorrect without current API support:

```tsx
const handleSelectionChanged = (args) => {
  console.log(args.selectedDataIndexes);
};

<Chart onSelectionChanged={handleSelectionChanged} />
```

When the application needs click information, use the documented `onPointClick` event and manage application state separately.

```tsx
import type { PointClickEvent } from "@syncfusion/react-charts";

const handlePointClick = (
  args: PointClickEvent,
): void => {
  console.log(args.seriesIndex, args.pointIndex);
};

<Chart onPointClick={handlePointClick}>
  <ChartSelection mode="Point" />
  {/* chart children */}
</Chart>
```

`onPointClick` reports the clicked point. It is not a replacement for a dedicated event that reports the complete current selection set.

## Controlled preselection

Use React state with `selectedDataIndexes` when the application must provide or update selected indexes declaratively.

```tsx
import { useState } from "react";
import type {
  ChartIndexesProps,
  PointClickEvent,
} from "@syncfusion/react-charts";

const [selectedIndexes, setSelectedIndexes] =
  useState<ChartIndexesProps[]>([]);

const handlePointClick = (
  args: PointClickEvent,
): void => {
  setSelectedIndexes([
    {
      seriesIndex: args.seriesIndex,
      pointIndex: args.pointIndex,
    },
  ]);
};

<Chart onPointClick={handlePointClick}>
  <ChartSelection
    mode="Point"
    selectedDataIndexes={selectedIndexes}
  />
</Chart>
```

Use this pattern only when application-controlled selection is required. For ordinary built-in interaction, `ChartSelection` can manage the visual state without extra React state.

## Complete example

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartHighlight,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSelection,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { month: "Jan", sales: 35, revenue: 42 },
  { month: "Feb", sales: 42, revenue: 48 },
  { month: "Mar", sales: 38, revenue: 45 },
  { month: "Apr", sales: 51, revenue: 59 },
  { month: "May", sales: 47, revenue: 63 },
];

export default function SelectionHighlightChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis valueType="Category">
        <ChartAxisTitle text="Month" />
        <ChartAxisLabel edgeLabelPlacement="Shift" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double" minimum={0}>
        <ChartAxisTitle text="Value" />
        <ChartAxisLabel format="{value}" />
      </ChartPrimaryYAxis>

      <ChartSelection
        mode="Point"
        allowMultiSelection={true}
        selectedDataIndexes={[
          { seriesIndex: 0, pointIndex: 1 },
        ]}
        pattern="Crosshatch"
      />

      <ChartHighlight
        mode="Cluster"
        fill="rgba(25, 118, 210, 0.25)"
        pattern="None"
      />

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="sales"
          type="Column"
          name="Sales"
        />
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="revenue"
          type="Column"
          name="Revenue"
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Migration from EJ2 React

Do not use EJ2 root-object settings in a Pure React chart.

EJ2-style configuration:

```tsx
<ChartComponent
  selectionMode="Point"
  highlightMode="Series"
  isMultiSelect={true}
  selectedDataIndexes={[
    { series: 0, point: 1 },
  ]}
/>
```

Pure React component configuration:

```tsx
<Chart>
  <ChartSelection
    mode="Point"
    allowMultiSelection={true}
    selectedDataIndexes={[
      { seriesIndex: 0, pointIndex: 1 },
    ]}
  />
  <ChartHighlight mode="Series" />
</Chart>
```

Use Pure React child components and Pure React index property names.

## Common errors

### Wrong hierarchy

Incorrect:

```tsx
<ChartSeries>
  <ChartSelection mode="Point" />
</ChartSeries>
```

Correct:

```tsx
<Chart>
  <ChartSelection mode="Point" />
</Chart>
```

### Number-only selected indexes

Incorrect:

```tsx
selectedDataIndexes={[0, 2, 4]}
```

Correct:

```tsx
selectedDataIndexes={[
  { seriesIndex: 0, pointIndex: 0 },
  { seriesIndex: 0, pointIndex: 2 },
  { seriesIndex: 0, pointIndex: 4 },
]}
```

### Wrong pattern casing

Incorrect:

```tsx
pattern="CrossHatch"
```

Correct:

```tsx
pattern="Crosshatch"
```

### Unsupported patterns

Do not use `Diagonals`, `LightBlue`, `Orange`, or `Pink`. Use an exact current `SelectionPattern` value.

### Unverified selection event

Do not use `onSelectionChanged` unless a future Pure React `Chart` API explicitly documents it.

### Multi-selection without an active mode

Incorrect:

```tsx
<ChartSelection
  mode="None"
  allowMultiSelection={true}
/>
```

Use `Point`, `Series`, or `Cluster` when multi-selection is required.

## Validation checklist

Before returning a selection or highlight implementation:

1. Import `ChartSelection` and `ChartHighlight` from `@syncfusion/react-charts` as needed.
2. Place both components directly inside `Chart`.
3. Use only `None`, `Series`, `Point`, or `Cluster` for `mode`.
4. Use `allowMultiSelection` only with an active selection mode.
5. Pass `selectedDataIndexes` as `ChartIndexesProps[]`.
6. Include both `seriesIndex` and `pointIndex` for each preselected point.
7. Keep all indexes zero-based and within the rendered data bounds.
8. Use only exact documented `SelectionPattern` values.
9. Use `Crosshatch`, not `CrossHatch`.
10. Use `fill` only on `ChartHighlight`; `ChartSelectionProps` does not document a selection fill property.
11. Keep selection and highlight states visually distinct.
12. Do not use `onSelectionChanged` unless it appears in the current Pure React API.
13. Use `onPointClick` only when clicked-point information is sufficient.
14. Do not mix EJ2 root props with Pure React child components.
15. Ensure every imported symbol is used.
16. Emit valid, unescaped TSX.
