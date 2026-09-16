# Events Reference

Use chart events to respond to pointer interaction, point clicks, legend clicks, axis-label clicks, resizing, and zooming. In Pure React Chart, the documented event callbacks belong to the root `Chart` component.

## Event naming rule

Pure React Chart event names use the `on` prefix. Use the exact names documented by the current `Chart` API.

```tsx
<Chart
  onClick={handleClick}
  onMouseMove={handleMouseMove}
  onLegendClick={handleLegendClick}
/>
```

Do not remove the `on` prefix or reuse older EJ2 event names.

Incorrect:

```tsx
<Chart
  click={handleClick}
  mouseMove={handleMouseMove}
  legendClick={handleLegendClick}
/>
```

## Documented Chart events

The Pure React `Chart` API documents these events:

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

Do not add events that are absent from the current `Chart` API.

## Mouse events

Use `ChartMouseEvent` for the documented chart mouse callbacks.

```tsx
import type { ChartMouseEvent } from "@syncfusion/react-charts";

const handleClick = (args: ChartMouseEvent): void => {
  console.log(args.target);
};

const handleMouseEnter = (args: ChartMouseEvent): void => {
  console.log(args.target);
};

const handleMouseLeave = (args: ChartMouseEvent): void => {
  console.log(args.target);
};

const handleMouseMove = (args: ChartMouseEvent): void => {
  console.log(args.target);
};

<Chart
  onClick={handleClick}
  onMouseEnter={handleMouseEnter}
  onMouseLeave={handleMouseLeave}
  onMouseMove={handleMouseMove}
>
  {/* chart children */}
</Chart>
```

Read only fields declared by `ChartMouseEvent`. Do not assume that a chart mouse event is identical to a browser or React mouse event.

Keep `onMouseMove` lightweight because it may run frequently while the pointer moves across the chart.

## Point click

Use `onPointClick` with `PointClickEvent` when the user must interact with a specific rendered point.

```tsx
import type { PointClickEvent } from "@syncfusion/react-charts";

const handlePointClick = (args: PointClickEvent): void => {
  console.log(args.seriesIndex, args.pointIndex);
};

<Chart onPointClick={handlePointClick}>
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

Use the event's documented series index, point index, position, and pointer-coordinate fields. Do not attach `onPointClick` to `ChartSeries`.

## Point render callback

`pointRender` is a root `Chart` prop, but the API lists it under component props rather than the Chart events table. It customizes a point color during rendering.

```tsx
import type { PointRenderProps } from "@syncfusion/react-charts";

const handlePointRender = (
  args: PointRenderProps,
): string => {
  return Number(args.yValue) > 1000
    ? "#D32F2F"
    : args.color;
};

<Chart pointRender={handlePointRender}>
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

`PointRenderProps` exposes:

- `color`
- `seriesIndex`
- `xValue`
- `yValue`

Return the required color. Do not mutate `args.fill`.

Incorrect:

```tsx
const handlePointRender = (args) => {
  args.fill = "#D32F2F";
};

<ChartSeries onPointRender={handlePointRender} />
```

## Legend click

Use `onLegendClick` with `LegendClickEvent`.

```tsx
import type { LegendClickEvent } from "@syncfusion/react-charts";

const handleLegendClick = (
  args: LegendClickEvent,
): void => {
  console.log(args.seriesName, args.text);

  if (args.seriesName === "Reference") {
    args.cancel = true;
  }
};

<Chart onLegendClick={handleLegendClick}>
  <ChartLegend visible={true} />

  <ChartSeriesCollection>
    <ChartSeries name="Sales" />
    <ChartSeries name="Reference" />
  </ChartSeriesCollection>
</Chart>
```

`LegendClickEvent` provides:

- `cancel`
- `seriesName`
- `shape`
- `text`

Set `args.cancel = true` to cancel the default action. Do not use `preventDefault`.

Incorrect:

