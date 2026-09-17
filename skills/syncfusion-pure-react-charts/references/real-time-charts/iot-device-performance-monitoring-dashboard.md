# IoT Device Performance Monitoring Dashboard Reference

Use this guidance for telemetry-heavy dashboards that show live device health, utilization, throughput, power, temperature, connectivity, and fault signals. Keep the visible series count controlled, preserve a short and predictable refresh window, and make every metric immediately identifiable by name and unit.

## Core rules

- Use a `DateTime` X-axis with real `Date` values.
- Limit simultaneously updating series to metrics operators must compare directly.
- Use separate charts, panes, or selectable metric groups for incompatible units.
- Keep a bounded rolling window by time, point count, or both.
- Keep chart dimensions stable while data refreshes.
- Disable or shorten animation for frequently updating telemetry.
- Keep axis labels, series names, and tooltips concise and unit aware.
- Distinguish current, stale, disconnected, warning, and critical device states.
- Clean up timers, sockets, subscriptions, and observers.
- Never present generated values as real device telemetry.

## Dashboard architecture

A practical device-monitoring view separates summary state from detailed trends:

```text
Device dashboard
├── Device identity and connection status
├── Current metric summaries
├── Primary live trend chart
├── Optional secondary metric chart
├── Alerts and recent events
└── Time-range and metric controls
```

Do not place every available sensor on one chart. A small metric selector or grouped set of charts is usually easier to scan.

## Telemetry model

Keep timestamps and values unformatted.

```tsx
interface DeviceTelemetry {
  timestamp: Date;
  cpuPercent: number;
  memoryPercent: number;
  temperatureCelsius: number;
  batteryPercent: number;
  signalDbm: number;
}
```

Validate each incoming row before adding it to state:

```tsx
function isFiniteTelemetry(point: DeviceTelemetry): boolean {
  return (
    point.timestamp instanceof Date &&
    Number.isFinite(point.timestamp.getTime()) &&
    Number.isFinite(point.cpuPercent) &&
    Number.isFinite(point.memoryPercent) &&
    Number.isFinite(point.temperatureCelsius) &&
    Number.isFinite(point.batteryPercent) &&
    Number.isFinite(point.signalDbm)
  );
}
```

## Limit active series

Render only the selected metric group rather than hiding dozens of live series visually.

```tsx
const metricGroups = {
  utilization: ["cpuPercent", "memoryPercent"],
  thermal: ["temperatureCelsius"],
  power: ["batteryPercent"],
  connectivity: ["signalDbm"],
} as const;
```

Use a native control so the metric scope is keyboard accessible:

```tsx
<label>
  Metric group
  <select
    value={metricGroup}
    onChange={(event) =>
      setMetricGroup(event.target.value as MetricGroup)
    }
  >
    <option value="utilization">CPU and memory</option>
    <option value="thermal">Temperature</option>
    <option value="power">Battery</option>
    <option value="connectivity">Signal strength</option>
  </select>
</label>
```

## Stable live chart

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartLegend,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
  ChartTitle,
  ChartTooltip,
} from "@syncfusion/react-charts";

interface UtilizationChartProps {
  data: DeviceTelemetry[];
}

export default function UtilizationChart({
  data,
}: UtilizationChartProps) {
  return (
    <div className="device-chart-canvas">
      <Chart
        width="100%"
        height="100%"
        accessibility={{
          ariaLabel: "Device CPU and memory utilization over time",
          role: "img",
          focusable: true,
          tabIndex: 0,
        }}
      >
        <ChartTitle text="Device utilization" />

        <ChartPrimaryXAxis
          valueType="DateTime"
          intervalType="Minutes"
        >
          <ChartAxisTitle text="Time" />
          <ChartAxisLabel
            skeleton="HH:mm"
            edgeLabelPlacement="Shift"
            intersectAction="Trim"
            maxLabelWidth={56}
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

        <ChartLegend
          visible={true}
          position="Bottom"
          align="Center"
        />

        <ChartTooltip
          enable={true}
          shared={true}
          showMarker={true}
          format="${series.name}: ${point.y}%"
        />

        <ChartSeriesCollection>
          <ChartSeries
            dataSource={data}
            xField="timestamp"
            yField="cpuPercent"
            type="Line"
            name="CPU"
            width={2}
            animation={{
              enable: false,
              duration: 0,
              delay: 0,
            }}
          />
          <ChartSeries
            dataSource={data}
            xField="timestamp"
            yField="memoryPercent"
            type="Line"
            name="Memory"
            width={2}
            animation={{
              enable: false,
              duration: 0,
              delay: 0,
            }}
          />
        </ChartSeriesCollection>
      </Chart>
    </div>
  );
}
```

```css
.device-chart-canvas {
  width: 100%;
  height: 360px;
  min-width: 0;
}

