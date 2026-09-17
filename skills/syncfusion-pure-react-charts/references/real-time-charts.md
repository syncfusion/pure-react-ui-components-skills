# Real-Time Charts Reference

Use this umbrella reference for live dashboards and continuously updating Pure React charts. Route detailed implementation work to the matching page in `references/real-time-charts/`, while keeping the shared architecture, update, accessibility, and validation rules below.

## Includes

1. [Cloud Monitoring Dashboard](./real-time-charts/cloud-monitoring-dashboard.md)
2. [IoT Device Performance Monitoring Dashboard](./real-time-charts/iot-device-performance-monitoring-dashboard.md)

## Core rules

- Use real `Date` values with a `DateTime` X-axis for time-based telemetry.
- Keep every live series bounded by a time window, point count, or both.
- Limit simultaneously updating series to metrics users must compare directly.
- Keep the chart footprint stable across loading, live, stale, disconnected, empty, and error states.
- Use concise series names, axis titles, labels, and tooltips with explicit units.
- Disable or shorten animation when frequent updates cause overlapping motion.
- Batch high-frequency samples before committing them to React state when appropriate.
- Clean up intervals, sockets, streams, listeners, observers, and pending requests.
- Preserve operationally meaningful peaks when aggregating.
- Never present synthetic samples as retrieved operational data.

## Routing guidance

Use the cloud-monitoring page for:

- service and infrastructure health
- CPU, memory, latency, throughput, and error-rate trends
- application or platform observability
- deployment and incident annotations
- warning and critical operating bands
- historical inspection with live-follow behavior

Use the IoT page for:

- device and sensor telemetry
- temperature, power, battery, signal, and utilization metrics
- device freshness and connectivity states
- selectable metric groups
- large device fleets and high-frequency packet streams
- device-specific operating thresholds

Use both pages when the dashboard combines fleet telemetry with cloud-side infrastructure or service health.

## Shared architecture

```text
Real-time dashboard
├── Scope controls
│   ├── service, device, or metric selector
│   └── time-range control
├── Current-state summary
│   ├── latest value
│   ├── status and severity
│   └── last-updated time
├── Live chart
│   ├── DateTime X-axis
│   ├── one compatible metric group
│   ├── concise tooltip
│   └── optional thresholds and zoom
├── Alerts or event timeline
└── Accessible historical detail
```

Keep application controls and status summaries outside the plot area so chart density remains predictable.

## Minimal time-series structure

```tsx
<Chart width="100%" height="360px">
  <ChartPrimaryXAxis
    valueType="DateTime"
    intervalType="Minutes"
  >
    <ChartAxisLabel
      skeleton="HH:mm"
      edgeLabelPlacement="Shift"
      intersectAction="Trim"
    />
  </ChartPrimaryXAxis>

  <ChartPrimaryYAxis valueType="Double">
    <ChartAxisTitle text="Metric unit" />
    <ChartAxisLabel format="{value}" />
  </ChartPrimaryYAxis>

  <ChartTooltip
    enable={true}
    shared={true}
    showMarker={true}
    format="${series.name}: ${point.y}"
  />

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="timestamp"
      yField="value"
      type="Line"
      name="Metric"
      width={2}
      animation={{
        enable: false,
        duration: 0,
        delay: 0,
      }}
    />
  </ChartSeriesCollection>
</Chart>
```

The Pure React Chart API supports responsive string dimensions and root accessibility configuration. Series support bound data, field mapping, names, tooltips, styling, and animation configuration. turn82view487

## Bounded update pattern

Bound by time and count:

```tsx
const windowDurationMs = 15 * 60 * 1000;
const maximumPoints = 600;

setData((current) => {
  const cutoff = nextPoint.timestamp.getTime() - windowDurationMs;

  return [...current, nextPoint]
    .filter((point) => point.timestamp.getTime() >= cutoff)
    .slice(-maximumPoints);
});
```

Use a time window to communicate recency and a point cap as a defensive memory bound. The appropriate values depend on sampling rate, monitor size, metric volatility, and operational needs.

## Data validation

Validate telemetry before it reaches the chart.

```tsx
interface LivePoint {
  timestamp: Date;
  value: number;
}

function isValidLivePoint(point: LivePoint): boolean {
  return (
    point.timestamp instanceof Date &&
    Number.isFinite(point.timestamp.getTime()) &&
    Number.isFinite(point.value)
  );
}
```

Treat missing samples as missing. Do not join a disconnected period with fabricated values or carry the last observation forward unless the source semantics explicitly require it.

## Metric grouping

Combine series only when the metrics share:

- a meaningful comparison task
- compatible units or a clearly labeled scale
- the same time domain
- similar sampling and aggregation semantics

Good shared-axis groups include CPU and memory percentages. Temperature, latency, signal strength, bytes per second, and percentages usually need separate axes, panes, or charts.

