# Keyboard Navigation Reference

Use keyboard-friendly chart patterns whenever a Pure React Chart is interactive or must support accessible navigation. Configure focus and accessible names through the documented accessibility objects, preserve a logical page-level focus order, and provide non-pointer alternatives for essential interactions.

## Core rules

- Keep the chart in the natural document focus order unless there is a specific reason to exclude it.
- Provide a concise, meaningful `ariaLabel` for the chart container.
- Keep a clearly visible focus outline.
- Do not make hover the only way to obtain essential values or instructions.
- Do not make drag, mouse wheel, or pinch the only way to operate important zoom behavior.
- Use accessible native buttons for application-level chart actions.
- Test the installed Pure React package rather than copying keyboard shortcuts from EJ2 or another chart library.
- Verify that users can enter, operate, and leave the chart without a keyboard trap.

## Focusable chart container

Configure the root chart through its `accessibility` object.

```tsx
import { Chart } from "@syncfusion/react-charts";

<Chart
  accessibility={{
    ariaLabel:
      "Monthly sales chart. A data table follows the chart.",
    role: "region",
    focusable: true,
    tabIndex: 0,
  }}
/>
```

The chart accessibility API provides `ariaLabel`, `role`, `focusable`, and `tabIndex`. Use `tabIndex: 0` for the natural tab order. Use `-1` only when focus will be moved to the chart programmatically and skipping normal keyboard navigation is intentional.

Do not use positive `tabIndex` values to force the chart ahead of other page elements.

## Visible focus outline

Use `focusOutline` on `Chart`.

```tsx
<Chart
  accessibility={{
    ariaLabel: "Monthly sales chart",
    role: "region",
    focusable: true,
    tabIndex: 0,
  }}
  focusOutline={{
    color: "#005FCC",
    width: 2,
    offset: 2,
  }}
/>
```

`FocusOutlineProps` documents `color`, `width`, and `offset`. The default width is `1.5` and the default offset is `0`.

Choose a focus color with sufficient contrast against the chart and surrounding page. Do not remove visible focus unless an equally visible replacement is provided.

## Preserve surrounding focus order

Place controls in a logical DOM order around the chart.

```tsx
<section aria-labelledby="sales-heading">
  <h2 id="sales-heading">Monthly sales</h2>

  <div>
    <button type="button" onClick={showPreviousPeriod}>
      Previous period
    </button>
    <button type="button" onClick={showNextPeriod}>
      Next period
    </button>
  </div>

  <Chart
    accessibility={{
      ariaLabel: "Monthly sales chart",
      role: "region",
      focusable: true,
      tabIndex: 0,
    }}
  >
    {/* chart children */}
  </Chart>

  <button type="button" onClick={downloadData}>
    Download data
  </button>
</section>
```

The visual order should match the keyboard order. Avoid CSS reordering that makes focus move unpredictably across the page.

## Keyboard-operable external controls

Use native interactive elements for filters, period changes, drill-down, reset, and other application actions.

```tsx
<button
  type="button"
  onClick={() => setSelectedIndexes([])}
>
  Clear selected points
</button>
```

Do not make a `div` clickable without also implementing the complete keyboard, focus, role, and state behavior expected of a button.

## Tooltips without hover-only dependence

A tooltip may supplement the chart, but essential values must remain available without pointer hover.

```tsx
<ChartTooltip
  enable={true}
  shared={true}
/>
```

Also provide one or more of:

- keyboard-accessible point navigation supported by the installed chart version
- visible labels for important values
- a concise text summary
- an accessible data table
- keyboard-operable detail controls

```tsx
<table>
  <caption>Monthly sales values</caption>
  <thead>
    <tr>
      <th scope="col">Month</th>
      <th scope="col">Sales</th>
    </tr>
  </thead>
  <tbody>
    {data.map((item) => (
      <tr key={item.month}>
        <th scope="row">{item.month}</th>
        <td>{item.sales}</td>
      </tr>
    ))}
  </tbody>
</table>
```

Do not claim that hidden data labels automatically appear on keyboard focus. Data labels and tooltips are separate features.

## Selection without pointer-only dependence

Use `ChartSelection` for persistent visual selection, but provide an external keyboard-operable action when selection is essential to the workflow.

