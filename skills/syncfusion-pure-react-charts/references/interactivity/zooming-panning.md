# Zooming & Panning Reference

Use `ChartZoomSettings` to configure selection zooming, mouse-wheel zooming, pinch zooming, panning, zoom direction, the zoom toolbar, and axis scrollbars. Place it directly inside `Chart`.

## Component hierarchy

```tsx
import {
  Chart,
  ChartSeries,
  ChartSeriesCollection,
  ChartZoomSettings,
} from "@syncfusion/react-charts";

<Chart>
  <ChartZoomSettings
    selectionZoom={true}
    mode="XY"
  />

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="x"
      yField="y"
      type="Line"
    />
  </ChartSeriesCollection>
</Chart>
```

Do not place `ChartZoomSettings` inside a series, axis, or series collection.

## Zoom settings properties

The Pure React `ChartZoomSettingsProps` API documents:

- `selectionZoom`: enables drag-selection zoom; default `false`
- `mouseWheelZoom`: enables mouse-wheel zoom; default `false`
- `pinchZoom`: enables touch pinch zoom; default `false`
- `pan`: enables direct panning of a zoomed chart; default `false`
- `mode`: controls the zoom direction; default `XY`
- `toolbar`: configures zoom controls; default toolbar visibility is `false`
- `enableScrollbar`: renders scrollbars for eligible zoomed axes; default `false`
- `accessibility`: accessibility settings for zoom-related UI

## Selection zoom

Set `selectionZoom={true}` to let users drag a rectangular region and zoom into it.

```tsx
<ChartZoomSettings
  selectionZoom={true}
  mode="XY"
/>
```

The earlier `selectionZoom` property name is correct for Pure React. Do not replace it with an unverified name such as `enableSelectionZooming`.

Selection zoom is disabled by default.

## Mouse-wheel zoom

```tsx
<ChartZoomSettings
  mouseWheelZoom={true}
  mode="X"
/>
```

Mouse-wheel zoom is disabled by default. Use it when desktop users need quick zooming without drawing a selection rectangle.

## Pinch zoom

```tsx
<ChartZoomSettings
  pinchZoom={true}
  mode="X"
/>
```

Pinch zoom is designed for touch-enabled devices and is disabled by default.

## Panning

Set `pan={true}` to allow direct dragging across an already zoomed chart.

```tsx
<ChartZoomSettings
  pan={true}
  mode="X"
/>
```

Panning does not create a zoomed range on its own. Combine it with a zoom interaction, an initially zoomed axis, or a scrollbar when navigation across a reduced visible range is required.

## Zoom mode

Use exact `mode` values:

- `X`: horizontal zooming only
- `Y`: vertical zooming only
- `XY`: both horizontal and vertical zooming

```tsx
<ChartZoomSettings mode="X" />
<ChartZoomSettings mode="Y" />
<ChartZoomSettings mode="XY" />
```

The documented default is `XY`.

The API notes that `mode` controls selection-zoom direction when `selectionZoom={true}`. The selected mode also determines which axes are eligible for scrollbar rendering.

## Combining zoom methods

More than one zoom method can be enabled.

```tsx
<ChartZoomSettings
  selectionZoom={true}
  mouseWheelZoom={true}
  pinchZoom={true}
  pan={true}
  mode="X"
/>
```

Choose only the interactions needed by the sample. Avoid enabling every interaction automatically when a simpler configuration is sufficient.

## Zoom toolbar

Configure the toolbar through `ChartZoomSettings.toolbar`.

```tsx
<ChartZoomSettings
  selectionZoom={true}
  toolbar={{
    visible: true,
    items: ["ZoomIn", "ZoomOut", "Pan", "Reset"],
    position: {
      hAlign: "Center",
      vAlign: "Top",
      x: 0,
      y: 0,
    },
  }}
/>
```

The documented toolbar items are:

- `ZoomIn`
- `ZoomOut`
- `Pan`
- `Reset`

The default item order includes all four. Toolbar `visible` defaults to `false`.

## Toolbar position

Use exact alignment values:

- `hAlign`: `Left`, `Center`, or `Right`
- `vAlign`: `Top`, `Center`, or `Bottom`
- `x`: horizontal pixel offset
- `y`: vertical pixel offset

```tsx
<ChartZoomSettings
  toolbar={{
    visible: true,
    position: {
      hAlign: "Right",
      vAlign: "Top",
      x: 0,
      y: 0,
    },
  }}
/>
```

The documented default position is right-aligned at the top with zero offsets.