The axis API supports DateTime value types, named axes, range configuration, interval settings, and zoom factors. Use those features deliberately rather than forcing unrelated metrics onto one scale. 

## Update strategy

Choose an update path according to source frequency:

- Low frequency: append each validated sample immutably.
- Moderate frequency: append small batches on a short interval.
- High frequency: aggregate before updating React state.
- Large fleets: update only visible or selected device charts.

Keep the incoming transport separate from display state. This makes reconnect, buffering, aggregation, and cleanup easier to reason about.

## Aggregation guidance

Select aggregation according to the signal:

- average for typical utilization
- minimum and maximum for ranges and spikes
- sum for counts within a bucket
- percentile for latency distributions
- first and last for state transitions
- last for current-state snapshots

Do not downsample by taking every nth point when the skipped points may contain short failures or peaks.

## Labels and tooltips

Keep time labels short and move date context into the title, period control, or tooltip. Tooltips can be shared across series with the same timestamp, include markers, and use format strings or templates. 

```tsx
<ChartTooltip
  enable={true}
  shared={true}
  showMarker={true}
  format="${series.name}: ${point.y}"
/>
```

Use per-series `tooltipFormat` when units differ. Avoid expensive templates during pointer motion.

## Animation and visual density

Series animation defaults to enabled, so make an explicit choice for continuously updating charts. 

For dense live signals:

- disable animation or use a duration shorter than the refresh interval
- omit point markers
- avoid data labels on every sample
- use a last-value label or external current-value summary
- keep annotations limited to important events
- use subtle gridlines and a restrained palette

## Zoom and live-follow behavior

The zoom settings API supports selection zoom, mouse-wheel zoom, pinch zoom, panning, scrollbars, and a toolbar. These interactions are disabled by default. 

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

When users inspect history, do not silently move the viewport back to the latest sample. Pause live-follow or provide a visible “Return to live” action.

## Status and freshness

Track status independently of chart points:

- loading: waiting for the first response
- live: samples arrive within the expected interval
- stale: the most recent sample is too old
- disconnected: the source connection is unavailable
- empty: the source returned no samples
- error: retrieval or parsing failed

Use a stable external status element and announce only meaningful changes, not every sample.

```tsx
<p role="status" aria-live="polite">
  {statusLabel}
</p>
```

## Accessibility

- Give each chart a scope-specific accessible name.
- Provide current value, trend, severity, and freshness in text.
- Make time-range, metric, pause, resume, and return-to-live controls keyboard operable.
- Keep focus order stable as values update.
- Do not rely only on color, hover, or animation.
- Provide a table, log, or download for precise historical values when needed.
- Avoid live-region announcements for every telemetry packet.

The root chart exposes accessibility and focus options, while documented chart events belong on the root component. 

## Performance checklist

Before optimizing, measure with production-like rates and retention:

- React commit duration
- memory growth over time
- stream backlog
- tooltip and pointer responsiveness
- zoom and pan responsiveness
- number of active series and visible charts
- aggregation cost
- reconnection behavior

Reduce unnecessary markers, labels, animation, templates, and active series before discarding meaningful data.

## Common errors

### Unbounded arrays

Always enforce a rolling window or point cap.

### Category strings for timestamps

Use real `Date` values and a `DateTime` axis when spacing and zooming must represent elapsed time.

### Every sensor on one chart

Group compatible metrics and let users select the current scope.

### One axis for incompatible units

Use named axes, panes, or separate charts.

### Refresh-dependent layout shifts

Reserve stable chart and card dimensions for every state.

### Overlapping animation

Ensure animation duration is shorter than the update interval or disable animation.

### Missing cleanup

Remove subscriptions and listeners, close sockets, clear timers, disconnect observers, and abort pending requests.

### Synthetic values presented as real

Clearly label demo data. Never imply generated telemetry came from operational systems.

## Validation checklist

Before returning a real-time chart implementation:

1. Route detailed decisions to the cloud or IoT page.
2. Use real `Date` values with a `DateTime` axis.
3. Define sampling rate, visible window, and freshness threshold.
4. Bound every live series.
5. Validate incoming timestamps and finite values.
6. Use immutable state updates.
7. Batch or aggregate high-frequency input before rendering.
8. Limit active series and visible live charts.
9. Group only compatible metrics.
10. Keep names, labels, tooltips, and units concise.
11. Make an explicit animation decision.
12. Preserve peaks and state transitions during aggregation.
13. Keep layout dimensions stable.
14. Distinguish loading, live, stale, disconnected, empty, and error states.
15. Define live-follow behavior during zoom.
16. Clean up all data-source resources.
17. Keep high-frequency event handlers lightweight.
18. Provide accessible status and historical detail.
19. Do not announce every sample.
20. Test long-running memory and reconnection behavior.
21. Do not fabricate operational data.
22. Do not mix EJ2 and Pure React APIs.
23. Ensure every imported symbol is used.
24. Emit valid, unescaped TSX.