```tsx
const [selectedIndexes, setSelectedIndexes] =
  useState<ChartIndexesProps[]>([]);

const selectFirstPoint = (): void => {
  setSelectedIndexes([
    { seriesIndex: 0, pointIndex: 0 },
  ]);
};

<button type="button" onClick={selectFirstPoint}>
  Select first point
</button>

<Chart>
  <ChartSelection
    mode="Point"
    selectedDataIndexes={selectedIndexes}
    pattern="Crosshatch"
  />
  {/* chart children */}
</Chart>
```

`selectedDataIndexes` requires objects with `seriesIndex` and `pointIndex`. Use a pattern or another non-color cue so the selected state does not rely only on color.

## Legend keyboard accessibility

Configure legend accessibility through the legend's `accessibility` object.

```tsx
<ChartLegend
  visible={true}
  toggleVisibility={true}
  accessibility={{
    ariaLabel: "Chart series legend",
    focusable: true,
    tabIndex: 0,
  }}
/>
```

The legend API exposes `accessibility`, and `toggleVisibility` defaults to `true`. Test that legend items can be reached and operated in the installed package and target browser and screen-reader combinations.

If the built-in legend does not meet the required workflow, create a separate group of native checkbox or button controls and render series conditionally.

```tsx
<fieldset>
  <legend>Visible series</legend>

  {series.map((item) => (
    <label key={item.id}>
      <input
        type="checkbox"
        checked={item.visible}
        onChange={() => toggleSeries(item.id)}
      />
      {item.name}
    </label>
  ))}
</fieldset>
```

## Keyboard-friendly zoom

Selection zoom, mouse-wheel zoom, pinch zoom, and direct panning are pointer-oriented interactions. When zooming is important, expose the built-in toolbar and configure accessibility for zoom-related UI.

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
  accessibility={{
    ariaLabel: "Chart zoom controls",
    focusable: true,
    tabIndex: 0,
  }}
/>
```

The zoom settings API documents accessibility configuration and toolbar items for Zoom In, Zoom Out, Pan, and Reset. Test actual toolbar keyboard operation with the installed package.

Do not invent keyboard shortcuts or undocumented imperative methods such as `chartRef.current.zoom()` and `resetZoom()`.

## Point drill-down

Use the documented `onPointClick` event for point details, and provide a parallel keyboard-operable list or table when drill-down is a critical task.

```tsx
import type { PointClickEvent } from "@syncfusion/react-charts";

const openPointDetails = (args: PointClickEvent): void => {
  const point = data[args.pointIndex];

  if (point) {
    setSelectedPoint(point);
  }
};

<Chart onPointClick={openPointDetails}>
  {/* chart children */}
</Chart>
```

A pointer event alone is not proof that drill-down is keyboard accessible. Test built-in point navigation and provide an equivalent external control if the workflow requires guaranteed keyboard access.

## Do not publish unverified shortcuts

Keyboard commands may vary by package version, chart type, enabled features, browser, and focused element. Do not document shortcuts such as `Alt + J`, arrow-key navigation, `Ctrl + Plus`, or `R` unless those commands are present in the current Pure React documentation and have been tested against the installed version.

Use behavior-based guidance instead:

1. Tab to the chart and its interactive controls.
2. Confirm the focus outline is visible.
3. Operate points, legend items, selection, and zoom controls using their supported keys.
4. Dismiss transient content when supported.
5. Move focus out of the chart in both directions.

## Complete keyboard-friendly example

```tsx
import { useState } from "react";
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartLegend,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSelection,
  ChartSeries,
  ChartSeriesCollection,
  ChartTitle,
  ChartTooltip,
  ChartZoomSettings,
} from "@syncfusion/react-charts";
import type { ChartIndexesProps } from "@syncfusion/react-charts";

const data = [
  { month: "Jan", sales: 100 },
  { month: "Feb", sales: 120 },
  { month: "Mar", sales: 110 },
  { month: "Apr", sales: 145 },
];

