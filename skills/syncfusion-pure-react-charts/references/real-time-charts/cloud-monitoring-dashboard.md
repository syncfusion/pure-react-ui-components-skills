# Cloud Monitoring Dashboard Reference

Use this guidance for time-based infrastructure monitoring, operational trends, service health, and bounded live telemetry in Pure React Chart. Keep the update path lightweight, preserve only the history needed for the monitoring task, and make current state readable without overwhelming the plot.

## Core rules

- Use a `DateTime` X-axis with real `Date` values for time-series telemetry.
- Keep live history bounded by a defined time window or maximum point count.
- Append data immutably and avoid rebuilding unrelated chart configuration on every sample.
- Keep labels, tooltips, markers, and animations restrained while data is moving.
- Use consistent units and clearly named series.
- Separate metrics with incompatible units into named axes, panes, or different charts.
- Make stale, disconnected, loading, and empty states explicit.
- Clean up timers, streams, sockets, and subscriptions when the component unmounts.
- Profile the real sampling rate and target devices before choosing aggregation rules.

## Recommended chart architecture

Use line or area series for continuous operational signals and columns for interval counts.

```text
Chart
├── ChartTitle
├── ChartPrimaryXAxis (DateTime)
├── ChartPrimaryYAxis (metric unit)
├── ChartTooltip
├── Optional ChartCrosshair
├── Optional ChartZoomSettings
└── ChartSeriesCollection
    ├── ChartSeries (CPU)
    ├── ChartSeries (memory)
    └── ChartSeries (requests or errors)
```

Do not force percentages, durations, byte counts, and request rates onto one unlabeled scale.

## Time-series data model

Keep timestamps and values typed and unformatted.

```tsx
interface MetricPoint {
  timestamp: Date;
  value: number;
}
```

For multi-metric rows:

```tsx
interface HealthPoint {
  timestamp: Date;
  cpuPercent: number;
  memoryPercent: number;
  requestsPerSecond: number;
}
```

Store numbers as numbers. Apply locale and unit formatting only to labels, tooltips, summaries, and tables.

## Minimal monitoring chart

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
  ChartTitle,
  ChartTooltip,
} from "@syncfusion/react-charts";

const data: MetricPoint[] = [
  { timestamp: new Date("2026-09-07T12:00:00Z"), value: 42 },
  { timestamp: new Date("2026-09-07T12:01:00Z"), value: 48 },
  { timestamp: new Date("2026-09-07T12:02:00Z"), value: 45 },
];

