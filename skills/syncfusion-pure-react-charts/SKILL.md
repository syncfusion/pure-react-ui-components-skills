---
name: syncfusion-pure-react-charts
description: Build interactive React Charts with Syncfusion components. Covers all chart types (line, bar, column, area, pie, scatter, bubble, radar, polar, financial charts, and specialized types). Use for data visualization, interactivity (tooltips, zoom, pan, selection), data binding (local/remote), accessibility (WCAG), styling, performance optimization, and real-time dashboards. Complete reference with 20+ chart types, patterns, troubleshooting, and production best practices.
metadata:
  author: "Syncfusion Inc"
  version: "1.0.0"
---

# React Chart Implementation Skill

**Build interactive, production-grade React Charts with Syncfusion components.**

This comprehensive skill covers all aspects of chart implementation: setup, all 20+ chart types, data binding patterns, interactivity, accessibility, styling, performance optimization, and real-world patterns.

## Scope Covered by This Skill

This skill is intentionally aligned to the chart topics documented in the charts source set, including:
- Getting started, overview, layout and styling, accessibility, keyboard navigation, globalization, RTL, print/export
- Events, tooltip, zooming and panning, selection and highlight, crosshair, animation
- Axis customization, labels, position, category, numeric, datetime, logarithmic, multiple axes, multiple panes
- Local and remote data binding
- Chart elements: annotations, data labels, markers, series labels, last value labels, error bars, striplines, trendlines, legends, indicators
- Line, spline, step line, area, range area, spline area, step area, column, bar, range column, stacking, combination charts
- Bubble, scatter, histogram, pareto, waterfall, pie, donut, polar, radar, and financial chart families
- Real-time dashboard patterns and live updates

When a request maps to one of those areas, prefer the corresponding reference guidance and the package-provided component hierarchy over custom drawing logic.

---

## 📋 Quick Start

### 1. Install Dependencies
```bash
npm install @syncfusion/react-charts @syncfusion/react-base
```

### 2. Import CSS
```tsx
import "@syncfusion/react-base/styles/material.css";
```