## Scrollbars

Set `enableScrollbar={true}` on `ChartZoomSettings`, then configure `ChartScrollbar` inside each axis that needs a scrollbar.

```tsx
import {
  ChartPrimaryXAxis,
  ChartScrollbar,
  ChartZoomSettings,
} from "@syncfusion/react-charts";

<Chart>
  <ChartZoomSettings
    enableScrollbar={true}
    mode="X"
  />

  <ChartPrimaryXAxis valueType="DateTime">
    <ChartScrollbar
      enable={true}
      enableZoom={true}
      thickness={15}
      thumbColor="#4ECDC4"
      trackColor="#E0E0E0"
      thumbRadius={3}
      trackRadius={3}
    />
  </ChartPrimaryXAxis>
</Chart>
```

Do not pass an invented `scrollbar={{ ... }}` object to `ChartZoomSettings`.

Incorrect:

```tsx
<ChartZoomSettings
  enableScrollbar={true}
  scrollbar={{
    enable: true,
    height: 15,
  }}
/>
```

The Pure React scrollbar is axis-owned configuration.

## Scrollbar properties

`ChartScrollbarProps` documents:

- `enable`: enables the scrollbar for its axis; default `true`
- `enableZoom`: allows resizing the selected range; default `false`
- `position`: places the scrollbar relative to the axis
- `thickness`: scrollbar thickness; default `14`
- `thumbColor`: thumb fill color
- `thumbRadius`: thumb corner radius; default `6`
- `trackColor`: track background color
- `trackRadius`: track corner radius; default `6`
- `resizeCircle`: styles scrollbar resize handles

Use `thickness`, not `height`.

## Scrollbar position

Use exact position values:

- `Top`: horizontal scrollbar at the top
- `Bottom`: horizontal scrollbar at the bottom
- `Left`: vertical scrollbar on the left
- `Right`: vertical scrollbar on the right
- `PlaceNextToAxisLine`: places it next to the axis line

```tsx
<ChartPrimaryXAxis valueType="DateTime">
  <ChartScrollbar
    enable={true}
    position="Bottom"
  />
</ChartPrimaryXAxis>
```

Use horizontal positions only for horizontal axes and vertical positions only for vertical axes.

## Panning-only scrollbar

Keep `enableZoom={false}` when the scrollbar should change only the visible position.

```tsx
<ChartScrollbar
  enable={true}
  enableZoom={false}
/>
```

When `enableZoom={true}`, users can resize the thumb to change the visible zoom range.

## Initial zoom range

Configure initial zoom through the owning axis when its API provides `zoomFactor` and `zoomPosition`.

```tsx
<ChartPrimaryXAxis
  valueType="DateTime"
  zoomFactor={0.4}
  zoomPosition={0.3}
>
  <ChartScrollbar enable={true} />
</ChartPrimaryXAxis>
```

Keep zoom values within the ranges documented by the axis API. A scrollbar is rendered when the visible range is smaller than the full data range and scrollbar rendering is enabled.

## Zoom events

The root `Chart` API documents `onZoomStart` and `onZoomEnd`.

```tsx
import type {
  ZoomEndEvent,
  ZoomStartEvent,
} from "@syncfusion/react-charts";

const handleZoomStart = (
  args: ZoomStartEvent,
): void => {
  console.log(args);
};

const handleZoomEnd = (
  args: ZoomEndEvent,
): void => {
  console.log(args);
};

<Chart
  onZoomStart={handleZoomStart}
  onZoomEnd={handleZoomEnd}
>
  <ChartZoomSettings selectionZoom={true} />
</Chart>
```

Do not use `onZoomComplete`, `zoomComplete`, or `zooming` unless a future Pure React API documents them.

Keep zoom handlers lightweight because they run during user interaction.

## Programmatic zoom

Do not assume the Pure React `Chart` ref exposes `zoom()` or `resetZoom()` methods. These methods are not documented by the current root Chart API page.

Do not generate this unverified pattern:

```tsx
const chartRef = useRef(null);

chartRef.current.zoom(1.2);
chartRef.current.resetZoom();
```

Use the built-in toolbar for Zoom In, Zoom Out, Pan, and Reset. For application-controlled ranges, update documented axis zoom properties through React state only when the current axis API supports that workflow.

## Complete example

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartScrollbar,
  ChartSeries,
  ChartSeriesCollection,
  ChartZoomSettings,
} from "@syncfusion/react-charts";