export default function CpuChart() {
  return (
    <Chart
      width="100%"
      height="360px"
      accessibility={{
        ariaLabel: "CPU utilization over time",
        role: "img",
        focusable: true,
        tabIndex: 0,
      }}
    >
      <ChartTitle text="CPU utilization" />

      <ChartPrimaryXAxis
        valueType="DateTime"
        intervalType="Minutes"
      >
        <ChartAxisTitle text="Time" />
        <ChartAxisLabel
          skeleton="HH:mm"
          edgeLabelPlacement="Shift"
          intersectAction="Trim"
        />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis
        valueType="Double"
        minimum={0}
        maximum={100}
      >
        <ChartAxisTitle text="Utilization (%)" />
        <ChartAxisLabel format="{value}%" />
      </ChartPrimaryYAxis>

      <ChartTooltip
        enable={true}
        format="${series.name}: ${point.y}%"
        showMarker={true}
      />

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="timestamp"
          yField="value"
          type="Line"
          name="CPU"
          width={2}
          animation={{
            enable: false,
            duration: 0,
            delay: 0,
          }}
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

The Pure React series API supports local arrays through `dataSource`, explicit field mappings, series animation settings, per-series tooltip control, and line styling. 

## Bounded live updates

Bound history by point count when samples arrive at a stable interval.

```tsx
const maximumPoints = 300;

setData((current) =>
  [...current, nextPoint].slice(-maximumPoints),
);
```

Bound history by time when the sampling interval can vary.

```tsx
const windowDurationMs = 15 * 60 * 1000;

setData((current) => {
  const cutoff = nextPoint.timestamp.getTime() - windowDurationMs;

  return [...current, nextPoint].filter(
    (point) => point.timestamp.getTime() >= cutoff,
  );
});
```

Choose one policy based on the monitoring requirement. A time window usually communicates operational recency more clearly than an unexplained point limit.

## Timer-based sample pattern

```tsx
import { useEffect, useState } from "react";

const maximumPoints = 120;

export default function LiveMetric() {
  const [data, setData] = useState<MetricPoint[]>([]);

  useEffect(() => {
    const timer = window.setInterval(() => {
      const nextPoint: MetricPoint = {
        timestamp: new Date(),
        value: readCurrentMetric(),
      };

      setData((current) =>
        [...current, nextPoint].slice(-maximumPoints),
      );
    }, 5000);

    return () => window.clearInterval(timer);
  }, []);

  return <MonitoringChart data={data} />;
}
```

Do not use randomly generated values in production monitoring. Bind real telemetry or clearly label synthetic demonstration data.

## Stream and WebSocket cleanup

```tsx
useEffect(() => {
  const socket = new WebSocket(streamUrl);

  const handleMessage = (event: MessageEvent<string>): void => {
    const nextPoint = parseMetricMessage(event.data);

    if (!nextPoint) {
      return;
    }

    setData((current) =>
      [...current, nextPoint].slice(-maximumPoints),
    );
  };

  socket.addEventListener("message", handleMessage);

  return () => {
    socket.removeEventListener("message", handleMessage);
    socket.close();
  };
}, [streamUrl]);
```

Validate incoming timestamps and values before adding them to the chart. Reject non-finite values and malformed dates.

## Sampling and aggregation

Do not discard every nth point without considering spikes. Choose an aggregation that preserves the signal needed for operations.

Useful interval summaries include:

- average for typical utilization
- maximum for short saturation events
- minimum for availability floors
- sum for interval request or error counts
- percentile for latency distributions
- first and last for state transitions

When spikes or outages matter, preserve maximum and minimum values within each display bucket instead of showing the average alone.

## Concise axis labels

The axis-label API supports formatting, custom formatting, maximum width, trimming, wrapping, rotation, and edge-label placement. 

```tsx
<ChartPrimaryXAxis
  valueType="DateTime"
  intervalType="Minutes"
>
  <ChartAxisLabel
    skeleton="HH:mm"
    edgeLabelPlacement="Shift"
    intersectAction="Trim"
    maxLabelWidth={56}
  />
</ChartPrimaryXAxis>
```

Avoid full date-time strings on every tick. Put the date in a title, subtitle, period control, or tooltip when the displayed window is within one day.

## Concise tooltips

The tooltip API supports format strings, shared mode, nearest-point behavior, markers, formatters, and templates. 

```tsx
<ChartTooltip
  enable={true}
  shared={true}
  showMarker={true}
  format="${series.name}: ${point.y}"
/>
```

For mixed units, use series-level `tooltipFormat` or a template, and include units with each value.

```tsx
<ChartSeries
  name="Latency"
  tooltipFormat="${series.name}: ${point.y} ms"
/>
```

Keep custom tooltip templates lightweight because pointer movement may trigger frequent updates.

## Markers and labels under motion

Do not render a marker or data label at every point in a dense live series. Prefer:

- no markers for continuous dense lines
- tooltips for per-point detail
- a last-value label for the current reading
- annotations for incidents or deployments
- visible labels only for exceptional values

Disable or shorten animation for frequently updating telemetry. Series animation defaults to enabled with duration and delay settings, so live dashboards should make an intentional choice. 

## Multiple metrics

Use a shared percentage axis only when all metrics use the same percentage definition.

```tsx
<ChartSeriesCollection>
  <ChartSeries
    dataSource={data}
    xField="timestamp"
    yField="cpuPercent"
    type="Line"
    name="CPU"
  />
  <ChartSeries
    dataSource={data}
    xField="timestamp"
    yField="memoryPercent"
    type="Line"
    name="Memory"
  />
</ChartSeriesCollection>
```

Put latency, requests per second, bytes, and percentages on clearly named axes or separate charts. Avoid adding a secondary axis solely to make unrelated values visually overlap.

## Thresholds and incidents

Use striplines for stable warning or critical thresholds and annotations for discrete operational events.

```tsx
<ChartPrimaryYAxis
  valueType="Double"
  minimum={0}
  maximum={100}
>
  <ChartStriplineCollection>
    <ChartStripline
      start={80}
      size={10}
      color="rgba(247, 99, 12, 0.14)"
      text="Warning"
      visible={true}
    />
    <ChartStripline
      start={90}
      size={10}
      color="rgba(164, 38, 44, 0.16)"
      text="Critical"
      visible={true}
    />
  </ChartStriplineCollection>
</ChartPrimaryYAxis>
```

Do not use only color to communicate severity. Include text, patterns, status summaries, or alert lists.

## Zooming and history exploration

Use X-axis zoom for longer telemetry windows. Zoom settings support selection, mouse wheel, pinch, panning, scrollbars, and a toolbar, and these interactions are disabled by default. 

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

When a user zooms into historical data, decide whether the live window should continue advancing, pause visually, or offer a clear return-to-live action. Do not silently pull the viewport away from the investigated interval.

## Service state and freshness

Show data freshness outside the chart.

```tsx
<p role="status" aria-live="polite">
  {connectionState === "connected"
    ? `Last updated ${lastUpdatedLabel}`
    : "Live data connection is unavailable"}
</p>
```

Distinguish:

- loading: no first response yet
- live: samples are arriving within the expected interval
- stale: a previous value exists but is older than the freshness threshold
- disconnected: the source connection is unavailable
- empty: the source returned no samples
- error: the request or parsing operation failed

Do not draw a flat line through a disconnected period unless the source explicitly reports unchanged values.

## Accessibility

The root Chart API supports accessibility configuration and focus styling, while documented interaction events belong on the root chart. 

- Provide a concise chart accessibility label.
- Add a text summary of current value, trend, threshold state, and freshness.
- Provide a table or downloadable data for precise historical values when needed.
- Do not rely only on rapidly changing tooltips.
- Keep status announcements useful and avoid announcing every incoming sample.
- Use non-color cues for warning, critical, stale, and disconnected states.
- Make time-range, pause, resume, and reset controls keyboard operable.

## Performance guidance

- Bound every live series.
- Avoid expensive work in `onMouseMove`.
- Prefer `onZoomEnd` when only the final zoom state matters.
- Memoize expensive aggregation, not trivial constants.
- Keep tooltip and label templates small.
- Reduce markers, labels, shadows, and animation before sacrificing data meaning.
- Batch or aggregate high-frequency telemetry before it reaches React state.
- Measure commit time, memory growth, and pointer responsiveness with production-like data.

## Common errors

### Unbounded history

Incorrect:

```tsx
setData((current) => [...current, nextPoint]);
```

Correct:

```tsx
setData((current) =>
  [...current, nextPoint].slice(-maximumPoints),
);
```

### Formatted time strings on a Category axis

Use real `Date` values and a `DateTime` axis when spacing and zooming should reflect time.

### Animation on every sample

Disable animation or verify that the selected duration does not cause continuously overlapping transitions.

### Too many live labels

Do not enable data labels and markers for every point in a dense moving window.

### Mixing incompatible units

Separate percentages, latency, throughput, and byte counts into named axes, panes, or charts.

### Random values presented as monitoring data

Synthetic values are acceptable only for a labeled demo. Never present them as retrieved operational telemetry.

### Reconnecting without cleanup

Avoid creating duplicate sockets, timers, or event listeners when dependencies change.

## Validation checklist

Before returning a cloud-monitoring implementation:

1. Use real `Date` values with a `DateTime` X-axis.
2. Define the displayed time window and expected sampling interval.
3. Bound history by time, point count, or both.
4. Validate incoming timestamps and finite numeric values.
5. Clean up intervals, sockets, listeners, and requests.
6. Keep series names and units explicit.
7. Separate incompatible units.
8. Keep axis labels short and use edge-label handling.
9. Keep tooltips concise and include units.
10. Avoid dense markers and data labels.
11. Make an intentional animation choice for live updates.
12. Preserve spikes with a suitable aggregation strategy.
13. Show warning and critical thresholds with text and non-color cues.
14. Distinguish loading, live, stale, disconnected, empty, and error states.
15. Decide how zoom interacts with live-follow behavior.
16. Keep high-frequency handlers lightweight.
17. Provide current-state summaries and accessible historical detail.
18. Do not announce every incoming sample to assistive technology.
19. Profile with production-like rates and history sizes.
20. Do not fabricate operational data.
21. Do not mix EJ2 and Pure React APIs.
22. Ensure every imported symbol is used.
23. Emit valid, unescaped TSX.