### 3. Create a Chart
```tsx
import { Chart, ChartSeriesCollection, ChartSeries } from "@syncfusion/react-charts";

export default function App() {
  const data = [
    { month: 'Jan', sales: 100 },
    { month: 'Feb', sales: 120 }
  ];

  return (
    <Chart>
      <ChartSeriesCollection>
        <ChartSeries 
          dataSource={data} 
          xField="month" 
          yField="sales" 
          type="Column" 
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

### Pie and Donut Charts
For pie and donut requests, use the consolidated pie/donut guidance in `references/chart-types.md` and the pie chart component family exported by `@syncfusion/react-charts`.

Do not build pie slices with SVG, Canvas, CSS arcs, or manual angle/circumference calculations.

### Verified implementation rules
- Use the exact component names exported by the package; do not invent aliases or rely on legacy names.
- Place chart-level elements such as `ChartTitle`, `ChartSubtitle`, `ChartArea`, `ChartLegend`, `ChartTooltip`, `ChartZoomSettings`, `ChartSelection`, and `ChartHighlight` directly under `Chart`.
- Configure axes with child configuration components instead of string props: use `ChartAxisTitle`, `ChartAxisLabel`, `ChartMajorGridLines`, `ChartMinorGridLines`, and `ChartMinorTickLines` inside `ChartPrimaryXAxis` / `ChartPrimaryYAxis`.
- Define extra axes with `ChartAxes`, and map each series to the matching axis name when a sample needs multiple scales or units.
- Configure strip lines through `ChartStripLines` and `ChartStripLine` children on the axis, and use the verified range object pattern when specifying an interval or single threshold.
- Nest point adornments in the series hierarchy: `ChartMarker` contains `ChartDataLabel`, and `ChartTrendlineCollection` contains `ChartTrendline`.
- Prefer the event prop names used by the chart surface in the package API, such as `onClick`, `onMouseMove`, and `onLegendClick`, and type event handlers with the exported event types when available.
- For legend and tooltip configuration, use the verified prop names from the package API, including `align`, `headerText`, and `textStyle.fontSize`.
- Before finalizing a generated sample, check that every imported symbol is used, every component is exported by `@syncfusion/react-charts`, and the resulting tree matches the expected parent/child hierarchy.

## Reference Routing

Read only the references required for the current request:

- Setup or basic chart: `references/getting-started.md`
- Component overview: `references/overview.md`
- Chart and series types: `references/chart-types.md`
- Axes and panes: `references/axis-configuration.md`
- Data binding and updates: `references/data-binding.md`
- Labels, markers, legends, annotations, indicators, and trendlines: `references/chart-elements.md`
- Tooltips, zooming, panning, crosshair, selection, and highlighting: `references/interactivity.md`
- Events and methods: `references/events.md`
- Layout and responsive sizing: `references/layout-and-styling.md`
- Themes and visual customization: `references/styling-and-appearance.md`
- Printing and exporting: `references/print-and-export.md`
- Globalization and localization: `references/globalization.md`
- Right-to-left rendering: `references/rtl.md`
- Accessibility: `references/accessibility.md`
- Keyboard behavior: `references/keyboard-navigation.md`
- Real-time updates: `references/real-time-charts.md`
- Errors and validation: `references/troubleshooting.md`

When a routed reference is an index, read only the linked nested reference
matching the requested feature. Do not load unrelated references.

## 📊 Chart Types Guide

### Quick Selector by Use Case

| Use Case | Best Chart | Why |
|----------|-----------|-----|
| Trend over time | Line, Spline | Shows progression naturally |
| Compare categories | Column, Bar | Side-by-side comparison |
| Part-to-whole | Pie, Donut | Visualize percentages |
| Distribution | Histogram | Show frequency bins |
| Correlation | Scatter, Bubble | Display relationships |
| Radial/Cyclic | Radar, Polar | Multi-dimensional comparison |
| Financial data | Candle, Hilo, HiloOpenClose | Stock market visualization |
| Multiple perspectives | Combo | Multiple metrics in one chart |

### All Supported Types
- **Line variants**: Line, Spline, StepLine, MultiColoredLine, StackingLine, StackingLine100
- **Area variants**: Area, SplineArea, StepArea, RangeArea, SplineRangeArea, RangeStepArea, StackingArea, StackingArea100, StackingStepArea, MultiColoredArea
- **Column/Bar**: Column, Bar, RangeColumn, StackingColumn, StackingColumn100, StackingBar, StackingBar100, Waterfall
- **Pie/Donut**: Use the separate `PieChart` component family; create a donut with `PieChartSeries` `innerRadius` above `0%` (there is no `isCircular` prop and no `Pie` series type on `ChartSeries`)
- **Scatter/Bubble**: Scatter, Bubble
- **Radial**: Polar and Radar variants (PolarLine, PolarArea, PolarColumn, PolarSpline, PolarSplineArea, PolarScatter, PolarRangeColumn, PolarStackingArea, PolarStackingColumn, RadarLine, RadarArea, RadarColumn, RadarSpline, RadarSplineArea, RadarScatter, RadarStackingArea, RadarStackingColumn, RadarRangeColumn)
- **Special**: Histogram, Pareto, BoxAndWhisker
- **Financial**: Candle, Hilo, HiloOpenClose

**Detailed reference**: See `references/chart-types.md` for configuration of all types.

---

## 🔗 Data Binding Patterns

### Local Data (Static)
```tsx
const data = [
  { x: 'A', y: 100 },
  { x: 'B', y: 120 }
];

<ChartSeries dataSource={data} xField="x" yField="y" />
```

### State-Driven (Dynamic Updates)
```tsx
const [data, setData] = useState([...]);

