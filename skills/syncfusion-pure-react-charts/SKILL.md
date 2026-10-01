---
name: syncfusion-pure-react-charts
description: Build interactive React charts with the Syncfusion Pure React package @syncfusion/react-charts (Chart, PieChart). Covers all 50 Cartesian series types (line, spline, area, column, bar, stacking, range, step, scatter, bubble, polar, radar, candle, hilo, waterfall, histogram, pareto, box-and-whisker, multi-colored) plus pie and donut. Use whenever the user mentions Syncfusion charts, @syncfusion/react-charts, or wants React data visualization with tooltips, zoom/pan, selection, crosshair, legends, axes and panes, multi-level labels, strip lines, annotations, trendlines, technical indicators, gradients, range color mapping, local/remote data binding, live dashboards, accessibility, theming, or image/PDF/Excel/CSV export and print.
metadata:
  author: "Syncfusion Inc"
  version: "34.1.29"
---

# React Chart Implementation Skill

**Build interactive, production-grade React Charts with Syncfusion components.**

This skill is a routing hub for chart implementation: setup, overview, types, data binding, interactivity, accessibility, styling, performance, real-time patterns, troubleshooting, and production workflows. Read only the references the current request needs; detailed patterns, API shape, and validation rules live in those references.

## Scope Covered by This Skill

This skill is intentionally aligned to the chart topics documented in the charts source set, including:

- Getting started, overview, layout and styling, accessibility, keyboard navigation, globalization, RTL, print/export
- Events, tooltip, zooming and panning, selection and highlight, crosshair, animation
- Axis customization, labels, position, category, numeric, datetime, logarithmic, multiple axes, multiple panes
- Local and remote data binding
- Chart elements: annotations, data labels, markers, series labels, last value labels, error bars, striplines, trendlines, legends, indicators
- Line, spline, step line, area, range area, spline area, step area, column, bar, range column, stacking, combination charts
- Bubble, scatter, histogram, pareto, waterfall, box-and-whisker, multi-colored line/area, pie, donut, polar, radar, and financial chart families
- Gradient fills, range color mapping, multi-level axis labels, donut center labels, pie selection/highlight/annotations
- Image (SVG/PNG/JPG), PDF, and data (XLSX/CSV) export through `exportChart`, and printing
- Real-time dashboard patterns and live updates

When a request maps to one of those areas, prefer the corresponding reference guidance and the package-provided component hierarchy over custom drawing logic.

---

## Quick Start

### Install Dependencies

```bash
npm install @syncfusion/react-charts @syncfusion/react-base
```

### Import CSS

```tsx
import "@syncfusion/react-base/styles/material.css";
```

### Create a Chart