export default function KeyboardFriendlyChart() {
  const [selectedIndexes, setSelectedIndexes] =
    useState<ChartIndexesProps[]>([]);

  const selectHighestPoint = (): void => {
    const highestIndex = data.reduce(
      (bestIndex, item, index, items) =>
        item.sales > items[bestIndex].sales
          ? index
          : bestIndex,
      0,
    );

    setSelectedIndexes([
      { seriesIndex: 0, pointIndex: highestIndex },
    ]);
  };

  return (
    <section aria-labelledby="sales-heading">
      <h2 id="sales-heading">Monthly sales</h2>
      <p>
        Sales values from January through April. A data table
        follows the chart.
      </p>

      <div>
        <button type="button" onClick={selectHighestPoint}>
          Select highest value
        </button>
        <button
          type="button"
          onClick={() => setSelectedIndexes([])}
        >
          Clear selection
        </button>
      </div>

      <Chart
        accessibility={{
          ariaLabel:
            "Monthly sales chart from January through April. A data table follows.",
          role: "region",
          focusable: true,
          tabIndex: 0,
        }}
        focusOutline={{
          color: "#005FCC",
          width: 2,
          offset: 2,
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

        <ChartLegend
          visible={true}
          accessibility={{
            ariaLabel: "Chart series legend",
            focusable: true,
            tabIndex: 0,
          }}
        />

        <ChartTooltip
          enable={true}
          format="${series.name}: ${point.y}"
        />

        <ChartSelection
          mode="Point"
          selectedDataIndexes={selectedIndexes}
          pattern="Crosshatch"
        />

        <ChartZoomSettings
          selectionZoom={true}
          mode="X"
          toolbar={{
            visible: true,
            items: ["ZoomIn", "ZoomOut", "Pan", "Reset"],
          }}
          accessibility={{
            ariaLabel: "Chart zoom controls",
            focusable: true,
            tabIndex: 0,
          }}
        />

        <ChartSeriesCollection>
          <ChartSeries
            dataSource={data}
            xField="month"
            yField="sales"
            type="Column"
            name="Sales"
            accessibility={{
              ariaLabel: "Monthly sales series",
              descriptionFormat:
                "${series.name}, ${point.x}: ${point.y}",
              focusable: true,
              role: "img",
              tabIndex: 0,
            }}
          />
        </ChartSeriesCollection>
      </Chart>

      <table>
        <caption>Monthly sales values</caption>
        <thead>
          <tr>
            <th scope="col">Month</th>
            <th scope="col">Sales</th>
          </tr>
        </thead>
        <tbody>
          {data.map((item) => (
            <tr key={item.month}>
              <th scope="row">{item.month}</th>
              <td>{item.sales}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </section>
  );
}
```

## Manual keyboard test

Test in both forward and reverse focus order:

1. Press Tab from the page content before the chart.
2. Confirm each external control receives visible focus.
3. Confirm the chart container and enabled chart controls are reachable.
4. Operate the legend, point navigation, selection, and zoom toolbar using the keys supported by the installed version.
5. Verify that tooltip or equivalent values are available without hover.
6. Verify that focus does not disappear behind chart content.
7. Press Tab to move to the content after the chart.
8. Repeat with Shift+Tab.
9. Test at browser zoom levels used by the product's accessibility target.
10. Test with at least one screen reader used by the target audience.

## Common errors

### Removing the chart from the tab order

Incorrect for an interactive chart:

```tsx
<Chart
  accessibility={{
    focusable: false,
    tabIndex: -1,
  }}
/>
```

Use a focusable chart or provide fully equivalent external controls.

### Positive tab indexes

Avoid `tabIndex` values greater than zero because they override natural document order and are difficult to maintain.

### Hover-only values

A tooltip alone is not sufficient when its content is essential and cannot be reached by keyboard. Provide labels, a summary, or a synchronized data table.

### Pointer-only zoom

Do not expose only wheel, drag, or pinch zoom. Show and test the zoom toolbar when keyboard zoom is required.

### Unsupported shortcuts

Do not copy shortcut tables from EJ2 or another product version without current Pure React verification.

## Validation checklist

Before returning a keyboard-navigation implementation:

1. Provide a meaningful root `accessibility.ariaLabel`.
2. Keep the chart focusable when it is interactive.
3. Use `tabIndex: 0` for natural focus order.
4. Avoid positive `tabIndex` values.
5. Configure a visible, high-contrast `focusOutline`.
6. Keep visual and DOM order aligned.
7. Use native buttons, inputs, and checkboxes for external controls.
8. Do not rely on hover for essential information.
9. Provide a summary or accessible table when needed.
10. Provide keyboard-operable selection actions for critical workflows.
11. Configure legend accessibility through `ChartLegend.accessibility`.
12. Expose and test the zoom toolbar when keyboard zoom is required.
13. Use documented chart events and exact event types.
14. Do not invent keyboard shortcuts or imperative chart methods.
15. Test Tab and Shift+Tab entry and exit.
16. Test chart interactions with the installed package version.
17. Test with a screen reader and browser zoom.
18. Verify that no keyboard trap is created.
19. Do not mix EJ2 keyboard behavior with Pure React guidance.
20. Ensure every imported symbol is used.
21. Emit valid, unescaped TSX.