const addPoint = () => {
  setData([...data, newPoint]);  // ← Always create new array
};
```

### Remote Data (API)
```tsx
useEffect(() => {
  fetch('/api/chart-data')
    .then(r => r.json())
    .then(setData)
    .catch(err => console.error(err));
}, []);
```

### Real-Time Streaming (WebSocket)
```tsx
useEffect(() => {
  const ws = new WebSocket('ws://stream.example.com');
  ws.onmessage = (event) => {
    const newPoint = JSON.parse(event.data);
    setData(prev => [...prev.slice(-59), newPoint]);  // Keep 60 points
  };
  return () => ws.close();
}, []);
```

**Detailed patterns**: See `references/data-binding.md`.
---

## 🎨 Styling & Appearance

### Basic Theme
```tsx
<Chart
  background="#FFFFFF"
  palettes={['#0066CC', '#00AA00', '#FF6600']}
>
  <ChartArea background="#F9F9F9" />
  <ChartLegend visible={true} position="Bottom" />
  <ChartSeriesCollection>
    <ChartSeries dataSource={data} fill="#0066CC" />
  </ChartSeriesCollection>
</Chart>
```

### Dark Mode Theme
```tsx
<Chart
  background="#1E1E1E"
  palettes={['#FF6B6B', '#4ECDC4', '#45B7D1']}
  border={{ color: '#404040', width: 1 }}
>
  <ChartArea background="#2D2D2D" />
  {/* Rest of configuration */}
</Chart>
```

### Responsive Sizing
```tsx
<div style={{ width: '100%', height: '500px' }}>
  <Chart width="100%" height="100%">
    {/* Chart fills container */}
  </Chart>
</div>
```

**Comprehensive styling guide**: See `references/styling-and-appearance.md`.

---

## 🖱️ Interactivity Features

### Tooltips
```tsx
<Chart>
  <ChartTooltip 
    enable={true}
    format="${point.x}: ${point.y}"
    shared={true}  // Show all series at same X
  />
</Chart>
```

### Zoom & Pan
```tsx
<Chart>
  <ChartZoomSettings
    selectionZoom={true}        // Drag to zoom
    mouseWheelZoom={true}       // Scroll to zoom
    pinchZoom={true}            // Touch pinch
    pan={true}                  // Drag to pan when zoomed
    toolbar={{ visible: true }} // Show zoom toolbar
  />
</Chart>
```

### Selection & Highlight
```tsx
<Chart>
  <ChartSelection mode="Point" allowMultiSelection={true} />
  <ChartHighlight mode="Point" fill="rgba(0,0,255,0.3)" />
</Chart>
```

### Events
```tsx
<Chart
  onClick={(args) => console.log('Click:', args.target)}
  onMouseMove={(args) => console.log('Move:', args.target)}
  onPointClick={(args) => console.log('Point:', args.seriesIndex, args.pointIndex)}
  onLegendClick={(args) => console.log('Legend:', args.seriesName)}
/>
```

**Full interactivity guide**: See `references/interactivity.md`.

---

## 📌 Chart Elements

### Annotations
```tsx
<ChartAnnotationCollection>
  <ChartAnnotation
    x={2020}
    y={50}
    coordinateUnit="Point"
    content="Peak Sales"
  />
</ChartAnnotationCollection>
```

### Data Labels
```tsx
<ChartSeries>
  <ChartMarker>
  <ChartDataLabel
    visible={true}
    position="Top"
    format="${point.y}"
  />
  </ChartMarker>
</ChartSeries>
```

### Markers
```tsx
<ChartSeries type="Line">
  <ChartMarker 
    visible={true}
    shape="Circle"
    width={10}
    height={10}
  />
</ChartSeries>
```

### Trendlines
```tsx
<ChartSeries type="Scatter">
  <ChartTrendlineCollection>
    <ChartTrendline type="Linear" name="Trend" />
  </ChartTrendlineCollection>
</ChartSeries>
```

### Striplines (Reference Lines)
```tsx
<ChartPrimaryYAxis>
  <ChartStripLines>
    <ChartStripLine
      range={{ start: 100, end: 100 }}
      style={{ color: '#FFE5E5' }}
      text={{ content: 'Target' }}
    />
  </ChartStripLines>
