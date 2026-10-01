# Print and Export Reference

Use the package-provided static functions when a Pure React Cartesian chart or pie chart must print or export. Prepare the chart for the target page or image size first, then call `exportChart` (image, PDF, Excel, or CSV) or `print` with a fully initialized chart instance.

## Core rules

- Keep titles, legends, axis labels, data labels, and annotations readable at the final output size.
- Give the chart a predictable width and height before printing or exporting.
- Use the package-provided static functions instead of undocumented instance methods.
- Export only after the chart instance and its root element are initialized.
- Prefer a simple, self-contained layout when export quality matters.
- Avoid hover-only information because tooltips are not a dependable part of static output.
- Test the actual image, PDF, and printed page rather than relying only on the browser preview.

## Supported operations

The current Pure React Chart static API provides:

- `exportChart(chart, type, fileName?, orientation?, header?, footer?)`: the single public export entry point for every format
- `print(chart)`: opens a print window and triggers the browser print dialog

`exportChart` accepts these `ExportType` values:

| `type` | Output | Notes |
| --- | --- | --- |
| `"SVG"` | Vector image | Best for scaling and further editing |
| `"PNG"` | Lossless raster image | Best default for documents and slides |
| `"JPG"` | Compressed raster image | Smaller files; no transparency |
| `"PDF"` | PDF document containing the chart image | Uses `orientation`, `header`, `footer` |
| `"XLSX"` | Excel workbook containing the chart **data** | Exports categories and series values, not the picture |
| `"CSV"` | Comma-separated chart **data** | Same data shape as XLSX |

`fileName` is the name without an extension; the extension is added automatically. When `fileName` is omitted, empty, or whitespace, `"Chart"` is used.

All functions accept either a Cartesian `IChart` or a pie-chart `IPieChart` instance. The chart must be fully initialized and contain a valid root element; otherwise the call returns without exporting.

### Deprecated functions

`exportImage(chart, type, fileName)` and `exportPDF(chart, fileName, orientation?, header?, footer?)` still work, but they are deprecated and only delegate to `exportChart`. They will be removed in a future major release. Generate new code with `exportChart`, and migrate existing calls:

```tsx
// Before (deprecated)
exportImage(chart, "PNG", "sales");
exportPDF(chart, "sales", PdfPageOrientation.Landscape);

// After
exportChart(chart, "PNG", "sales");
exportChart(chart, "PDF", "sales", PdfPageOrientation.Landscape);
```

## Do not use undocumented instance methods

Incorrect:

```tsx
chartRef.current?.export("PNG", "sales-chart");
chartRef.current?.print();
```

Use the exported static functions instead.

## Chart instance pattern

Keep a reference to the initialized chart. The chart forwards `IChart` through `ref`; its `element` is set once mounted, so the static functions are safe to call after the first render commits. There is no `onLoaded` event on the chart. Gate actions with a mounted state flag if the UI must disable them before the first render.

```tsx
import { useEffect, useRef, useState } from "react";
import {
  Chart,
  ChartSeries,
  ChartSeriesCollection,
  exportChart,
  print,
} from "@syncfusion/react-charts";
import type { ExportType, IChart } from "@syncfusion/react-charts";

const data = [
  { month: "Jan", sales: 35 },
  { month: "Feb", sales: 28 },
  { month: "Mar", sales: 34 },
];

export default function ExportableChart() {
  const chartRef = useRef<IChart | null>(null);
  const [ready, setReady] = useState(false);

  useEffect(() => {
    setReady(true);
  }, []);

  const handleExport = (type: ExportType): void => {
    if (chartRef.current) {
      exportChart(chartRef.current, type, "monthly-sales");
    }
  };

  const handlePrint = (): void => {
    if (chartRef.current) {
      print(chartRef.current);
    }
  };

  return (
    <section>
      <div className="chart-actions" role="group" aria-label="Chart output actions">
        <button type="button" onClick={handlePrint} disabled={!ready}>Print</button>
        <button type="button" onClick={() => handleExport("PNG")} disabled={!ready}>PNG</button>
        <button type="button" onClick={() => handleExport("SVG")} disabled={!ready}>SVG</button>
        <button type="button" onClick={() => handleExport("PDF")} disabled={!ready}>PDF</button>
        <button type="button" onClick={() => handleExport("XLSX")} disabled={!ready}>Excel</button>
        <button type="button" onClick={() => handleExport("CSV")} disabled={!ready}>CSV</button>
      </div>

      <Chart ref={chartRef} width="100%" height="420px">
        <ChartSeriesCollection>
          <ChartSeries
            dataSource={data}
            xField="month"
            yField="sales"
            type="Column"
            name="Sales"
          />
        </ChartSeriesCollection>
      </Chart>
    </section>
  );
}
```