```tsx
args.preventDefault = true;
```

## Axis-label click

Use `onAxisLabelClick` on `Chart` with `AxisLabelClickEvent`.

```tsx
import type { AxisLabelClickEvent } from "@syncfusion/react-charts";

const handleAxisLabelClick = (
  args: AxisLabelClickEvent,
): void => {
  console.log(args.axisName, args.text, args.value);
};

<Chart onAxisLabelClick={handleAxisLabelClick}>
  <ChartPrimaryXAxis valueType="Category" />
</Chart>
```

`AxisLabelClickEvent` provides:

- `axisName`
- `index`
- `location`
- `text`
- `value`

Do not attach the callback directly to `ChartPrimaryXAxis`, `ChartPrimaryYAxis`, or `ChartAxis`.

Incorrect:

```tsx
<ChartPrimaryXAxis
  onAxisLabelClick={handleAxisLabelClick}
/>
```

## Multi-level-label click

Use `onMultiLevelLabelClick` with `MultiLevelLabelClickEvent`.

```tsx
import type {
  MultiLevelLabelClickEvent,
} from "@syncfusion/react-charts";

const handleMultiLevelLabelClick = (
  args: MultiLevelLabelClickEvent,
): void => {
  console.log(args.axisName, args.text);
};

<Chart
  onMultiLevelLabelClick={handleMultiLevelLabelClick}
>
  {/* axes and series */}
</Chart>
```

Use this event only when the chart contains multi-level axis labels. Read only the axis, text, level, range, and other fields documented by the event type.

## Resize event

Use `onResize` with `ResizeEvent`.

```tsx
import type { ResizeEvent } from "@syncfusion/react-charts";

const handleResize = (args: ResizeEvent): void => {
  console.log(args);
};

<Chart onResize={handleResize}>
  {/* chart children */}
</Chart>
```

The event provides details about the chart size before and after resizing. Use the exact size fields declared by `ResizeEvent`; do not assume direct `args.width` and `args.height` properties.

Avoid unnecessary React state updates on every resize notification.

## Zoom start

Use `onZoomStart` with `ZoomStartEvent`.

```tsx
import type { ZoomStartEvent } from "@syncfusion/react-charts";

const handleZoomStart = (
  args: ZoomStartEvent,
): void => {
  console.log(args);
};

<Chart onZoomStart={handleZoomStart}>
  <ChartZoomSettings
    enableSelectionZooming={true}
  />
</Chart>
```

Use only fields declared by `ZoomStartEvent`. Do not assume EJ2 zoom argument names or cancellation behavior.

## Zoom end

Use `onZoomEnd` with `ZoomEndEvent`.

```tsx
import type { ZoomEndEvent } from "@syncfusion/react-charts";

const handleZoomEnd = (
  args: ZoomEndEvent,
): void => {
  console.log(args);
};

<Chart onZoomEnd={handleZoomEnd}>
  <ChartZoomSettings
    enableSelectionZooming={true}
  />
</Chart>
```

The event provides information about the affected axis, zoom factor, zoom position, and visible ranges. Keep the handler lightweight.

Do not use `onZoomComplete`, `zoomComplete`, or `zooming` unless a future Pure React API explicitly documents them.

## Complete example