</ChartPrimaryYAxis>
```

**Complete elements reference**: See `references/chart-elements.md`.
---

## ♿ Accessibility (WCAG 2.2)

### Configuration
```tsx
<Chart
  accessibility={{
    ariaLabel: "Quarterly sales performance chart",
    focusable: true,
    tabIndex: 0
  }}
  focusOutline={{
    color: '#0066CC',
    width: 2
  }}
>
  {/* Chart content */}
</Chart>
```

### Keyboard Navigation
- **Tab**: Focus the chart container
- Chart-level keyboard commands vary by package version, chart type, and enabled features
- Provide keyboard-operable external controls for essential actions
- Show the zoom toolbar when keyboard zoom is required

**Do not publish unverified shortcuts** (such as `Ctrl + +/-` or `R`) unless they are documented for the installed Pure React version and tested. See `references/keyboard-navigation.md`.

### Screen Reader Support
- All elements have ARIA labels
- Semantic structure announced
- Series names and values read aloud

**Full accessibility guide**: See `references/accessibility.md`.

---

## ⚡ Performance Optimization

### Large Datasets (1000+ points)
```tsx
// Strategy 1: Aggregate data
const aggregated = data.filter((_, idx) => idx % 10 === 0);

// Strategy 2: Disable animations
<ChartSeries animation={{ enable: data.length < 1000 }} />

// Strategy 3: Memoize transformations
const chartData = useMemo(() => transform(data), [data]);
```

### Real-Time Updates
```tsx
// Limit buffer size
setData(prev => {
  const updated = [...prev, newPoint];
  return updated.length > 100 
    ? updated.slice(-100)  // Keep only last 100
    : updated;
});
```

**Advanced optimization patterns**: See `references/advanced-patterns.md`.

---

## 🔧 Common Patterns

### Multi-Axis Charts
```tsx
<Chart>
  <ChartPrimaryYAxis>
    <ChartAxisTitle text="Revenue" />
  </ChartPrimaryYAxis>

  <ChartAxes>
    <ChartAxis name="growthAxis" valueType="Double" opposedPosition={true}>
      <ChartAxisTitle text="Growth %" />
    </ChartAxis>
  </ChartAxes>

  <ChartSeriesCollection>
    <ChartSeries dataSource={data} xField="month" yField="revenue" type="Column" />
    <ChartSeries dataSource={data} xField="month" yField="growth" type="Line" yAxisName="growthAxis" />
  </ChartSeriesCollection>
</Chart>
```

### Dynamic Series Management
```tsx
const [series, setSeries] = useState([...]);

<ChartSeriesCollection>
  {series.map(s => (
    <ChartSeries key={s.id} dataSource={s.data} name={s.name} />
  ))}