@media (max-width: 640px) {
  .device-chart-canvas {
    height: 320px;
  }
}
```

The chart accepts string dimensions, while series support bound data, explicit fields, names, line styling, and animation configuration. turn81view529

## Bounded refresh window

Use a time window when sample frequency can vary:

```tsx
const refreshWindowMs = 10 * 60 * 1000;

setTelemetry((current) => {
  const cutoff = nextPoint.timestamp.getTime() - refreshWindowMs;

  return [...current, nextPoint].filter(
    (point) => point.timestamp.getTime() >= cutoff,
  );
});
```

Add a point cap as a defensive bound:

```tsx
const maximumPoints = 600;

setTelemetry((current) => {
  const cutoff = nextPoint.timestamp.getTime() - refreshWindowMs;

  return [...current, nextPoint]
    .filter((point) => point.timestamp.getTime() >= cutoff)
    .slice(-maximumPoints);
});
```

Document the visible period near the chart, such as “Last 10 minutes.”

## Batch high-frequency updates

Do not commit React state for every raw sensor packet when devices publish faster than the display needs.

```tsx
useEffect(() => {
  const pending: DeviceTelemetry[] = [];

  const unsubscribe = telemetryStream.subscribe((point) => {
    if (isFiniteTelemetry(point)) {
      pending.push(point);
    }
  });

  const timer = window.setInterval(() => {
    if (pending.length === 0) {
      return;
    }

    const batch = pending.splice(0, pending.length);

    setTelemetry((current) =>
      [...current, ...batch].slice(-maximumPoints),
    );
  }, 1000);

  return () => {
    window.clearInterval(timer);
    unsubscribe();
  };
}, [telemetryStream]);
```

Choose a batch interval that preserves operationally significant spikes and state changes.

## Units and metric separation

Recommended grouping:

- CPU and memory: percentage axis
- battery: percentage axis, often separate from utilization
- temperature: Celsius or Fahrenheit axis
- signal strength: dBm axis
- throughput: bytes per second or bits per second
- latency: milliseconds
- faults: interval count or event timeline

Do not plot temperature, battery, signal strength, and CPU against one unlabeled numeric axis.

## Concise labels

Use short series names and concise axis titles:

- `CPU`, not `Current central processing unit utilization percentage`
- `Memory`, not `Total active memory utilization percentage`
- `Temperature (°C)`
- `Signal (dBm)`
- `Latency (ms)`

The axis-label API supports formatting, trimming, wrapping, rotation, and edge handling for constrained layouts. 

```tsx
<ChartAxisLabel
  skeleton="HH:mm"
  intersectAction="Trim"
  edgeLabelPlacement="Shift"
  maxLabelWidth={56}
/>
```

## Concise tooltips

Tooltips support shared values, markers, format strings, templates, and nearest-point behavior. 

```tsx
<ChartTooltip
  enable={true}
  shared={true}
  showMarker={true}
  format="${series.name}: ${point.y}"
/>
```

For different units, use series-level formats:

```tsx
<ChartSeries
  name="Temperature"
  tooltipFormat="${series.name}: ${point.y} °C"
