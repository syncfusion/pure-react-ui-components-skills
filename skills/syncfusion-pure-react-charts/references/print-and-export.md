# Print and Export Reference

Use the package-provided static chart functions when a Pure React Cartesian chart or pie chart must print or export. Prepare the chart for the target page or image size first, then call `print`, `exportImage`, or `exportPDF` with a fully initialized chart instance.

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

- `exportImage(chart, type, fileName)` for image export
- `exportPDF(chart, fileName, orientation?, header?, footer?)` for PDF export
- `print(chart)` to open a print window and trigger the browser print dialog

These functions accept either a Cartesian chart instance or a pie-chart instance. The chart must be fully initialized and contain a valid root element.

## Do not use undocumented instance methods

Incorrect:

```tsx
chartRef.current?.export("PNG", "sales-chart");
chartRef.current?.print();
```

Use the exported static functions documented for the package version instead.

## Chart instance pattern

Keep a reference to the initialized chart. The chart forwards `IChart` through `ref`; its `element` is set once mounted, so the static functions are safe to call after the first render commits. There is no `onLoaded` event on the chart — gate actions with a mounted state flag if the UI must disable them before the first render.

```tsx
import { useEffect, useRef, useState } from "react";
import {
  Chart,
  ChartSeries,
  ChartSeriesCollection,
  exportImage,
  exportPDF,
  print,
} from "@syncfusion/react-charts";
import type { IChart } from "@syncfusion/react-charts";

export default function ExportableChart() {
  const chartRef = useRef<IChart | null>(null);
  const [ready, setReady] = useState(false);

  useEffect(() => {
    setReady(true);
  }, []);

  const handlePrint = (): void => {
    if (chartRef.current) {
      print(chartRef.current);
    }
  };

  const handlePngExport = (): void => {
    if (chartRef.current) {
      exportImage(chartRef.current, "PNG", "monthly-sales");
    }
  };

  const handlePdfExport = (): void => {
    if (chartRef.current) {
      exportPDF(chartRef.current, "monthly-sales", "Landscape");
    }
  };

  return (
    <section>
      <div className="chart-actions" aria-label="Chart output actions">
        <button type="button" onClick={handlePrint} disabled={!ready}>
          Print chart
        </button>
        <button type="button" onClick={handlePngExport} disabled={!ready}>
          Export PNG
        </button>
        <button type="button" onClick={handlePdfExport} disabled={!ready}>
          Export PDF
        </button>
      </div>

      <Chart
        ref={chartRef}
        width="100%"
        height="420px"
      >
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

The static functions themselves require an initialized `IChart` or `IPieChart` with a valid element; they return early if `element` is missing.

## Image export

Use `exportImage` with the desired `ExportType` and a filename without an extension.

```tsx
const handleImageExport = (): void => {
  const chart = chartRef.current;

  if (!chart) {
    return;
  }

  exportImage(chart, "PNG", "quarterly-revenue");
};
```

Use a lossless raster format when text clarity is important. Inspect exported text, thin gridlines, borders, and transparency against the intended destination background.

Do not append an extension unless the current API explicitly requires it. The documented `fileName` parameter is the name without the extension.

## PDF export

Use `exportPDF` for a document-oriented output. Select portrait or landscape according to the chart aspect ratio.

```tsx
const handlePdfExport = (): void => {
  const chart = chartRef.current;

  if (!chart) {
    return;
  }

  exportPDF(
    chart,
    "quarterly-revenue",
    "Landscape",
    {
      text: "Quarterly Revenue",
      fontSize: 14,
      x: 24,
      y: 18,
    },
    {
      text: "Generated from the analytics dashboard",
      fontSize: 9,
      x: 24,
      y: 18,
    },
  );
};
```

The header and footer arguments are optional. Verify `PdfPageOrientation` and `IPdfTextArgs` values against the installed version before finalizing typed production code.

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
  exportImage,
  print,
} from "@syncfusion/react-charts";
import type { IPieChart } from "@syncfusion/react-charts";

const pieRef = useRef<IPieChart | null>(null);

const exportPie = (): void => {
  if (pieRef.current) {
    exportImage(pieRef.current, "PNG", "market-share");
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
  <button type="button" onClick={handlePngExport} disabled={!ready}>
    Download PNG
  </button>
  <button type="button" onClick={handlePdfExport} disabled={!ready}>
    Download PDF
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

1. Use `print`, `exportImage`, or `exportPDF` from the package's documented static API.
2. Pass a fully initialized `IChart` or `IPieChart` instance.
3. Do not invent imperative instance methods.
4. Trigger print or download from a user action.
5. Disable controls until the chart is ready.
6. Use a filename without an extension when required by the static API.
7. Select a supported image export type.
8. Choose PDF orientation from the chart aspect ratio.
9. Verify optional PDF header and footer arguments against the installed version.
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
