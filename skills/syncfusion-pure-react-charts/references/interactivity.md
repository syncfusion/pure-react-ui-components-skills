# Interactivity Reference

Use this umbrella reference to route interactive-chart requests to the correct focused page. In Pure React Chart, interactive behavior is configured through root-level child components such as `ChartTooltip`, `ChartZoomSettings`, `ChartSelection`, `ChartHighlight`, and `ChartCrosshair`, plus documented event props on the root `Chart`.

## Table of contents

1. [Tooltips](./interactivity/tooltips.md)
2. [Zooming and panning](./interactivity/zooming-panning.md)
3. [Selection and highlight](./interactivity/selection-highlight.md)
4. [Events](./interactivity/events.md)
5. [Crosshair](./interactivity/crosshair.md)
6. [Advanced patterns](./advanced-patterns.md)

## Ownership summary

Place these components directly inside `Chart`:

- `ChartTooltip`
- `ChartZoomSettings`
- `ChartSelection`
- `ChartHighlight`
- `ChartCrosshair`

Place `ChartCrosshairTooltip` inside each axis that requires a crosshair label.

Place documented event callbacks such as `onPointClick`, `onLegendClick`, and `onZoomEnd` on the root `Chart`.

## Combined hierarchy

```tsx
import {
  Chart,
  ChartCrosshair,
  ChartCrosshairTooltip,
  ChartHighlight,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSelection,
  ChartSeries,
  ChartSeriesCollection,
  ChartTooltip,
  ChartZoomSettings,
} from "@syncfusion/react-charts";
import type { PointClickEvent } from "@syncfusion/react-charts";

const handlePointClick = (args: PointClickEvent): void => {
  console.log(args.seriesIndex, args.pointIndex);
};

<Chart onPointClick={handlePointClick}>
  <ChartPrimaryXAxis valueType="Category">
    <ChartCrosshairTooltip enable={true} />
  </ChartPrimaryXAxis>

  <ChartPrimaryYAxis valueType="Double">
    <ChartCrosshairTooltip enable={true} />
  </ChartPrimaryYAxis>

  <ChartTooltip enable={true} />
  <ChartCrosshair enable={true} lineType="Both" />
  <ChartSelection mode="Point" />
  <ChartHighlight mode="Point" />
  <ChartZoomSettings mouseWheelZoom={true} mode="X" />

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="category"
      yField="value"
      type="Line"
    />
  </ChartSeriesCollection>
</Chart>
```

Do not enable every interaction automatically. Select only the features that support the chart's purpose.

## Tooltips

Use `ChartTooltip` for point information on hover or supported touch interaction.

```tsx
<ChartTooltip
  enable={true}
  shared={true}
  format="${series.name}: ${point.y}"
/>
```

Tooltips are disabled by default. Shared tooltips combine points with the same X value. Use a formatter or JSX template only when the built-in format is insufficient.

Read [Tooltips](./interactivity/tooltips.md) for formatting, templates, appearance, nearest-point behavior, fixed positioning, and series tooltip fields.

## Zooming and panning

Use `ChartZoomSettings` for drag-selection, wheel, pinch, panning, toolbars, and scrollbar activation.

```tsx
<ChartZoomSettings
  selectionZoom={true}
  mouseWheelZoom={true}
  pinchZoom={true}
  pan={true}
  mode="X"
  toolbar={{
    visible: true,
    items: ["ZoomIn", "ZoomOut", "Pan", "Reset"],
  }}
/>
```

The interaction flags default to `false`. Use only `X`, `Y`, or `XY` for `mode`.

Read [Zooming and panning](./interactivity/zooming-panning.md) before adding scrollbars, toolbar positioning, initial zoom ranges, or zoom events.

## Selection and highlight

Use `ChartSelection` for persistent click-based selection and `ChartHighlight` for temporary hover emphasis.

```tsx
<ChartSelection
  mode="Point"
  allowMultiSelection={true}
  selectedDataIndexes={[
    { seriesIndex: 0, pointIndex: 1 },
  ]}
  pattern="Crosshatch"
/>

<ChartHighlight
  mode="Series"
  fill="rgba(25, 118, 210, 0.25)"
  pattern="Dots"
/>
```

Supported modes are `None`, `Point`, `Series`, and `Cluster`. `selectedDataIndexes` requires objects containing `seriesIndex` and `pointIndex`.

Read [Selection and highlight](./interactivity/selection-highlight.md) for mode behavior, pattern values, multi-selection, preselection, and controlled selection patterns.

## Events

Use the exact documented `on`-prefixed event props on `Chart`.

```tsx
<Chart
  onClick={handleClick}
  onMouseMove={handleMouseMove}
  onPointClick={handlePointClick}
  onLegendClick={handleLegendClick}
  onZoomStart={handleZoomStart}
  onZoomEnd={handleZoomEnd}
>
  {/* chart children */}
</Chart>
```

The current root API documents:

- `onAxisLabelClick`
- `onClick`
- `onLegendClick`
- `onMouseEnter`
- `onMouseLeave`
- `onMouseMove`
- `onMultiLevelLabelClick`
- `onPointClick`
- `onResize`
- `onZoomEnd`
- `onZoomStart`