```tsx
import { useState } from "react";
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartLegend,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";
import type {
  AxisLabelClickEvent,
  ChartMouseEvent,
  LegendClickEvent,
  PointClickEvent,
  PointRenderProps,
  ResizeEvent,
} from "@syncfusion/react-charts";

const data = [
  { category: "A", value: 850 },
  { category: "B", value: 1250 },
  { category: "C", value: 940 },
  { category: "D", value: 1380 },
];

export default function ChartEventsExample() {
  const [selectedLabel, setSelectedLabel] = useState<string | null>(null);

  const handlePointRender = (
    args: PointRenderProps,
  ): string => {
    return Number(args.yValue) > 1000
      ? "#D32F2F"
      : args.color;
  };

  const handleClick = (
    args: ChartMouseEvent,
  ): void => {
    console.log(args.target);
  };

  const handlePointClick = (
    args: PointClickEvent,
  ): void => {
    console.log(args.seriesIndex, args.pointIndex);
  };

  const handleLegendClick = (
    args: LegendClickEvent,
  ): void => {
    console.log(args.seriesName, args.text);
  };

  const handleAxisLabelClick = (
    args: AxisLabelClickEvent,
  ): void => {
    setSelectedLabel(args.text);
  };

  const handleResize = (
    args: ResizeEvent,
  ): void => {
    console.log(args);
  };

  return (
    <section>
      <p>
        Selected category: {selectedLabel ?? "None"}
      </p>

      <Chart
        pointRender={handlePointRender}
        onClick={handleClick}
        onPointClick={handlePointClick}
        onLegendClick={handleLegendClick}
        onAxisLabelClick={handleAxisLabelClick}
        onResize={handleResize}
      >
        <ChartPrimaryXAxis valueType="Category">
          <ChartAxisTitle text="Category" />
          <ChartAxisLabel edgeLabelPlacement="Shift" />
        </ChartPrimaryXAxis>

        <ChartPrimaryYAxis valueType="Double" minimum={0}>
          <ChartAxisTitle text="Value" />
          <ChartAxisLabel format="{value}" />
        </ChartPrimaryYAxis>

        <ChartLegend visible={true} />

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
    </section>
  );
}
```

## Unsupported event patterns

Do not generate these names for the current Pure React Chart API:

```tsx
<Chart
  onChartDoubleClick={handleDoubleClick}
  onChartMouseLeave={handleMouseLeave}
  onChartLoad={handleLoad}
  onChartResize={handleResize}
  onAnimationComplete={handleAnimationComplete}
  onSelectionChanged={handleSelectionChanged}
  onZoomComplete={handleZoomComplete}
/>
```

Also do not generate root event names without `on`:

```tsx
<Chart
  click={handleClick}
  legendClick={handleLegendClick}
  axisLabelClick={handleAxisLabelClick}
  resized={handleResize}
  zoomStart={handleZoomStart}
  zoomEnd={handleZoomEnd}
/>
```

Do not put assumed render events on child components:

```tsx
<ChartSeries onSeriesRender={handleSeriesRender} />
<ChartSeries onPointRender={handlePointRender} />
<ChartDataLabel onDataLabelRender={handleDataLabelRender} />
<ChartTooltip onTooltipRender={handleTooltipRender} />
```

## Validation checklist

Before returning an event implementation:

1. Verify the event against the current Pure React `Chart` API.
2. Use the exact `on`-prefixed event name.
3. Place documented chart events on the root `Chart`.
4. Import the exact exported argument type.
5. Use only fields declared by that event type.
6. Use `onClick`, `onMouseEnter`, `onMouseLeave`, and `onMouseMove` for mouse interaction.
7. Use `onPointClick` for a rendered-point click.
8. Use `onLegendClick` with `LegendClickEvent`.
9. Use `args.cancel`, not `preventDefault`, for legend cancellation.
10. Use `onAxisLabelClick` on `Chart`, not on an axis.
11. Use `onMultiLevelLabelClick` only for multi-level labels.
12. Use `onResize` for chart resizing.
13. Use `onZoomStart` and `onZoomEnd` for documented zoom events.
14. Treat `pointRender` as a root callback prop that returns a color.
15. Do not mutate `PointRenderProps` with an invented `fill` field.
16. Do not invent series, data-label, tooltip, load, animation, selection, or double-click events.
17. Do not mix EJ2 event names or argument contracts with Pure React.
18. Keep high-frequency handlers lightweight.
19. Ensure every imported symbol is used.
20. Emit valid, unescaped TSX.