## Image export

```tsx
const handleImageExport = (): void => {
  const chart = chartRef.current;
  if (!chart) {
    return;
  }
  exportChart(chart, "PNG", "quarterly-revenue");
};
```

Use `PNG` when text clarity matters and `SVG` when the image will be scaled or edited. Inspect exported text, thin gridlines, borders, and transparency against the intended destination background. `JPG` has no transparency, so the chart background is filled.

## PDF export

Use `"PDF"` for a document-oriented output. The orientation is a `PdfPageOrientation` value from `@syncfusion/pdf-export` (a dependency of the charts package). The header and footer are `{ content, fontSize?, x?, y? }` objects; `content` is required. Header font size defaults to `14`, footer to `12`, and `x`/`y` default to `10`.

```tsx
import { PdfPageOrientation } from "@syncfusion/pdf-export";
import { exportChart } from "@syncfusion/react-charts";

const handlePdfExport = (): void => {
  const chart = chartRef.current;
  if (!chart) {
    return;
  }

  exportChart(
    chart,
    "PDF",
    "quarterly-revenue",
    PdfPageOrientation.Landscape,
    { content: "Quarterly Revenue", fontSize: 14, x: 24, y: 10 },
    { content: "Generated from the analytics dashboard", fontSize: 9, x: 24, y: 10 },
  );
};
```

Do not use a `text` key for the header or footer. The field is `content`. The `orientation`, `header`, and `footer` arguments are ignored for every type other than `"PDF"`.

## Data export (Excel and CSV)

`"XLSX"` and `"CSV"` export the chart's **data**, not its appearance. The export preserves visible categories, series values, numeric values, missing cells, and the extra fields of specialized series (for example `high`/`low`/`open`/`close` for financial series). Use it to give users a "Download data" action next to the chart.

```tsx
const downloadData = (format: "XLSX" | "CSV"): void => {
  if (chartRef.current) {
    exportChart(chartRef.current, format, "quarterly-revenue");
  }
};
```

Data export works for both `Chart` and `PieChart`. Hidden series (toggled off through the legend) are not part of the visible data. Do not build a manual CSV string from `dataSource` when the user only wants the plotted data.

## Printing

Use the static `print` function with the initialized chart instance.

```tsx
const handlePrint = (): void => {
  const chart = chartRef.current;

  if (chart) {
    print(chart);
  }
};
```

The function opens the chart in a new browser window and triggers the browser print dialog. Browser popup and print policies may affect the behavior, so call it directly from a user action such as a button click.

## Pie-chart output

The same static functions accept an initialized pie-chart instance.

```tsx
import {
  PieChart,
  PieChartSeries,
  PieChartSeriesCollection,
  exportChart,
  print,
} from "@syncfusion/react-charts";
import type { IPieChart } from "@syncfusion/react-charts";

const pieRef = useRef<IPieChart | null>(null);

const exportPie = (): void => {
  if (pieRef.current) {
    exportChart(pieRef.current, "PNG", "market-share");
  }
};

const printPie = (): void => {
  if (pieRef.current) {
    print(pieRef.current);
  }
};

<PieChart ref={pieRef} width="100%" height="420px">
  <PieChartSeriesCollection>
    <PieChartSeries
      dataSource={data}
      xField="category"
      yField="value"
    />
  </PieChartSeriesCollection>
</PieChart>
```

Do not convert a pie chart to a Cartesian chart merely to export it.

## Output-oriented layout

Use a dedicated export size when the on-screen responsive size is not suitable for the output.

```tsx
const [outputMode, setOutputMode] =
  useState<"screen" | "export">("screen");

const chartWidth = outputMode === "export" ? "1200px" : "100%";
const chartHeight = outputMode === "export" ? "675px" : "420px";

<Chart width={chartWidth} height={chartHeight}>
  {/* chart children */}
</Chart>
```

If the chart must resize before export, wait for the resized render to finish before calling the export function. Do not switch size and export synchronously in the same tick unless the package guarantees that the export sees the updated layout.

## Print stylesheet

Use print CSS for the surrounding page when printing a dashboard section. The static chart print function may open only the chart, while normal browser printing includes page chrome and surrounding UI.

```css
@media print {
  .chart-actions,
  .app-navigation,
  .filters {
    display: none !important;
  }

  .chart-card {
    break-inside: avoid;
    border: 0;
    box-shadow: none;
    padding: 0;
  }

  .chart-canvas {
    width: 100%;
    height: 160mm;
  }
}
```

Use physical units only for print-specific layout. Test both portrait and landscape page settings.

## Prevent clipped content

Before export or print:

1. Give the chart enough height for its title, subtitle, legend, axes, and labels.
2. Increase chart margins when edge labels or annotations approach the boundary.
3. Use `edgeLabelPlacement="Shift"` for axis edge labels where appropriate.
4. Wrap or trim long labels deliberately.
5. Move a large legend to the bottom or enable paging.
6. Keep outside pie labels within the available canvas.
7. Avoid absolutely positioned HTML that sits outside the chart root.
8. Test the longest localized text.

```tsx
<Chart
  width="100%"
  height="520px"
  margin={{
    top: 20,
    right: 24,
    bottom: 24,
    left: 24,
  }}
>
  {/* chart children */}
</Chart>
```

## Static-output content

Tooltips and hover highlights are transient. Move information required in the exported artifact into visible elements such as:

- chart and axis titles
- legend entries
- selected data labels
- annotations
- striplines and their labels
- visible source or period notes outside the chart when printing the full page

Do not enable every data label merely because tooltips are absent from static output. Choose a readable subset or provide an accompanying table.

## Color and contrast

- Use colors that remain distinguishable on the target printer and background.
- Avoid very light gridlines that disappear in print.
- Add marker shapes, dash patterns, labels, or selection patterns as non-color cues.
- Test grayscale output if users may print without color.
- Avoid transparent fills whose appearance changes against a different export background.

## Fonts and localization

- Use fonts available in the browser environment where export occurs.
- Avoid relying on remotely loaded fonts that may not be ready when export starts.
- Test long translations and locale-specific number or date formats.
- Keep raw plotted values numeric and localize only display text.
- Verify that the exported PDF or image contains all required glyphs.

## Export controls

Use native buttons with clear labels and disabled states.

```tsx
<div role="group" aria-label="Export chart">
  <button type="button" onClick={handlePrint} disabled={!ready}>
    Print
  </button>
  <button type="button" onClick={() => handleExport("PNG")} disabled={!ready}>
    Download PNG
  </button>
  <button type="button" onClick={() => handleExport("PDF")} disabled={!ready}>
    Download PDF
  </button>
  <button type="button" onClick={() => handleExport("CSV")} disabled={!ready}>
    Download data (CSV)
  </button>
</div>
```

Report failures through an accessible status message rather than silently doing nothing.

```tsx
<p role="status" aria-live="polite">
  {exportStatus}
</p>
```

## Testing checklist

Test each required output independently:

- PNG or other requested image type
- XLSX/CSV data export opened in a spreadsheet application
- PDF in portrait and landscape as applicable
- browser print preview
- physical or virtual printer output
- light and dark application themes
- grayscale output
- narrow and wide chart dimensions
- longest supported labels and translations
- charts with and without legends
- empty, loading, and error states
- pie charts with outside labels

Inspect the downloaded file rather than checking only that a download occurred.

## Common errors

### Calling export before initialization

The static functions skip export when the supplied chart is absent or lacks a valid root element. Disable export controls until the chart is ready.

### Undocumented ref methods

Do not assume `chartRef.current.export()`, `.exportPdf()`, or `.print()` exists. Use the documented static functions.

### Clipped percentage-height chart

A chart using `height="100%"` needs a parent with a resolved height before export.

### Exporting hover-only information

Tooltips and hover states should not contain the only copy of essential values.

### Exporting the full dashboard unintentionally

Choose deliberately between the static chart `print` function and normal page printing with `window.print()` plus print CSS.

### Tiny exported text

Do not export a large logical chart into a very small output size. Use a dedicated output dimension and retest label density.

## Validation checklist

Before returning print or export code:

1. Use `exportChart` or `print` from the package's static API; do not generate the deprecated `exportImage` / `exportPDF`.
2. Pass a fully initialized `IChart` or `IPieChart` instance.
3. Do not invent imperative instance methods.
4. Trigger print or download from a user action.
5. Disable controls until the chart is ready.
6. Use a filename without an extension when required by the static API.
7. Select a supported `ExportType`: `SVG`, `PNG`, `JPG`, `PDF`, `XLSX`, or `CSV`.
8. Choose PDF orientation from the chart aspect ratio.
9. Use `PdfPageOrientation` from `@syncfusion/pdf-export` and `{ content, fontSize?, x?, y? }` for PDF header and footer.
10. Give the chart a predictable output size.
11. Wait for layout updates before exporting a resized chart.
12. Keep titles, legends, labels, and annotations inside the output bounds.
13. Do not rely on tooltips or hover states in static output.
14. Test long translations, fonts, and glyph coverage.
15. Test contrast, grayscale, and target backgrounds.
16. Use print CSS when printing surrounding page content.
17. Inspect each generated file and print preview.
18. Keep export layouts simple when quality is the priority.
19. Do not mix Cartesian, pie, EJ2, or custom export contracts.
20. Ensure every imported symbol is used.
21. Emit valid, unescaped TSX.
