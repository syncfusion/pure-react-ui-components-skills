# Events Reference

Use the event callback props exported by `@syncfusion/react-charts` on the root `Chart` component. Use this umbrella reference together with [`interactivity/events.md`](./interactivity/events.md) for complete event-specific signatures, examples, and validation rules.

## Common events

```tsx
import { Chart } from "@syncfusion/react-charts";
import type {
  ChartMouseEvent,
  LegendClickEvent,
  PointClickEvent,
} from "@syncfusion/react-charts";

const handleClick = (args: ChartMouseEvent): void => {
  console.log(args.target, args.x, args.y);
};

const handleMouseMove = (args: ChartMouseEvent): void => {
  console.log(args.target);
};

const handlePointClick = (args: PointClickEvent): void => {
  console.log(args.seriesIndex, args.pointIndex);
};

const handleLegendClick = (args: LegendClickEvent): void => {
  console.log(args.seriesName, args.text);
};

<Chart
  onClick={handleClick}
  onMouseMove={handleMouseMove}
  onPointClick={handlePointClick}
  onLegendClick={handleLegendClick}
>
  {/* chart children */}
</Chart>
```

## Documented event names

The current Pure React `Chart` API documents:

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

Use the exact `on`-prefixed names. Do not reuse EJ2 names such as `click`, `legendClick`, `mouseMove`, `zoomStart`, or `zoomComplete`.

## Handler rules

- Place documented interaction events on the root `Chart`.
- Import and use the exact exported event argument type.
- Use `ChartMouseEvent` only for callbacks whose API signature declares it.
- Use `PointClickEvent` when the interaction must identify a rendered point.
- Use `LegendClickEvent.seriesName` and `LegendClickEvent.text` for legend details.
- Set `args.cancel = true` to cancel the default legend action when required.
- Keep `onMouseMove` and zoom handlers lightweight because they may run frequently.
- Avoid unnecessary React state updates in high-frequency handlers.
- Do not attach assumed event props to series, axes, labels, legends, or tooltips.

## Render callback distinction

`pointRender` is a root `Chart` callback prop, but it is listed under chart props rather than the event table. It receives `PointRenderProps` and returns a color.

```tsx
import type { PointRenderProps } from "@syncfusion/react-charts";

const handlePointRender = (
  args: PointRenderProps,
): string => {
  return Number(args.yValue) > 100
    ? "#C62828"
    : args.color;
};

<Chart pointRender={handlePointRender}>
  {/* chart children */}
</Chart>
```

Do not rename it to `onPointRender`, attach it to `ChartSeries`, or mutate an invented `args.fill` property.

## Validation checklist

1. Verify the callback against the current Pure React `Chart` API.
2. Use the exact event name and casing.
3. Place root events on `Chart`.
4. Import the documented argument type.
5. Read only properties declared by that type.
6. Keep high-frequency handlers lightweight.
7. Treat `pointRender` as a separate color-returning chart prop.
8. Do not mix EJ2 and Pure React event contracts.
9. Read [`interactivity/events.md`](./interactivity/events.md) before generating less common events.
10. Emit valid, unescaped TSX.