const data = [
  { date: new Date(2026, 0, 1), value: 32 },
  { date: new Date(2026, 1, 1), value: 38 },
  { date: new Date(2026, 2, 1), value: 35 },
  { date: new Date(2026, 3, 1), value: 47 },
  { date: new Date(2026, 4, 1), value: 44 },
  { date: new Date(2026, 5, 1), value: 53 },
  { date: new Date(2026, 6, 1), value: 58 },
  { date: new Date(2026, 7, 1), value: 55 },
  { date: new Date(2026, 8, 1), value: 64 },
  { date: new Date(2026, 9, 1), value: 61 },
  { date: new Date(2026, 10, 1), value: 69 },
  { date: new Date(2026, 11, 1), value: 74 },
];

export default function ZoomingPanningChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis
        valueType="DateTime"
        interval={1}
        intervalType="Months"
      >
        <ChartAxisTitle text="Month" />
        <ChartAxisLabel format="MMM" />
        <ChartScrollbar
          enable={true}
          enableZoom={true}
          position="Bottom"
          thickness={14}
          thumbColor="#607D8B"
          trackColor="#ECEFF1"
          thumbRadius={6}
          trackRadius={6}
        />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double">
        <ChartAxisTitle text="Value" />
        <ChartAxisLabel format="{value}" />
      </ChartPrimaryYAxis>

      <ChartZoomSettings
        selectionZoom={true}
        mouseWheelZoom={true}
        pinchZoom={true}
        pan={true}
        mode="X"
        enableScrollbar={true}
        toolbar={{
          visible: true,
          items: ["ZoomIn", "ZoomOut", "Pan", "Reset"],
          position: {
            hAlign: "Right",
            vAlign: "Top",
            x: 0,
            y: 0,
          },
        }}
      />

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="date"
          yField="value"
          type="Line"
          name="Value"
          width={2}
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Migration from EJ2 React

Do not use EJ2 root zoom settings in a Pure React chart.

EJ2-style configuration:

```tsx
<ChartComponent
  zoomSettings={{
    enableSelectionZooming: true,
    enableMouseWheelZooming: true,
    enablePinchZooming: true,
    enablePan: true,
    mode: "X",
  }}
/>
```

Pure React component configuration:

```tsx
<Chart>
  <ChartZoomSettings
    selectionZoom={true}
    mouseWheelZoom={true}
    pinchZoom={true}
    pan={true}
    mode="X"
  />
</Chart>
```

Use the Pure React component and shortened property names.

## Common errors

### Wrong property names

Incorrect:

```tsx
<ChartZoomSettings
  enableSelectionZooming={true}
  enableMouseWheelZooming={true}
  enablePinchZooming={true}
  enablePan={true}
/>
```

Correct:

```tsx
<ChartZoomSettings
  selectionZoom={true}
  mouseWheelZoom={true}
  pinchZoom={true}
  pan={true}
/>
```

### Wrong scrollbar configuration

Do not pass `scrollbar` to `ChartZoomSettings`. Place `ChartScrollbar` inside its owning axis.

### Wrong scrollbar thickness property

Use `thickness`, not `height`.

### Unsupported programmatic methods

Do not call `chartRef.current.zoom()` or `resetZoom()` without a current Pure React method definition.

### Wrong zoom events

Use `onZoomStart` and `onZoomEnd`, not `zoomStart`, `zoomComplete`, or `onZoomComplete`.

## Validation checklist

Before returning a zooming or panning implementation:

1. Import `ChartZoomSettings` from `@syncfusion/react-charts`.
2. Place it directly inside `Chart`.
3. Use `selectionZoom`, `mouseWheelZoom`, `pinchZoom`, and `pan`.
4. Use only `X`, `Y`, or `XY` for `mode`.
5. Remember that all interaction flags default to `false`.
6. Configure toolbar settings through `toolbar`.
7. Use only `ZoomIn`, `ZoomOut`, `Pan`, and `Reset` toolbar items.
8. Use valid toolbar alignment and offset properties.
9. Set `enableScrollbar={true}` when scrollbars are required.
10. Place `ChartScrollbar` inside each owning axis.
11. Use `thickness`, not `height`, for scrollbar size.
12. Use `enableZoom` to control thumb resizing.
13. Match scrollbar position to axis orientation.
14. Use `onZoomStart` and `onZoomEnd` for documented zoom events.
15. Do not invent programmatic zoom methods.
16. Do not mix EJ2 `zoomSettings` objects with Pure React components.
17. Ensure every imported symbol is used.
18. Emit valid, unescaped TSX.