```tsx
import { Chart, ChartSeriesCollection, ChartSeries } from "@syncfusion/react-charts";

const data = [
  { month: "Jan", sales: 100 },
  { month: "Feb", sales: 120 },
];

export default function App() {
  return (
    <Chart>
      <ChartSeriesCollection>
        <ChartSeries dataSource={data} xField="month" yField="sales" type="Column" />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

For pie and donut requests, render with the `PieChart` component family exported by `@syncfusion/react-charts`; create a donut by setting `PieChartSeries.innerRadius` above `0%`. Do not build slices with SVG, Canvas, CSS arcs, or manual angle math.

Full setup walkthrough, theme options, and the canonical first-chart pattern: [getting-started.md](./references/getting-started.md).

---

## Reference Routing

| Request topic | Read first |
| --- | --- |
| Setup, basic chart | [getting-started.md](./references/getting-started.md) |
| Component overview, surface vs. series hierarchy | [overview.md](./references/overview.md) |
| Choosing a chart `type` and required fields | [chart-types.md](./references/chart-types.md) |
| Pie or donut, center label, pie selection/annotations | [pie-and-donut.md](./references/pie-and-donut.md) |
| Axes, panes, labels, multiple scales | [axis-configuration.md](./references/axis-configuration.md) |
| Data binding (local, React state, remote API, `DataManager`) | [data-binding.md](./references/data-binding.md) |
| Annotations, labels, markers, legends, trendlines, striplines | [chart-elements.md](./references/chart-elements.md) |
| Gradient fills on series, trendlines, indicators | [chart-elements/gradients.md](./references/chart-elements/gradients.md) |
| Coloring points by value band, range/gradient legend | [chart-elements/range-color-mapping.md](./references/chart-elements/range-color-mapping.md) |
| Grouped (multi-level) axis labels | [axis-configuration/multi-level-labels.md](./references/axis-configuration/multi-level-labels.md) |
| Tooltips, zoom, pan, crosshair, selection, highlight | [interactivity.md](./references/interactivity.md) |
| Event callback props and signatures | [events.md](./references/events.md) |
| Responsive sizing, container layout, density | [layout-and-styling.md](./references/layout-and-styling.md) |
| Themes, palettes, visual customization | [styling-and-appearance.md](./references/styling-and-appearance.md) |
| Print, image/PDF export, Excel/CSV data export | [print-and-export.md](./references/print-and-export.md) |
| Localization, locale-aware formatting | [globalization.md](./references/globalization.md) |
| Right-to-left rendering | [rtl.md](./references/rtl.md) |
| WCAG/ARIA, screen readers, color contrast, RTL | [accessibility.md](./references/accessibility.md) |
| Keyboard behavior | [keyboard-navigation.md](./references/keyboard-navigation.md) |
| Real-time / live updating dashboards | [real-time-charts.md](./references/real-time-charts.md) |
| Multi-axis, dynamic series, optimization, testing | [advanced-patterns.md](./references/advanced-patterns.md) |
| Errors, validation, diagnostics | [troubleshooting.md](./references/troubleshooting.md) |

When a routed reference is an index, read only the linked nested reference matching the requested feature. Do not load unrelated references.

---

## Chart Selector by Use Case

| Use case | Best chart | Why |
| --- | --- | --- |
| Trend over time | Line, Spline | Shows progression naturally |
| Compare categories | Column, Bar | Side-by-side comparison |
| Part-to-whole | Pie, Donut | Visualize percentages |
| Distribution | Histogram | Show frequency bins |
| Correlation | Scatter, Bubble | Display relationships |
| Radial / Cyclic | Radar, Polar | Multi-dimensional comparison |
| Financial data | Candle, Hilo, HiloOpenClose | Stock market visualization |
| Multiple perspectives | Combination charts | Multiple metrics in one chart |

The exhaustive list of supported series `type` literals, exact field mappings, and per-type configuration lives in [chart-types.md](./references/chart-types.md). Pie and donut specifics (component hierarchy, `innerRadius`, labels) live in [pie-and-donut.md](./references/pie-and-donut.md).

---

## Verified Implementation Rules

These rules are enforced across every reference. Conform to them when generating code.

- Use the exact component names exported by `@syncfusion/react-charts`; do not invent aliases or rely on legacy names.
- Place chart-level elements such as `ChartTitle`, `ChartSubtitle`, `ChartArea`, `ChartLegend`, `ChartTooltip`, `ChartZoomSettings`, `ChartSelection`, and `ChartHighlight` directly under `Chart`.
- Configure axes with the verified child configuration components: place `ChartAxisTitle`, `ChartAxisLabel`, `ChartMajorGridLines`, `ChartMinorGridLines`, and `ChartMinorTickLines` inside `ChartPrimaryXAxis` / `ChartPrimaryYAxis`.
- Wrap panes as `ChartRows > ChartRow` and `ChartColumns > ChartColumn`; a bare `ChartRow` or `ChartColumn` under `Chart` is ignored.
- Define extra axes with `ChartAxes`, give each axis a unique `name`, and map each series through `xAxisName` / `yAxisName`. Do not use a guessed `ChartSecondaryYAxis` tag.
- Configure strip lines through `ChartStripLines` and `ChartStripLine` nested inside the axis; set `range={{ start, end }}`, `style={{ color }}`, and `text={{ content }}` objects (not flat `start`/`color`/`text` props).
- Use `ChartAxisLabel intersectAction` for overlapping axis labels; `intersectMode` belongs only to `ChartDataLabel`.
- Nest point adornments in the series hierarchy: `ChartMarker` contains `ChartDataLabel`, and `ChartTrendlineCollection` contains `ChartTrendline`. Pareto line options go in a `ChartParetoOptions` child; gradients go in a `ChartLinearGradient` / `ChartRadialGradient` child of the series, trendline, or indicator.
- Map bubble size with `sizeField` (not `size`); enable last-value labels with `ChartLastValueLabel enable`.
- Configure legend and tooltip with the verified prop names, examples: `ChartTooltip headerText`, `ChartLegend align`, `ChartTooltip textStyle.fontSize`.
- Use the verified event prop names such as `onClick`, `onMouseMove`, `onPointClick`, `onLegendClick`, `onAxisLabelClick`, `onMultiLevelLabelClick`, `onZoomStart`, `onZoomEnd`, and `onResize` on the root `Chart`, and type handlers with the exported event types (`LegendClickEvent`, `PointClickEvent`, and so on).
- Export with `exportChart(chart, type, fileName)` where `type` is `SVG`, `PNG`, `JPG`, `PDF`, `XLSX`, or `CSV`; `exportImage` and `exportPDF` are deprecated.
- For pie and donut, use the `PieChart` family. Create a donut with `PieChartSeries.innerRadius`; there is no `isCircular` prop and no `Pie` series type on `ChartSeries`.
- Before finalizing any generated sample, verify every imported symbol is used, every component is exported by `@syncfusion/react-charts`, and the resulting tree matches the verified parent/child hierarchy.

---

## Workflow

1. **Choose chart type** — Use the selector table above, then confirm the exact `type` literal and field requirements in [chart-types.md](./references/chart-types.md).
2. **Prepare data** — Ensure data is an array of objects with consistent structure; for pie/donut, follow [pie-and-donut.md](./references/pie-and-donut.md).
3. **Create a basic chart** — Use a template from [getting-started.md](./references/getting-started.md) or [chart-types.md](./references/chart-types.md).
4. **Add interactivity** — Enable tooltips, zoom/pan, selection, crosshair per [interactivity.md](./references/interactivity.md); wire events per [events.md](./references/events.md).
5. **Style and theme** — Apply palettes, theme stylesheet, and responsive sizing per [styling-and-appearance.md](./references/styling-and-appearance.md) and [layout-and-styling.md](./references/layout-and-styling.md).
6. **Accessibility** — Set `ariaLabel`, focus outline, keyboard support, and color contrast per [accessibility.md](./references/accessibility.md) and [keyboard-navigation.md](./references/keyboard-navigation.md); honor RTL via [rtl.md](./references/rtl.md) and locales via [globalization.md](./references/globalization.md).
7. **Optimize performance** — For 1000+ points or live streams, follow [advanced-patterns.md](./references/advanced-patterns.md) and [real-time-charts.md](./references/real-time-charts.md).
8. **Print/export and ship** — Use [print-and-export.md](./references/print-and-export.md); when something breaks, walk [troubleshooting.md](./references/troubleshooting.md).

---

## Validation Checklist for Generated Samples

- Use package-exported chart components only; do not invent custom SVG or HTML chart primitives.
- Use the correct chart family for the request: Cartesian `Chart` for line/bar/column/area, `PieChart` for pie and doughnut.
- Bind data directly to series via `dataSource` with the appropriate field mappings (`xField`, `yField`, plus `sizeField`, `low`/`high`, `open`/`high`/`low`/`close` as required).
- Avoid manual geometry, coordinate math, or slice path construction when the package supports the chart type.
- Keep component nesting declarative and consistent with the verified reference patterns.
- Match axes, labels, tooltip, legend, and interactivity props to the documented API shape.
- Ensure the final sample is minimal for basic requests and is expanded only when the user asks for more features.

---

## Best Practices Checklist

- Base theme CSS imported (`@syncfusion/react-base/styles/material.css`)
- Data shape verified; new array reference used for React state updates
- Expensive transformations memoized with `useMemo`; effects cleaned up on unmount
- Real-time data bounded (60–100 points); animations disabled for 1000+ points when needed
- Keyboard navigation, color contrast (WCAG AA), legend, tooltips, responsive sizing verified
- Error handling present for API failures; production license key configured

---

## When to Use Each Reference

| Stage | Read |
| --- | --- |
| Just starting | `chart-types.md` + `getting-started.md` |
| Binding data | `data-binding.md` |
| Making it interactive | `interactivity.md` |
| Styling and theming | `styling-and-appearance.md` + `layout-and-styling.md` |
| Accessibility | `accessibility.md` + `keyboard-navigation.md` |
| Performance issues | `advanced-patterns.md` + `real-time-charts.md` |
| Something broken | `troubleshooting.md` |
| Need complex features | `advanced-patterns.md` + `chart-elements.md` |
| Pie or donut specifically | `pie-and-donut.md` |
| Localization / RTL | `globalization.md` + `rtl.md` |
| Print or export | `print-and-export.md` |
| Register a license key | `registering-license-keys.md` |
| Gradients, range colors, grouped axis labels | `chart-elements/gradients.md`, `chart-elements/range-color-mapping.md`, `axis-configuration/multi-level-labels.md` |

---
