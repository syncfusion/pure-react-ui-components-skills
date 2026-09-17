# Troubleshooting Reference

Use this diagnostic guide for common setup, data-binding, rendering, styling, interactivity, and performance issues in Pure React Chart. Start from a minimal valid chart, verify one layer at a time, and use only APIs exported by `@syncfusion/react-charts`.

## Table of contents

1. [Fast diagnostic sequence](#fast-diagnostic-sequence)
2. [Setup issues](#setup-issues)
3. [Data and binding issues](#data-and-binding-issues)
4. [Rendering issues](#rendering-issues)
5. [Performance issues](#performance-issues)
6. [Styling and appearance](#styling-and-appearance)
7. [Interactivity issues](#interactivity-issues)
8. [Debugging strategies](#debugging-strategies)
9. [Validation checklist](#validation-checklist)

## Fast diagnostic sequence

When a chart fails, check in this order:

1. Confirm `@syncfusion/react-charts` is installed.
2. Confirm one compatible theme stylesheet is loaded.
3. Reduce the implementation to `Chart`, `ChartSeriesCollection`, and one `ChartSeries`.
4. Verify `dataSource` is a non-empty array.
5. Verify `xField` and `yField` exactly match real object keys.
6. Match the X-axis `valueType` to the bound values.
7. Give the chart a resolved width and height.
8. Check TypeScript, build, browser-console, and network errors.
9. Add titles, labels, legends, tools, and interactions back one at a time.
10. Compare every prop with the installed Pure React API, not EJ2 examples.

## Setup issues

### Chart does not render

The current getting-started flow installs `@syncfusion/react-charts`, imports the base theme stylesheet, and renders series inside `ChartSeriesCollection`.

```bash
npm install @syncfusion/react-charts
```

```css
@import "@syncfusion/react-base/styles/material.css";
```

```tsx
import {
  Chart,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { x: "Jan", y: 100 },
  { x: "Feb", y: 120 },
];

export default function App() {
  return (
    <Chart width="100%" height="420px">
      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="x"
          yField="y"
          type="Column"
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

If this minimum does not render:

- inspect the first compile or console error
- verify the package version and import path
- verify the stylesheet resolves through the bundler
- check that the parent is not hidden or zero-sized
- remove unsupported props copied from another package

### Missing module

If the build reports that `@syncfusion/react-charts` cannot be found, install it in the same workspace that builds the application and verify it appears in `package.json` and the lockfile.

Install `@syncfusion/react-base` separately only when the application imports its utilities directly, such as `Provider` for RTL.

### Mixed chart packages

Do not combine Pure React components with EJ2 components.

Incorrect Pure React usage includes:

```tsx
<ChartComponent>
  <SeriesCollectionDirective>
    <SeriesDirective xName="x" yName="y" />
  </SeriesCollectionDirective>
</ChartComponent>
```

Pure React uses:

```tsx
<Chart>
  <ChartSeriesCollection>
    <ChartSeries xField="x" yField="y" />
  </ChartSeriesCollection>
</Chart>
```

### License messages

Treat licensing warnings as product and deployment configuration, not as a chart-rendering workaround. Follow the current Syncfusion licensing documentation for the installed package and organization. Do not recommend ignoring a warning in production or publish placeholder keys as a fix.

## Data and binding issues

### No points appear

Inspect the actual data rather than assuming its shape.

```tsx
console.log("row count", data?.length);
console.log("first row", data?.[0]);
console.log("keys", data?.[0] ? Object.keys(data[0]) : []);
```

Then map real fields:

```tsx
<ChartSeries
  dataSource={data}
  xField="month"
  yField="sales"
  type="Column"
/>
```

Check that:

- `dataSource` is defined and non-empty
- `xField` exists on every relevant row
- `yField` contains finite numbers for numeric series
- values are not formatted strings such as `"₹1,200"`
- the series type has all required fields

### Wrong axis type

Use:

- `Category` for text categories
- `DateTime` for real `Date` values
- `Double` for continuous numbers
- `Logarithmic` for valid positive logarithmic values

```tsx
<ChartPrimaryXAxis valueType="Category" />
```

A DateTime axis should receive real dates rather than preformatted category labels.

### State updates do not render

Do not mutate the existing array or row object.

Incorrect:

```tsx
data.push(newItem);
setData(data);
```

Correct:

```tsx
setData((current) => [...current, newItem]);
```

Correct row update:

```tsx
setData((current) =>
  current.map((item, index) =>
    index === 0 ? { ...item, sales: 200 } : item,
  ),
);
```

### Empty points

`null` and `undefined` Y values are treated as empty points. Configure the behavior deliberately.

```tsx
<ChartSeries
  dataSource={data}
  xField="x"
  yField="y"
  type="Line"
  emptyPointSettings={{
    mode: "Gap",
    fill: "#A0A0A0",
    border: {
      color: "#707070",
      width: 1,
    },
  }}
/>
```

Supported empty-point modes include `Gap`, `Zero`, `Drop`, and `Average`. Do not silently filter missing observations if the gap itself is meaningful.

### Remote data stays empty

Track loading, success, empty, and failure separately.

```tsx
const [data, setData] = useState<DataPoint[]>([]);
const [status, setStatus] =
  useState<"loading" | "ready" | "empty" | "error">("loading");

useEffect(() => {
  const controller = new AbortController();

  async function load(): Promise<void> {
    try {
      const response = await fetch("/api/data", {
        signal: controller.signal,
      });

      if (!response.ok) {
        throw new Error(`Request failed: ${response.status}`);
      }

      const result: DataPoint[] = await response.json();
      setData(result);
      setStatus(result.length > 0 ? "ready" : "empty");
    } catch (error) {
      if (!controller.signal.aborted) {
        console.error(error);
        setStatus("error");
      }
    }
  }

  void load();
  return () => controller.abort();
}, []);
```

Do not render fabricated fallback data unless synthetic data is explicitly required and clearly labeled.

## Rendering issues

### Chart has zero height

A chart using `height="100%"` needs a parent with a resolved height.

```tsx
<div className="chart-canvas">
  <Chart width="100%" height="100%">
    {/* chart children */}
  </Chart>
</div>
```

```css
.chart-canvas {
  width: 100%;
  height: 420px;
  min-width: 0;
}
```

Also check hidden tabs, collapsed panels, dialogs, and split panes. If a chart first renders in a zero-sized container, retest after the container becomes visible.

### Axis labels overlap

Axis-label styling belongs in `ChartAxisLabel`.

```tsx
<ChartPrimaryXAxis valueType="Category">
  <ChartAxisLabel
    intersectAction="Wrap"
    enableWrap={true}
    maxLabelWidth={90}
    edgeLabelPlacement="Shift"
  />
</ChartPrimaryXAxis>
```

Other documented strategies include `Trim`, `Hide`, `MultipleRows`, `Rotate45`, and `Rotate90`.

Do not use EJ2-style props such as `labelStyle`, `labelFormat`, or `labelRotationAngle` on the axis.

### Legend is missing or crowded

Give each series a meaningful `name`, keep `ChartLegend` directly inside `Chart`, and verify `visible`.

```tsx
<ChartLegend
  visible={true}
  position="Bottom"
  align="Center"
  enablePages={true}
  maxLabelWidth={160}
/>
```

```tsx
<ChartSeries
  name="Actual"
  dataSource={data}
  xField="month"
  yField="actual"
  type="Column"
/>
```

If the legend consumes too much plot space, shorten names, move it to the bottom, enable pages, increase chart height, or reconsider how many series belong in one chart.

### Colors do not apply

Check color ownership:

- root `palettes` assigns series colors sequentially
- series `fill` sets a series color
- series `colorField` maps row-level colors
- root `pointRender` computes and returns point colors

```tsx
<Chart palettes={["#005A9C", "#107C10"]}>
  {/* chart children */}
</Chart>
```

```tsx
<ChartSeries fill="#005A9C" colorField="color" />
```

Do not use `pointColorMapping`, `onPointRender`, or mutate an invented `args.fill` in Pure React samples.

## Performance issues

### Slow initial render

Measure before applying a universal threshold. Common contributors include:

- large or frequently recreated data arrays
- many series
- dense markers and data labels
- complex tooltip or label templates
- expensive transformations during render
- animation during rapid updates

Memoize expensive deterministic transforms:

```tsx
const processedData = useMemo(
  () => transformData(data),
  [data],
);
```

Disable animation when profiling or reduced-motion requirements justify it:

```tsx
<ChartSeries
  animation={{
    enable: false,
    duration: 0,
    delay: 0,
  }}
/>
```

Do not downsample by taking every tenth row unless that method is analytically appropriate. Aggregation must preserve the intended meaning of the data.

### Interaction lag

Keep `onMouseMove` and zoom handlers lightweight. `onZoomEnd` is preferable when the application only needs the final zoom state.

```tsx
const handleZoomEnd = useCallback(
  (args: ZoomEndEvent): void => {
    queueAnalyticsUpdate(args);
  },
  [],
);

<Chart onZoomEnd={handleZoomEnd}>
  {/* chart children */}
</Chart>
```

Avoid synchronous expensive calculations or state updates for every pointer movement.

### Growing memory use

Bound live-data windows and clean up subscriptions.

```tsx
setData((current) =>
  [...current, newPoint].slice(-maximumPoints),
);
```

```tsx
useEffect(() => {
  stream.addEventListener("message", handleMessage);

  return () => {
    stream.removeEventListener("message", handleMessage);
    stream.close();
  };
}, []);
```

Also inspect detached DOM nodes, timers, observers, and repeated listener registration in browser performance tools.

## Styling and appearance

### Title is invisible

Use a non-empty title, CSS-style font-size string, and sufficient contrast.

```tsx
<ChartTitle
  text="Monthly sales"
  color="#242424"
  fontSize="18px"
  fontWeight="Bold"
/>
```

### Grid lines are invisible

Keep the grid-line component inside its axis and use a visible width and color.

```tsx
<ChartPrimaryYAxis valueType="Double">
  <ChartMajorGridLines
    visible={true}
    width={1}
    color="#D9D9D9"
    dashArray=""
  />
</ChartPrimaryYAxis>
```

### Theme looks inconsistent

Use one supported chart theme and one compatible stylesheet. Do not mix light and dark resources or assume EJ2 theme names are supported by the installed Pure React version.

### CSS overrides do nothing

Prefer documented props and wrapper-level CSS. Do not depend on EJ2 selectors such as `.e-chart`, `.e-series-group`, or `.e-data-label` for Pure React internals.

## Interactivity issues

### Tooltip does not appear

Place `ChartTooltip` directly inside `Chart` and enable it.

```tsx
<Chart>
  <ChartTooltip enable={true} />

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="x"
      yField="y"
      type="Line"
      enableTooltip={true}
    />
  </ChartSeriesCollection>
</Chart>
```

Check that the series has rendered points and that `enableTooltip` was not disabled for that series.

For event diagnostics, use the documented `onMouseMove`, not `onChartMouseMove`.

### Zoom does not work

Enable the required interaction explicitly.

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

Verify that `ChartZoomSettings` is a direct child of `Chart`, data spans a meaningful range, and no overlay is intercepting pointer input.

### Selection does not work

The selection default is `None`. Set a valid mode.

```tsx
<ChartSelection
  mode="Point"
  allowMultiSelection={true}
  selectedDataIndexes={[
    { seriesIndex: 0, pointIndex: 1 },
  ]}
  pattern="Crosshatch"
/>
```

The root Chart API does not currently document `onSelectionChanged`. Use documented events such as `onPointClick` for application logic, or manage `selectedDataIndexes` through external state when appropriate.

### Legend click does not toggle a series

`ChartLegend.toggleVisibility` defaults to `true`, but can be set explicitly.

```tsx
<ChartLegend
  visible={true}
  toggleVisibility={true}
/>
```

If `onLegendClick` is used, do not set `args.cancel = true` unless the default toggle must be prevented.

## Debugging strategies

### Isolate the minimum reproduction

Create a small local array and one series. Remove:

- remote data
- templates
- multiple axes
- annotations
- selection and highlight
- zoom and crosshair
- custom formatters
- wrapper CSS overrides

Add each feature back individually until the failure returns.

### Inspect public structure, not private class names

Use React DevTools to inspect props and state. In browser DevTools, inspect computed size, visibility, clipping, and pointer overlays. Do not build diagnostics around undocumented internal selectors.

### Log useful facts

```tsx
useEffect(() => {
  console.debug("chart diagnostics", {
    rows: data.length,
    firstRow: data[0],
    keys: data[0] ? Object.keys(data[0]) : [],
    containerWidth: containerRef.current?.clientWidth,
    containerHeight: containerRef.current?.clientHeight,
  });
}, [data]);
```

Do not log credentials, private records, or full production datasets.

### Error boundary

```tsx
class ChartErrorBoundary extends React.Component<
  React.PropsWithChildren,
  { failed: boolean }
> {
  state = { failed: false };

  static getDerivedStateFromError() {
    return { failed: true };
  }

  componentDidCatch(error: Error, info: React.ErrorInfo) {
    console.error("Chart render failed", error, info);
  }

  render() {
    if (this.state.failed) {
      return <p role="alert">The chart could not be displayed.</p>;
    }

    return this.props.children;
  }
}
```

An error boundary catches render failures in its descendant tree, but not all asynchronous request or event-handler errors.

## Getting help

When escalating an issue, provide:

- exact package versions
- framework, bundler, browser, and operating system
- the first console or build error
- a minimal reproducible sample
- sanitized representative data
- expected and actual behavior
- whether the issue occurs in both a minimal local-data chart and the full application

Check the current Pure React API and examples rather than similarly named EJ2 documentation.

## Validation checklist

Before returning troubleshooting guidance:

1. Start from a minimal valid Pure React chart.
2. Verify installation and one compatible theme stylesheet.
3. Keep every series inside `ChartSeriesCollection`.
4. Verify data shape and exact field mappings.
5. Match axis type to data values.
6. Keep plotted numeric values numeric.
7. Give percentage-height charts a resolved parent height.
8. Use `ChartAxisLabel` for label formatting and intersection handling.
9. Use immutable React state updates.
10. Handle empty points deliberately.
11. Track remote loading, empty, and error states separately.
12. Use documented names such as `onMouseMove` and `onPointClick`.
13. Do not use undocumented `onSelectionChanged`.
14. Keep high-frequency handlers lightweight.
15. Profile before downsampling or disabling features.
16. Clean up streams, listeners, timers, and requests.
17. Prefer documented props over internal CSS selectors.
18. Test tooltip, zoom, selection, legend, keyboard, RTL, print, and export independently.
19. Do not advise ignoring licensing requirements.
20. Do not mix EJ2, Cartesian, and pie APIs.
21. Ensure every imported symbol is used.
22. Emit valid, unescaped TSX.