</ChartSeriesCollection>
```

**Complete advanced patterns**: See `references/advanced-patterns.md`.

---

## 📝 Examples

Ready-to-copy examples for 10 common scenarios:
- Basic Column Chart
- Multi-Series Line Chart
- Real-Time Data Updates
- Pie Chart
- Stacked Chart
- Zoom & Pan
- Data Labels & Markers
- Scatter Chart
- Combo Chart
- Area Chart

**See**: `references/chart-types.md` and `references/getting-started.md`

---

## 🐛 Troubleshooting

### Chart Not Rendering
- **Check**: CSS import (`@syncfusion/react-base/styles/material.css`)
- **Check**: dataSource is not undefined
- **Check**: Browser console for errors

### Data Not Updating
- **Problem**: Mutating array instead of creating new one
- **Solution**: Use spread operator: `[...data, newItem]`

### Performance Issues
- **Large datasets**: Aggregate or paginate
- **Real-time**: Limit buffer to 60-100 points
- **Animations**: Disable for 1000+ points

**Full troubleshooting guide**: See `references/troubleshooting.md`.

---

## 📚 Reference Guides

All references are organized by topic:

| Reference | Purpose |
|-----------|---------|
| `chart-types.md` | Configuration for all 20+ chart types |
| `data-binding.md` | Local, state, and remote data patterns |
| `interactivity.md` | Zoom, pan, selection, tooltips, events |
| `chart-elements.md` | Annotations, labels, markers, trendlines |
| `pie-and-donut.md` | Pie and doughnut chart patterns |
| `styling-and-appearance.md` | Colors, themes, responsive design |
| `accessibility.md` | WCAG compliance, keyboard, screen readers |
| `advanced-patterns.md` | Multi-axis, streaming, optimization |
| `troubleshooting.md` | Common issues and solutions |

---

## 🚀 Workflow

### 1. Choose Chart Type
Consult "Quick Selector" table to pick best type for your data.

### 2. Prepare Data
Ensure data is array of objects with consistent structure.

### 3. Create Basic Chart
Use a template from `references/chart-types.md` or `references/getting-started.md`.

For pie and doughnut requests, use the dedicated pie-chart family guidance in `references/pie-and-donut.md`.

### 4. Add Interactivity
Enable tooltips, zoom, selection based on needs.

### 5. Style & Theme
Apply brand colors, responsive sizing.

### 6. Test Accessibility
Keyboard navigation, screen reader, color contrast.

### 7. Optimize Performance
Aggregate data, memoize, disable animations if needed.

### 8. Deploy
Test in production, monitor performance.

## Validation Checklist for Generated Samples

- Use package-exported chart components only; do not invent custom SVG or HTML chart primitives.
- Use the correct chart family for the request: cartesian `Chart` for standard line/bar/column/area charts, `PieChart` for pie and doughnut charts.
- Bind data directly to series via `dataSource` and the appropriate field mappings.
- Avoid manual geometry, coordinate math, or slice path construction when the package supports the chart type.
- Keep component nesting declarative and consistent with the verified reference patterns.
- Match axes, labels, tooltip, legend, and interactivity props to the documented API shape.
- Ensure the final sample is minimal for basic requests and expanded only when the user asks for more features.

---

## ✅ Best Practices Checklist

- [ ] CSS imported: `@syncfusion/react-base/styles/material.css`
- [ ] Data structure verified (console.log first item)
- [ ] Using new array reference for state updates
- [ ] Expensive transformations memoized with useMemo
- [ ] Event listeners cleaned up (useEffect return)
- [ ] Real-time data limited to 60-100 points
- [ ] Animations disabled for 1000+ points
- [ ] Keyboard navigation tested
- [ ] Color contrast verified (WCAG AA minimum)
- [ ] Tooltips informative and non-blocking
- [ ] Legend visible and functional
- [ ] Responsive sizing on mobile/tablet
- [ ] Error handling for API failures
- [ ] Production license key configured
- [ ] Charts tested on target browsers

---

## 🔗 External Resources

- **Syncfusion React Chart Docs**: https://www.syncfusion.com/react-components/react-charts
- **GitHub Examples**: https://github.com/syncfusion/ej2-react-samples
- **WCAG 2.2 Standards**: https://www.w3.org/WAI/WCAG22/quickref/
- **React Accessibility**: https://react.dev/learn#accessibility

---

## 💡 When to Use Each Reference

**Just starting?** → `chart-types.md` + `getting-started.md`

**Need to bind data?** → `data-binding.md`

**Making it interactive?** → `interactivity.md`

**Styling & theming?** → `styling-and-appearance.md`

**Need accessibility?** → `accessibility.md`

**Performance issues?** → `advanced-patterns.md` + `troubleshooting.md`

**Something broken?** → `troubleshooting.md`

**Need complex features?** → `advanced-patterns.md` + `chart-elements.md`

---

**Skill Category**: Data Visualization / React Components
**Complexity**: Intermediate to Advanced
**Time to Proficiency**: 2-4 hours for basics, 1-2 weeks for advanced patterns
**Last Updated**: 2026
**Version**: 1.1.0

