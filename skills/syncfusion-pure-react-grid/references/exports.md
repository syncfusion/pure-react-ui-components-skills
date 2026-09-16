---
name: exports
description: Print, PDF Export, Excel Export hooks for the Syncfusion React Data Grid — `useGridPrint`, `useGridPdfExport`, `useGridExcelExport` with range, customRange, fileName, header/footer config, cell customization, before/after lifecycle, toolbar integration. Load when adding a print button, configuring PDF/Excel exports, customizing per-cell styles in the export, or fetching all data for remote-backed grids.
---

# Exports — print, PDF, Excel

The grid surfaces three hook-based exporters in `@syncfusion/react-grid`. Each hook:

- Accepts `{ gridRef, getAllData? }`.
- Returns `{ <name>, isExporting, ... }`.
- Tools up the matching toolbar button (`'Print'`, `'PdfExport'`, `'ExcelExport'`).

| Hook | Returns | Toolbar key |
|---|---|---|
| `useGridPrint` | `{ print, isPrinting }` | `'Print'` |
| `useGridPdfExport` | `{ pdfExport, isExporting }` | `'PdfExport'` |
| `useGridExcelExport` | `{ excelExport, isExporting, progress, error }` | `'ExcelExport'` |

Quick wiring:

```tsx
import { useRef } from 'react';
import { Grid, GridRef, useGridPrint, useGridPdfExport, useGridExcelExport } from '@syncfusion/react-grid';

function App() {
  const gridRef = useRef<GridRef>(null);

  const { print }      = useGridPrint({ gridRef });
  const { pdfExport }  = useGridPdfExport({ gridRef });
  const { excelExport }= useGridExcelExport({ gridRef });

  return (
    <>
      <button onClick={() => print({ range: 'All' })}>Print</button>
      <button onClick={() => pdfExport({ fileName: 'orders.pdf' })}>PDF</button>
      <button onClick={() => excelExport({ fileName: 'orders.xlsx' })}>Excel</button>

      <Grid ref={gridRef} dataSource={data} toolbar={['Print','PdfExport','ExcelExport']} ... />
    </>
  );
}
```

## Common options

```ts
{
  range: 'All' | 'CurrentPage' | 'Custom',     // default 'All'
  customRange?: { startRow: number; endRow: number },  // zero-based inclusive; required for 'Custom'
  fileName?: string,
  maxRowsWarningThreshold?: number,            // show a warning when export exceeds N rows
}
```

| `range` | Data exported |
|---|---|
| `'All'` | Entire dataset |
| `'CurrentPage'` | Active page only |
| `'Custom'` | Explicit `customRange` |

For remote-backed data with paging, provide `getAllData` to resolve the full dataset across pages — without it the grid only sees the current page.

```tsx
const { excelExport } = useGridExcelExport({
  gridRef,
  getAllData: async () => {
    const r = await fetch('/api/all');
    return r.json();
  },
});
```

## Print (`useGridPrint`)

```tsx
print({
  range: 'Custom',
  customRange: { startRow: 0, endRow: 99 },
  title: 'Sales Report — Q1 2026',
  onBeforePrint: ({ columns, dataSource, cancel }) => {
    /* modify or cancel */
  },
  onAfterPrint: () => {},
});
```

Lifecycle events: `onBeforePrint` (`{ columns, dataSource, cancel }`) and `onAfterPrint`. The print strips filter icons, pager, and sorting indicators automatically.

For batch printing (large datasets), split into chunks of ~1000 rows.

## PDF export (`useGridPdfExport`)

```tsx
pdfExport({
  range: 'All',
  fileName: 'orders.pdf',
  pageOrientation: 'Landscape',             // 'Portrait' | 'Landscape'
  isBlob: true,                             // surface blobData via onAfterPdfExport
  columns: [{ field: 'OrderDate', format: 'yMd', width: 100 }],
  header: pdfHeader,
  footer: pdfFooter,
  onPdfCellCustomize: (args: PdfCellCustomizeArgs) => {
    args.style = args.style || {};
    if (args.column.field === 'Status' && args.value === 'Approved') {
      args.style.textBrushColor = [46, 125, 50];
      args.style.bold = true;
    }
  },
  onBeforePdfExport: (ev) => { /* … */ },
  onAfterPdfExport: (ev) => /* ev.promise resolves to { blobData } when isBlob=true */,
});
```

### `PdfHeader` / `PdfFooter`

```ts
const pdfHeader: PdfHeader = {
  fromTop: 10,
  height: 30,
  contents: [
    {
      type: 'Text',                       // 'Text' | 'PageNumber' | 'Date' | 'Image'
      value: 'Restaurant Order Tracker',
      position: { x: 0, y: 5 },
      style: { fontSize: 14, bold: true },
    },
  ],
};

const pdfFooter: PdfFooter = {
  fromBottom: 10,
  height: 20,
  contents: [
    {
      type: 'PageNumber',
      format: 'Page {$current} of {$total}',
      position: { x: 0, y: 0 },
      style: { fontSize: 10 },
    },
  ],
};
```

