---
name: integrated-charts
description: Visualize grid data as a chart via the Syncfusion React Data Grid's `GridChart` companion component — installed from `@syncfusion/react-grid-chart`, opened with `gridChartRef.show(settings)`, fully configurable `chart` (Column / Bar / Line / Area / Pie / Scatter plus stacked and 100%-stacked variants), `category`, `series`, `enableExport` (`Print | PNG | JPEG | SVG | PDF`), `enablePropertyPanel` (Chart / Data / Format tabs), and `dialogSettings`. Triggering via the context menu's built-in chart items or programmatically through `onContextMenuClick` is supported. Load when wiring a chart dialog out of the grid, adding chart exports, or surfacing chart-from-selection in a custom context menu.
---

# Integrated chart (GridChart)

The companion `@syncfusion/react-grid-chart` package renders a chart dialog using the grid's data. The grid and chart share the same `dataSource`, the dialog appears as a `Dialog`, and the chart supports edit / export / customization through tabs and a property panel.

## Install and import

```bash
npm install @syncfusion/react-grid-chart --save
```

```tsx
import {
  Grid, Columns, Column, GridRef, ContextMenuModule, ContextMenuSettings,
} from '@syncfusion/react-grid';
import { GridChart, GridChartRef, GridChartSettings } from '@syncfusion/react-grid-chart';

const modules = { ContextMenuModule };
```

> `GridChart` is rendered alongside `<Grid>` as a sibling, not as a descendant. The `gridChartRef.show(...)` method opens the dialog imperatively.

## Programmatic show

```tsx
const gridChartRef = useRef<GridChartRef>(null);
const chartSettings: GridChartSettings = {
  chart: 'Column',                            // see "Chart types"
  dataSource: chartData,                      // use the grid's data, optionally restricted to selection
  category:  ['Product', 'Year'],             // fields grouped on the X axis (or slice)
  series:    ['Revenue', 'ProfitLoss'],       // measure fields
  enableExport:         true,
  chartExportItems:     ['Print', 'PNG', 'JPEG', 'SVG', 'PDF'],
  enablePropertyPanel:  true,                // Chart / Data / Format tabs
  dialogSettings: {
    header:   'Sales Performance by Product',
    className:'sales-chart-dialog',
    draggable: true,
  },
};

const handleShowChart = () => gridChartRef.current?.show(chartSettings);

return (
  <>
    <button onClick={handleShowChart}>Show Sales Chart</button>
    <Grid dataSource={sales} /* … */ >
      <Columns>{/* … */}</Columns>
    </Grid>
    <GridChart ref={gridChartRef} />
  </>
);
```

### `GridChartSettings` shape

| Property | Type | Description |
|---|---|---|
| `chart` | `ChartType` (string) | Chart family — see "Chart types". |
| `dataSource` | `object[]` | Records passed to the chart. Use the grid's data or a subset (`getSelectedRecords()`). |
| `category` | `string[]` | Field names used on the category axis. |
| `series` | `string[]` | Measure field names rendered as separate series. |
| `enablePropertyPanel` | `boolean` | Show the Chart / Data / Format tabs. |
| `enableExport` | `boolean` | Show the chart Export button. |
| `chartExportItems` | `('Print' \| 'PNG' \| 'JPEG' \| 'SVG' \| 'PDF')[]` | Which export formats appear. |
| `dialogSettings` | `DialogSettings` | Passthrough to the underlying Dialog — `header`, `className`, `draggable`, `animation`, `width`, `height`, etc. |

## Triggering from the context menu

`enableGridChart: true` plus `contextMenuSettings` with built-in chart items wires the trigger directly. Each menu item carries a chart type as `args.chart`:

```tsx
const [contextMenuSettings] = useState<ContextMenuSettings>({
  enabled: true,
  items: [
    'Bar', 'StackingBar', 'StackingBar100',
    'Pie',
    'Column', 'StackingColumn', 'StackingColumn100',
    'Line', 'StackingLine', 'StackingLine100',
    'Area', 'StackingArea', 'StackingArea100',
    'Scatter',
  ],
});

const onContextMenuClick = (args: { chart: ChartType }) => {
  const selected = gridRef.current?.getSelectedRecords() ?? [];
  gridChartRef.current?.show({
    chart: args.chart,
    dataSource: selected.length ? selected : sales,
    category: ['Product', 'Year'],
    series: ['Revenue', 'ProfitLoss'],
    enablePropertyPanel: true,
    enableExport: true,
    chartExportItems: ['Print', 'PNG', 'JPEG', 'SVG', 'PDF'],
    dialogSettings: { header: `${args.chart} Chart - Sales Performance` },
  });
};

<Grid
  ref={gridRef}
  dataSource={sales}
  contextMenuSettings={contextMenuSettings}
  onContextMenuClick={onContextMenuClick}
  enableGridChart={true}
  modules={modules}
/>
```

When **any rows are selected**, the chart restricts to the selection; otherwise it falls back to the full dataset. This is a useful default — pass `selected.length ? selected : sales` to grant users a "chart my selection" UX for free.

## Chart types

| Family | Variants |
|---|---|
| Bar | `'Bar'`, `'StackingBar'`, `'StackingBar100'` |
| Column | `'Column'`, `'StackingColumn'`, `'StackingColumn100'` |
| Line | `'Line'`, `'StackingLine'`, `'StackingLine100'` |
| Area | `'Area'`, `'StackingArea'`, `'StackingArea100'` |
| Pie | `'Pie'` |
| Scatter | `'Scatter'` |

Each chart family supports its own layout conventions; see `chart-types` upstream for end-to-end samples.

## Property panel — Chart / Data / Format

`enablePropertyPanel: true` opens three tabs that edit the chart in-place:

| Tab | Description |
|---|---|
| Chart | Switch between supported chart types — the standard / stacked / 100%-stacked variants are grouped by family (Column, Bar, Line, Area, Pie, Scatter). |
| Data | Reassign `category` and `series` fields; the chart re-renders immediately. |
| Format | Customize layout, colors, fonts, legends, tooltips, data labels, axis settings. |

Toggle the property panel at runtime by binding `enablePropertyPanel` to state.

## Export

`enableExport: true` + `chartExportItems: ['Print', 'PNG', 'JPEG', 'SVG', 'PDF']` exposes the Export menu. The chart dialog takes a snapshot and renders it through whichever converter is registered:

| Format | Use |
|---|---|
| `Print` | Browser print dialog with the chart rendered at print size. |
| `PNG`, `JPEG`, `SVG` | Encoded raster / vector images, suitable for embedding in reports. |
| `PDF` | Chart converted to PDF; useful for export pipelines. |

Toggling `enableExport` at runtime shows/hides the Export button instantly.

## Dialog customization

`dialogSettings` accepts the standard Dialog config:

```tsx
dialogSettings: {
  header: 'Sales Performance by Product',
  className: 'sales-chart-dialog',
  draggable: true,
  animation: { effect: 'FadeZoom', duration: 300 },
  width: 720,
  height: 520,
}
```

`draggable: true` lets the user reposition the chart dialog inside the grid; `animation` controls the open / close transitions.

## Constraints & guardrails

- **Missing `@syncfusion/react-grid-chart`** — runtime imports silently fail. Install the package and verify it is listed in `package.json`.
- **`GridChart`** must be rendered as a sibling of `<Grid>` with the `ref` attached; rendering it inside the grid is a known regression.
- **Custom context-menu items** must include `'chart'`-like handlers — built-in items already provide `args.chart`. Custom items need a switch on `args.item.id`.
- **Selection-driven charts**: when `selectionSettings.persistSelection: true` or rows are scattered across pages, the user's selection may include items not currently rendered. Always reconcile via `gridRef.current?.getSelectedRecords()` rather than reading directly off the DOM.
- **Large datasets**: charts swallow all `dataSource` records into the SVG / canvas. For >5k rows, pre-aggregate (server-side or via `AggregateModule`) before passing to `dataSource`.
- **High-risk exports**: PDF export of selection or full grid can leak sensitive data. Confirm intent before invoking `chartExportItems: ['PDF']` programmatically.

## Cross-references

- For data flow into and out of `GridChart`, see `references/data-binding.md` (DataManager adaptors) and `references/selection.md` (selection-mode selection).
- For other export hooks (PDF / Excel / Print without the chart), see `references/exports.md`.
- For the context menu wiring, see `references/interactivity.md` (built-in items, `onContextMenuClick`).