/>
```

Keep templates small and avoid expensive work during pointer movement.

## Device freshness and connectivity

Track freshness separately from chart data:

```tsx
const staleAfterMs = 30_000;
const isStale = Date.now() - lastReceivedAt.getTime() > staleAfterMs;
```

Show a stable status summary:

```tsx
<p role="status" aria-live="polite">
  {connectionState === "disconnected"
    ? "Device disconnected"
    : isStale
      ? "Telemetry is stale"
      : `Live. Last sample ${lastSampleLabel}`}
</p>
```

Do not announce every sample to assistive technology. Announce meaningful connection or threshold changes.

## Warning and critical states

Use thresholds appropriate to the device specification and deployment environment. Pair color with text or patterns.

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
      text="Warning"
      color="rgba(247, 99, 12, 0.14)"
      visible={true}
    />
    <ChartStripline
      start={90}
      size={10}
      text="Critical"
      color="rgba(164, 38, 44, 0.16)"
      visible={true}
    />
  </ChartStriplineCollection>
</ChartPrimaryYAxis>
```

Do not reuse percentage thresholds for temperature, battery, signal, or latency without a device-specific basis.

## History exploration

Use X-axis zoom when operators need to inspect recent events. Zoom settings support selection, wheel, pinch, panning, scrollbars, and a toolbar. 

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

If the operator zooms into history, pause live-follow behavior or show a clear “Return to live” control.

## Layout stability

- Reserve fixed card and chart heights.
- Keep loading, empty, error, and disconnected states inside the same footprint.
- Avoid adding and removing legends as values change.
- Keep metric controls outside the plot area.
- Prevent long device names from changing chart height.
- Use `min-width: 0` inside Grid and Flex layouts.
- Avoid changing series count on every packet.

## Performance guidance

- Render only the active metric group.
- Bound every device history.
- Batch high-frequency updates.
- Avoid markers and data labels on dense live lines.
- Disable unnecessary animation.
- Keep high-frequency callbacks lightweight.
- Aggregate by a domain-appropriate method, preserving peaks when required.
- Virtualize long device lists outside the chart.
- Profile with the real device count, sample rate, and retention window.

## Accessibility

The root chart supports accessibility settings, focus styling, and documented interaction events. 

- Give each chart a metric- and device-specific accessible name.
- Provide current value, status, freshness, and threshold summaries in text.
- Make device and metric selectors keyboard accessible.
- Keep focus order stable as telemetry updates.
- Do not rely only on hover, motion, or color.
- Provide tabular or downloadable history when precise values are required.
- Avoid live-region announcements for every sample.

## Common errors

### Too many active series

Do not render every sensor in one updating chart. Group related metrics and let operators switch views.

### Unbounded telemetry

Always enforce a rolling time window or maximum point count.

### Layout jumping during refresh

Reserve stable dimensions for all data and connection states.

### Ambiguous units

Include the unit in the axis title, tooltip, summary, or series name.

### One axis for incompatible metrics

Separate percentages, temperature, signal, throughput, and latency.

### Random demo data presented as live telemetry

Clearly label synthetic samples and never imply that generated values came from devices.

## Validation checklist

Before returning an IoT monitoring implementation:

1. Use real `Date` values with a `DateTime` axis.
2. Define the sample interval and visible refresh window.
3. Bound history by time, count, or both.
4. Validate timestamps and finite metric values.
5. Limit simultaneously updating series.
6. Group only metrics with compatible units and comparison goals.
7. Keep metric labels and units concise and explicit.
8. Reserve stable chart and card dimensions.
9. Batch high-frequency updates where appropriate.
10. Clean up timers, sockets, streams, listeners, and observers.
11. Avoid dense markers, labels, and animation.
12. Keep tooltips concise and unit aware.
13. Preserve meaningful spikes during aggregation.
14. Show live, stale, disconnected, loading, empty, and error states.
15. Use device-specific warning and critical thresholds.
16. Pause or clearly control live-follow during historical zoom.
17. Keep controls keyboard accessible and focus order stable.
18. Do not rely only on color, hover, or motion.
19. Profile with production-like device counts and rates.
20. Do not fabricate operational telemetry.
21. Do not mix EJ2 and Pure React APIs.
22. Ensure every imported symbol is used.
23. Emit valid, unescaped TSX.