`PdfCellCustomizeArgs`: `{ data, column, value, style }`. Style includes `textBrushColor: [r, g, b]` (e.g., `[46, 125, 50]` for green), `bold`, `italic`, `background`, `fontSize`.

For remote grouped exports, register `GroupModule`, `FilterModule`, `PagerModule`, `ToolbarModule`.

When `isBlob: true`, `onAfterPdfExport.safePromise` (a Promise on the event payload) resolves to `{ blobData: Blob }`. Read it via:

```ts
onAfterPdfExport: (ev) => {
  if (ev.promise) {
    ev.promise.then(result => {
      // result.blobData is a Blob
    });
  }
}
```

`pageOrientation` default: `'Portrait'`.

## Excel export (`useGridExcelExport`)

```tsx
import { CellStyle } from '@syncfusion/excel-export';

const titleStyle = new CellStyle();
titleStyle.bold = true;
titleStyle.fontSize = 16;

excelExport({
  fileName: 'orders.xlsx',
  range: 'All',
  isBlob: true,
  header: {
    headerRows: 2,
    rows: [
      { cells: [{ colSpan: 6, value: 'Restaurant Order Tracker', style: titleStyle }] },
      { cells: [{ colSpan: 6, value: 'Prepared for internal review', style: subtitleStyle }] },
    ],
  },
  footer: {
    footerRows: 1,
    rows: [{ cells: [{ colSpan: 6, value: 'End of report', style: footerStyle }] }],
  },
  onExcelCellCustomize: (args: ExcelCellCustomizeArgs<T>) => {
    args.style = args.style || new CellStyle();
    if (args.column.field === 'Amount' && args.data.Priority === 'High') {
      args.style.bold = true;
    }
    if (args.column.field === 'Profile' && args.value) {
      args.hyperLink = { target: args.value as string, displayText: 'View profile' };
    }
  },
  onBeforeExcelExport: (ev) => { /* … */ },
  onAfterExcelExport: (ev) => /* ev.promise resolves to { blobData } when isBlob=true */,
});
```

### Header / footer cell shape

```ts
{
  colSpan: number;
  rowSpan?: number;
  value: string | number;
  hyperlink?: { target: string; displayText: string };
  style: CellStyle;
}
```

### `CellStyle` (from `@syncfusion/excel-export`)

| Field | Type | Notes |
|---|---|---|
| `bold` | `boolean` | |
| `italic` | `boolean` | |
| `fontSize` | `number` | |
| `fontColor` | `string` | `'#RRGGBB'` |
| `backgroundColor` / `fill` | `string` | `'#RRGGBB'` |
| `borders` | border config | top/right/bottom/left per-cell |
| `alignment` | `HorizontalAlignment \| VerticalAlignment` | |
| `numberFormat` | `string` | Excel number format strings |

### `ExcelCellCustomizeArgs<T>`

```ts
{
  data: T;
  column: ColumnProps;
  value: any;
  style: CellStyle;             // mutate in place
  hyperLink?: { target: string; displayText: string };
}
```

### Toolbar

`toolbar={['ExcelExport']}` renders the toolbar button. `ToolbarClickEvent.item === 'ExcelExport'` fires on click.

## Common patterns

| Need | Configuration |
|---|---|
| Quick PDF | `pdfExport({ fileName: 'report.pdf' })` |
| Print the current page only | `print({ range: 'CurrentPage' })` |
| Restrict rows | `excelExport({ range: 'Custom', customRange: { startRow: 0, endRow: 49 } })` |
| Excel with branding | `header: { headerRows: 1, rows: [{ cells: [{ colSpan, value, style }] }] }` |
| Color rows in PDF | `onPdfCellCustomize` mutates `args.style.textBrushColor` |
| Hyperlinks in Excel | set `args.hyperLink = { target, displayText }` |
| Get blob for downstream upload | set `isBlob: true`, read `ev.promise` in `onAfterXxxExport` |

## Constraints & guardrails

- **Toolbar integration**: a grid with `'PdfExport'` in toolbar but no `useGridPdfExport()` in scope will appear functional but the click is a no-op. Ensure the hook is mounted in the same component tree.
- **`getAllData` is the only way to export remote paged data** — without it, only the current page is exported. This is a deployment gap if not configured.
- **Custom formats** like `format: 'C2'` round-trip correctly through PDF/Excel; ensure the cell's `format` is set.
- **Templates**: cell templates render as raw values in exports (the original data); set `disableHtmlEncode` only when the result is fully trusted content.
- **Grouping + filters** are honored — exports respect current filter/sort state.
- **High-risk operations**: exporting a 100k row dataset to Excel in the browser can OOM the tab. Use `maxRowsWarningThreshold` to gate, or split into batches server-side.
- **Sensitive data**: PDFs and Excel files download to the user's machine — handle exports with the same care as a remote download (confirm intent before triggering bulk exports).
- **Integration: groups + aggregates in exports**: register `GroupModule`, `AggregateModule`, `PagerModule`, `ToolbarModule` if the export needs any of them.