Keep high-frequency handlers lightweight. Import the exact event argument type and use only the fields it declares.

Read [Events](./interactivity/events.md) before generating event-specific logic.

## Crosshair

Use `ChartCrosshair` for pointer-following reference lines. Use `ChartCrosshairTooltip` inside each axis that needs an intersection label.

```tsx
<ChartPrimaryXAxis valueType="DateTime">
  <ChartCrosshairTooltip enable={true} />
</ChartPrimaryXAxis>

<ChartPrimaryYAxis valueType="Double">
  <ChartCrosshairTooltip enable={true} />
</ChartPrimaryYAxis>

<ChartCrosshair
  enable={true}
  lineType="Both"
  snap={true}
/>
```

Crosshair is disabled by default. Use only `Vertical`, `Horizontal`, or `Both` for `lineType`. Category highlighting applies to a Category axis.

Read [Crosshair](./interactivity/crosshair.md) for line styling, snapping, category highlighting, axis labels, and formatters.

## Choosing the right interaction

- Show point details without changing state: tooltip
- Read precise X and Y axis values: crosshair
- Keep a clicked point, series, or cluster emphasized: selection
- Emphasize an item only while hovering: highlight
- Explore a dense or long range: zooming and panning
- Trigger application behavior from user input: documented chart events
- Coordinate multiple charts or external controls: advanced patterns

Avoid overlapping interactions that compete for the same gesture. For example, confirm that click selection, point drill-down, and legend toggling still behave clearly when combined.

## Accessibility guidance

Interactive charts must not depend only on hover, color, mouse wheel, drag, or pinch gestures.

- Provide a meaningful chart accessibility label.
- Keep focus indicators visible.
- Test point, legend, and zoom-toolbar navigation with a keyboard.
- Provide non-color cues for selection and highlight states.
- Make essential values available through labels, summaries, or an accessible data table.
- Test touch interaction at supported mobile sizes.
- Verify that users can leave the chart without a keyboard trap.

Do not publish keyboard shortcuts unless they are documented for and tested with the installed Pure React version.

## Performance guidance

- Keep `onMouseMove` and zoom handlers lightweight.
- Avoid setting React state for every pointer movement unless necessary.
- Limit dense markers, labels, and selection states.
- Keep live-data windows bounded.
- Disable unnecessary animation during rapid updates.
- Profile combinations of shared tooltips, crosshair, markers, and zooming on target devices.

Do not use arbitrary point-count limits as universal performance rules.

## Routing rules

Use the focused page that matches the request:

- Hover or tap details: `tooltips.md`
- Magnification, navigation, toolbar, or scrollbars: `zooming-panning.md`
- Persistent selection or hover emphasis: `selection-highlight.md`
- Application callbacks and argument types: `events.md`
- Pointer-following axis reference lines: `crosshair.md`
- Drill-down, synchronized charts, streaming, or state integration: `advanced-patterns.md`

## Common errors

### Wrong component owner

Incorrect:

```tsx
<ChartSeries>
  <ChartTooltip enable={true} />
  <ChartCrosshair enable={true} />
  <ChartSelection mode="Point" />
</ChartSeries>
```

Correct:

```tsx
<Chart>
  <ChartTooltip enable={true} />
  <ChartCrosshair enable={true} />
  <ChartSelection mode="Point" />
</Chart>
```

### Wrong event names

Incorrect:

```tsx
<Chart
  pointClick={handlePointClick}
  legendClick={handleLegendClick}
  zoomComplete={handleZoomEnd}
/>
```

Correct:

```tsx
<Chart
  onPointClick={handlePointClick}
  onLegendClick={handleLegendClick}
  onZoomEnd={handleZoomEnd}
/>
```

### Mixed tooltip types

Use `ChartTooltip` for point content and `ChartCrosshairTooltip` for axis-intersection labels.

### Invalid selection indexes

Incorrect:

```tsx
selectedDataIndexes={[0, 2]}
```

Correct:

```tsx
selectedDataIndexes={[
  { seriesIndex: 0, pointIndex: 0 },
  { seriesIndex: 0, pointIndex: 2 },
]}
```

## Validation checklist

Before returning an interactive chart implementation:

1. Read every focused interactivity page relevant to the request.
2. Place root interaction components directly inside `Chart`.
3. Place `ChartCrosshairTooltip` inside its owning axis.
4. Use exact Pure React property and event names.
5. Import the documented event argument types.
6. Use only `X`, `Y`, or `XY` for zoom mode.
7. Use only `None`, `Point`, `Series`, or `Cluster` for selection and highlight mode.
8. Pass object-based indexes to `selectedDataIndexes`.
9. Keep point tooltips and crosshair labels separate.
10. Avoid competing gestures and unnecessary features.
11. Keep high-frequency handlers lightweight.
12. Provide keyboard and non-hover access to essential information.
13. Test keyboard, screen-reader, touch, and mobile behavior.
14. Do not mix EJ2 configuration with Pure React child components.
15. Ensure every imported symbol is used.
16. Emit valid, unescaped TSX.
