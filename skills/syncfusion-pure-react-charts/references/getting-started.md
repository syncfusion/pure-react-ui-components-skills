# Getting Started Reference

Use this reference for the minimum Pure React Chart setup. Start with one local data source and one series, confirm the chart renders, then add titles, axes, markers, labels, tooltips, or interactions only as needed.

## Prerequisites

The current Pure React Chart getting-started guide requires Node.js 20 or later and supports development on Windows, macOS, and Linux.

## Install packages

Install the chart package:

```bash
npm install @syncfusion/react-charts
```

`@syncfusion/react-charts` installs its required dependencies. Install `@syncfusion/react-base` separately only when the application directly imports base utilities or the project setup requires an explicit dependency.

```bash
npm install @syncfusion/react-base
```

Do not mix `@syncfusion/react-charts` with the older `@syncfusion/ej2-react-charts` package in the same sample.

## Import the theme stylesheet

Import the required Syncfusion base theme stylesheet once in the application stylesheet or entry point.

```css
@import "@syncfusion/react-base/styles/material.css";
```

The official guide also shows a relative `node_modules` import. Prefer the package-style import when it is supported by the project's bundler.

Do not import the same theme more than once or combine unrelated Syncfusion themes in one page.

## Minimum chart

A chart requires `Chart`, `ChartSeriesCollection`, and at least one `ChartSeries`.

```tsx
import {
  Chart,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { year: 2020, value: 4.8 },
  { year: 2021, value: 7.2 },
  { year: 2022, value: 10.4 },
  { year: 2023, value: 13.8 },
  { year: 2024, value: 17.0 },
];

export default function App() {
  return (
    <Chart>
      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="year"
          yField="value"
          type="Line"
          name="Value"
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

Always provide mappings that match the data object keys.

## Recommended starter chart

Use a Category X-axis when the data contains labels such as month names.

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
} from "@syncfusion/react-charts";

const data = [
  { month: "Jan", sales: 35 },
  { month: "Feb", sales: 32 },
  { month: "Mar", sales: 34 },
  { month: "Apr", sales: 38 },
  { month: "May", sales: 41 },
  { month: "Jun", sales: 39 },
];

export default function App() {
  return (
    <Chart
      accessibility={{
        ariaLabel: "Monthly sales from January through June",
        role: "img",
        focusable: true,
        tabIndex: 0,
      }}
    >
      <ChartTitle text="Monthly sales" />

      <ChartPrimaryXAxis valueType="Category">
        <ChartAxisTitle text="Month" />
        <ChartAxisLabel edgeLabelPlacement="Shift" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double" minimum={0}>
        <ChartAxisTitle text="Sales" />
        <ChartAxisLabel format="{value}" />
      </ChartPrimaryYAxis>

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
  );
}
```

## Add markers

Place `ChartMarker` inside the series.

```tsx
<ChartSeries
  dataSource={data}
  xField="month"
  yField="sales"
  type="Line"
>
  <ChartMarker
    visible={true}
    shape="Circle"
    width={7}
    height={7}
  />
</ChartSeries>
```

Markers are hidden by default. Use direct `width` and `height` properties.

## Add data labels

Place `ChartDataLabel` inside `ChartMarker` for marker-based Cartesian series.

```tsx
<ChartSeries
  dataSource={data}
  xField="month"
  yField="sales"
  type="Line"
>
  <ChartMarker visible={true}>
    <ChartDataLabel
      visible={true}
      position="Top"
      format="{value}"
    />
  </ChartMarker>
</ChartSeries>
```

Data labels are hidden by default. Avoid enabling every label when the chart is dense.

## Add a tooltip

Place `ChartTooltip` directly inside `Chart`.

```tsx
<Chart>
  <ChartTooltip enable={true} />

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="month"
      yField="sales"
      type="Line"
    />
  </ChartSeriesCollection>
</Chart>
```

Tooltips are disabled by default.

## Component ownership

Use this basic hierarchy:

```text
Chart
├── ChartTitle
├── ChartPrimaryXAxis
│   ├── ChartAxisTitle
│   └── ChartAxisLabel
├── ChartPrimaryYAxis
│   ├── ChartAxisTitle
│   └── ChartAxisLabel
├── ChartTooltip
└── ChartSeriesCollection
    └── ChartSeries
        └── ChartMarker
            └── ChartDataLabel
```

Do not place `ChartSeries` directly under `Chart`, and do not move axis or series child components to the root.

## Run the project

For a Vite application:

```bash
npm run dev
```

Open the local URL printed by Vite and verify that the chart, axes, and series render without console errors.

## Good starter pattern

1. Start with a small local array.
2. Render a single `Line` or `Column` series.
3. Add explicit `xField` and `yField` mappings.
4. Match the X-axis `valueType` to the data.
5. Add a concise title and axis titles.
6. Confirm the theme stylesheet is loaded.
7. Add markers, labels, tooltips, legends, and interactions one at a time.
8. Add remote data only after local binding works.

## Common errors

### Missing collection wrapper

Incorrect:

```tsx
<Chart>
  <ChartSeries dataSource={data} />
</Chart>
```

Correct:

```tsx
<Chart>
  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="x"
      yField="y"
      type="Line"
    />
  </ChartSeriesCollection>
</Chart>
```

### Missing field mappings

Incorrect:

```tsx
<ChartSeries dataSource={data} type="Column" />
```

Correct:

```tsx
<ChartSeries
  dataSource={data}
  xField="month"
  yField="sales"
  type="Column"
/>
```

### Axis and data mismatch

Use `Category` for text categories, `Double` for numeric values, `DateTime` for real dates, and `Logarithmic` for valid positive logarithmic data.

### Theme not loaded

If the chart appears unstyled, verify that the Syncfusion base theme stylesheet is imported once and that the import path resolves through the bundler.

### Mixing EJ2 and Pure React

Do not use `ChartComponent`, `SeriesDirective`, `Inject`, `xName`, or `yName` in a Pure React sample. Use `Chart`, `ChartSeriesCollection`, `ChartSeries`, `xField`, and `yField`.

## Validation checklist

Before returning a starter implementation:

1. Import components from `@syncfusion/react-charts`.
2. Import one compatible Syncfusion base theme stylesheet.
3. Place every `ChartSeries` inside `ChartSeriesCollection`.
4. Bind a valid local array through `dataSource`.
5. Map `xField` and `yField` to real data keys.
6. Set a valid series `type`.
7. Match the X-axis `valueType` to the bound values.
8. Place axis titles and labels inside their owning axes.
9. Place markers and data labels inside their owning series hierarchy.
10. Place tooltips directly inside `Chart`.
11. Add a meaningful accessibility label.
12. Avoid unused imports and unnecessary features.
13. Do not mix EJ2 and Pure React APIs.
14. Emit valid, unescaped TSX.
