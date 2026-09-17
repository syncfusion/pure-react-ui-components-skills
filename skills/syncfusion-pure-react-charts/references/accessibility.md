# Accessibility Reference

Use the Pure React Chart accessibility APIs together with accessible data presentation, keyboard testing, screen-reader testing, sufficient contrast, responsive layout, and non-color cues. Do not treat component configuration alone as proof that an application is WCAG 2.2 compliant.

## Accessibility support

The official Pure React Chart documentation identifies support for accessibility areas including:

- WCAG 2.2
- Section 508
- WAI-ARIA roles
- screen readers
- keyboard navigation
- color contrast
- right-to-left content
- mobile devices
- axe-based validation

Actual conformance depends on the complete application, chart content, colors, labels, surrounding controls, and testing process.

## Root chart accessibility

Configure the chart container through the `accessibility` object.

```tsx
import { Chart } from "@syncfusion/react-charts";

<Chart
  accessibility={{
    ariaLabel: "Monthly sales from January through December 2026",
    role: "img",
    focusable: true,
    tabIndex: 0,
  }}
/>
```

The root `Chart` API documents these defaults:

```tsx
{
  ariaLabel: "",
  focusable: true,
  role: "",
  tabIndex: 0,
}
```

Use:

- `ariaLabel` to provide a concise text alternative
- `role` to identify the chart container appropriately
- `focusable` to control keyboard focus
- `tabIndex` to control focus order

Do not repeat the visible title word for word when a more useful summary can describe the chart's purpose, scope, units, and time range.

## Choosing an ARIA role

The Chart accessibility documentation describes roles including `img`, `region`, and `text` within chart output.

Use a chart-level role that matches the experience:

```tsx
<Chart
  accessibility={{
    ariaLabel: "Quarterly revenue chart with four columns",
    role: "img",
    focusable: true,
    tabIndex: 0,
  }}
/>
```

Use `region` only when treating the chart as a navigable landmark is useful in the surrounding page structure.

```tsx
<Chart
  accessibility={{
    ariaLabel: "Interactive regional sales analysis",
    role: "region",
    focusable: true,
    tabIndex: 0,
  }}
/>
```

Avoid adding unnecessary landmarks when a page already contains many regions.

## Writing an effective ariaLabel

Include the information users need to understand the chart without seeing it:

- chart purpose
- primary metric
- unit
- relevant categories or time range
- important interaction instructions, when needed

```tsx
<Chart
  accessibility={{
    ariaLabel:
      "Monthly revenue in US dollars for 2026. Use keyboard navigation to inspect data points.",
    role: "region",
    focusable: true,
    tabIndex: 0,
  }}
/>
```

Avoid labels such as `Chart`, `Data chart`, or `Sales graphic` because they do not communicate enough context.

## Focus outline

Use `focusOutline` on `Chart` to make keyboard focus visible.

```tsx
<Chart
  accessibility={{
    ariaLabel: "Monthly sales performance",
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

`FocusOutlineProps` documents:

- `color`: focus border color; default empty
- `width`: focus border width; default `1.5`
- `offset`: focus border margin; default `0`

Choose a focus color that remains visible against adjacent chart and page backgrounds. Do not remove the focus outline unless an equally visible alternative is provided.

## Series accessibility

Configure series accessibility through `ChartSeries.accessibility`, not a direct `ariaLabel` prop.

```tsx
<ChartSeries
  dataSource={data}
  xField="month"
  yField="sales"
  type="Column"
  name="Sales"
  accessibility={{
    ariaLabel: "Monthly sales series",
    descriptionFormat: "${series.name}, ${point.x}: ${point.y}",
    focusable: true,
    role: "img",
    tabIndex: 0,
  }}
/>
```

The series API documents accessibility settings with defaults including:

```tsx
{
  ariaLabel: "",
  descriptionFormat: "",
  focusable: true,
  role: "",
  tabIndex: 0,
}
```

Use a meaningful series `name` even when the legend is hidden. Keep the description format concise enough for repeated point navigation.

Incorrect:

```tsx
<ChartSeries
  ariaLabel="Monthly sales series"
/>
```

Correct:

```tsx
<ChartSeries
  accessibility={{
    ariaLabel: "Monthly sales series",
  }}
/>
```

## Legend accessibility

Configure legend accessibility through `ChartLegend.accessibility`.

```tsx
<ChartLegend
  visible={true}
  position="Bottom"
  accessibility={{
    ariaLabel: "Chart series legend",
    focusable: true,
    tabIndex: 0,
  }}
/>
```

The legend API documents an `accessibility` object and interactive visibility toggling. `toggleVisibility` defaults to `true`.

Incorrect:

```tsx
<ChartLegend ariaLabel="Series legend" />
```

Correct:

```tsx
<ChartLegend
  accessibility={{
    ariaLabel: "Series legend",
  }}
/>
```

If legend toggling changes what the chart displays, test the interaction with keyboard and screen-reader users.

## Titles and visible descriptions

Provide a clear visible heading near the chart. Use `ChartTitle` for the chart title and normal HTML for longer explanations or summaries.

```tsx
<section aria-labelledby="sales-chart-heading">
  <h2 id="sales-chart-heading">2026 monthly sales</h2>
  <p id="sales-chart-summary">
    Sales increased overall, with the highest value in December.
  </p>

  <Chart
    accessibility={{
      ariaLabel: "Monthly sales chart for 2026",
      role: "img",
      focusable: true,
      tabIndex: 0,
    }}
  >
    <ChartTitle text="Monthly sales" />
  </Chart>
</section>
```

Do not assume `ChartTitle` or every chart child supports a direct `ariaLabel` property. Verify child accessibility through its current API before adding one.

## Keyboard navigation

Keyboard behavior can vary by enabled features and package version. Test the current Pure React build rather than copying a shortcut table from an older EJ2 source.

At minimum, verify that users can:

- reach the chart in a logical focus order
- see the focused element clearly
- move through interactive chart elements using documented keys
- activate interactive legend or toolbar controls
- dismiss transient content such as tooltips
- exit the chart without a keyboard trap

Do not publish shortcuts such as `Alt + J`, `Ctrl + +`, or `R` unless they appear in the current Pure React keyboard-navigation documentation and work in the tested package version.

## Accessible zoom controls

Use the zoom settings accessibility object when zoom controls are enabled.

```tsx
<ChartZoomSettings
  selectionZoom={true}
  mouseWheelZoom={true}
  pinchZoom={true}
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

Test the toolbar with keyboard navigation and a screen reader. Mouse-wheel, drag, and pinch interactions should not be the only way to inspect important detail.

## Color contrast

Apply WCAG contrast requirements according to the type of content:

- normal text: at least `4.5:1` under WCAG AA
- large text: at least `3:1` under WCAG AA
- meaningful graphical objects and adjacent state indicators: at least `3:1` where WCAG 1.4.11 applies

A `3:1` graphics requirement is associated with WCAG AA non-text contrast, not automatically WCAG AAA.

Check contrast for:

- title and axis text
- legend labels
- data labels
- tooltip text and background
- focus outlines
- series against the plot background
- selected and highlighted states
- grid lines when needed to interpret values

## Accessible color palette

Use distinguishable colors that maintain contrast against the chart background.

```tsx
const accessiblePalette = [
  "#005A9C",
  "#A4262C",
  "#107C10",
  "#5C2D91",
];

<Chart
  background="#FFFFFF"
  border={{
    color: "#767676",
    width: 1,
    dashArray: "",
  }}
  palettes={accessiblePalette}
/>
```

Do not assume a palette is accessible merely because colors are bright or named “high contrast.” Test actual color pairs and state changes.

Avoid combinations such as white and pale yellow on a white background.

## Do not rely on color alone

Combine color with another visual cue such as:

- selection or highlight patterns
- marker shapes
- direct labels
- distinct dash patterns for lines
- textual status or legend descriptions

```tsx
<Chart>
  <ChartSelection
    mode="Point"
    pattern="Crosshatch"
  />

  <ChartHighlight
    mode="Point"
    fill="rgba(0, 90, 156, 0.3)"
    pattern="Dots"
  />
</Chart>
```

Use exact Pure React pattern names. The current pattern value is `Crosshatch`, not `CrossHatch`.

## Selection pattern values

Verified selection and highlight patterns include:

- `None`
- `Chessboard`
- `Dots`
- `DiagonalForward`
- `Crosshatch`
- `Pacman`
- `DiagonalBackward`
- `Grid`
- `Turquoise`
- `Star`
- `Triangle`
- `Circle`
- `Tile`
- `HorizontalDash`
- `VerticalDash`
- `Rectangle`
- `Box`
- `VerticalStripe`
- `HorizontalStripe`
- `Bubble`

Do not use unsupported values such as `Diagonals`, `LightBlue`, `Orange`, or `Pink`.

## Markers as a non-color cue

Use different marker shapes for line or scatter series.

```tsx
<ChartSeries
  dataSource={salesData}
  xField="month"
  yField="value"
  type="Line"
  name="Sales"
>
  <ChartMarker
    visible={true}
    shape="Circle"
    width={10}
    height={10}
  />
</ChartSeries>

<ChartSeries
  dataSource={targetData}
  xField="month"
  yField="value"
  type="Line"
  name="Target"
  dashArray="5,5"
>
  <ChartMarker
    visible={true}
    shape="Diamond"
    width={10}
    height={10}
  />
</ChartSeries>
```

Use direct `width` and `height` properties on `ChartMarker`. Do not use an unsupported `size={{ ... }}` object.

Marker dimensions do not automatically create a 44-by-44 CSS-pixel target. Test the actual interactive hit area on touch devices.

## Data labels

Use `ChartDataLabel` inside `ChartMarker` for marker-based series.

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
    width={8}
    height={8}
  >
    <ChartDataLabel
      visible={true}
      position="Top"
      format="{value}"
      font={{
        color: "#1A1A1A",
        fontFamily: "Arial",
        fontSize: "12px",
        fontStyle: "Normal",
        fontWeight: "Normal",
        opacity: 1,
      }}
    />
  </ChartMarker>
</ChartSeries>
```

The Pure React data-label font uses `fontSize`, not `size`.

Avoid displaying every label when labels overlap or make the chart harder to read. A nearby accessible data table or summary may be more effective for dense data.

## Axis labels

Use axis child components and provide clear axis titles.

```tsx
<ChartPrimaryXAxis valueType="DateTime">
  <ChartAxisTitle text="Month" />
  <ChartAxisLabel
    format="MMM yyyy"
    fontSize="12px"
    color="#242424"
  />
</ChartPrimaryXAxis>

<ChartPrimaryYAxis valueType="Double">
  <ChartAxisTitle text="Revenue in US dollars" />
  <ChartAxisLabel
    format="{value}"
    fontSize="12px"
    color="#242424"
  />
</ChartPrimaryYAxis>
```

Do not use unverified root-axis properties such as `labelStyle`, `labelFormat`, or `labelRotationAngle` when the Pure React architecture provides `ChartAxisLabel` as a child.

Use `rotationAngle` on the verified component when rotation is required. There is no universal 45-degree maximum. Choose the smallest rotation that prevents overlap while preserving readability.

## Tooltips and hover-only information

Tooltips are useful supplements, but essential information should not be available only on hover.

```tsx
<ChartTooltip
  enable={true}
  shared={true}
/>
```

Also provide one or more of:

- visible labels for key values
- a concise text summary
- an accessible data table
- keyboard-accessible point navigation
- drill-down controls that work without a pointer

Do not claim that `visible={false}` on a data label means the label appears on hover. Data-label visibility and tooltip behavior are separate features.

## Accessible data alternative

For complex or high-density charts, provide a table or downloadable data representation near the chart.

```tsx
<section aria-labelledby="revenue-heading">
  <h2 id="revenue-heading">Quarterly revenue</h2>

  <Chart
    accessibility={{
      ariaLabel:
        "Quarterly revenue chart. A data table follows the chart.",
      role: "img",
      focusable: true,
      tabIndex: 0,
    }}
  >
    {/* axes and series */}
  </Chart>

  <table>
    <caption>Quarterly revenue values</caption>
    <thead>
      <tr>
        <th scope="col">Quarter</th>
        <th scope="col">Revenue</th>
      </tr>
    </thead>
    <tbody>
      {data.map((item) => (
        <tr key={item.quarter}>
          <th scope="row">{item.quarter}</th>
          <td>{item.revenue}</td>
        </tr>
      ))}
    </tbody>
  </table>
</section>
```

Keep the table data synchronized with the chart data.

## Responsive and touch support

Use responsive container sizing and verify the result at mobile zoom levels.

```tsx
<div style={{ width: "100%", minHeight: "360px" }}>
  <Chart width="100%" height="360px">
    {/* chart children */}
  </Chart>
</div>
```

Adapt layout with React state, CSS media queries, or a media-query hook rather than reading `window.innerWidth` directly during render.

Ensure custom buttons and controls meet the applicable WCAG target-size requirement. WCAG 2.2 AA includes Target Size (Minimum) at 24 by 24 CSS pixels, subject to exceptions. The 44-by-44 target is an enhanced AAA criterion, not a universal AA requirement.

## Right-to-left content

Do not assume an `enableRtl` prop exists on the current Pure React root API when it is absent from `ChartProps`. Prefer a semantic `dir` wrapper and verify the chart's rendering.

```tsx
<div dir="rtl">
  <Chart
    accessibility={{
      ariaLabel: "مخطط المبيعات الشهرية",
      role: "img",
      focusable: true,
      tabIndex: 0,
    }}
  >
    <ChartTitle text="المبيعات الشهرية" />
  </Chart>
</div>
```

Use locale-aware number and date formatting consistently across axes, tooltips, labels, summaries, and tables.

## Automated testing

Use automated scanning as one part of accessibility testing.

```tsx
import { render } from "@testing-library/react";
import { axe, toHaveNoViolations } from "jest-axe";

expect.extend(toHaveNoViolations);

test("chart has no detectable accessibility violations", async () => {
  const { container } = render(
    <Chart
      accessibility={{
        ariaLabel: "Monthly sales chart",
        role: "img",
        focusable: true,
        tabIndex: 0,
      }}
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
    </Chart>,
  );

  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

Automated tools cannot verify chart meaning, reading order, keyboard usability, equivalent data access, or whether the description communicates the correct interpretation.

## Manual testing

Test with:

- keyboard-only navigation
- browser zoom and text scaling
- at least one screen reader used by the target audience
- high-contrast or forced-colors mode where applicable
- touch input at target mobile sizes
- reduced-motion settings when animations are enabled
- color-vision deficiency simulations plus manual contrast checks

Verify:

1. Focus reaches the chart in a logical order.
2. Focus remains visible.
3. Users can leave the chart without a keyboard trap.
4. Interactive points, legend items, and toolbar controls are operable.
5. Tooltips or equivalent content are available without hover alone.
6. Titles, axes, series, and units are understandable.
7. Selected and highlighted states do not rely only on color.
8. Screen-reader output is concise and accurate.
9. A text summary or data alternative is available when needed.
10. Layout remains usable at mobile widths and browser zoom.

## Complete accessible chart example

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartDataLabel,
  ChartLegend,
  ChartMarker,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSelection,
  ChartSeries,
  ChartSeriesCollection,
  ChartTitle,
  ChartTooltip,
} from "@syncfusion/react-charts";

const data = [
  { month: "Jan", sales: 100 },
  { month: "Feb", sales: 120 },
  { month: "Mar", sales: 110 },
  { month: "Apr", sales: 145 },
];

export default function AccessibleChart() {
  return (
    <section aria-labelledby="sales-heading">
      <h2 id="sales-heading">2026 sales analysis</h2>
      <p>
        Monthly sales increased overall from January through April.
      </p>

      <Chart
        accessibility={{
          ariaLabel:
            "Monthly sales from January through April 2026. Values range from 100 to 145.",
          role: "img",
          focusable: true,
          tabIndex: 0,
        }}
        focusOutline={{
          color: "#005FCC",
          width: 2,
          offset: 2,
        }}
        background="#FFFFFF"
        border={{
          color: "#767676",
          width: 1,
          dashArray: "",
        }}
        palettes={["#005A9C"]}
      >
        <ChartTitle
          text="Monthly sales"
          color="#1A1A1A"
          fontSize="18px"
          fontWeight="Bold"
        />

        <ChartPrimaryXAxis valueType="Category">
          <ChartAxisTitle text="Month" />
          <ChartAxisLabel
            fontSize="12px"
            color="#242424"
          />
        </ChartPrimaryXAxis>

        <ChartPrimaryYAxis valueType="Double" minimum={0}>
          <ChartAxisTitle text="Sales" />
          <ChartAxisLabel
            format="{value}"
            fontSize="12px"
            color="#242424"
          />
        </ChartPrimaryYAxis>

        <ChartLegend
          visible={true}
          position="Bottom"
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
          pattern="Crosshatch"
        />

        <ChartSeriesCollection>
          <ChartSeries
            dataSource={data}
            xField="month"
            yField="sales"
            type="Column"
            name="Sales"
            fill="#005A9C"
            accessibility={{
              ariaLabel: "Monthly sales series",
              descriptionFormat:
                "${series.name}, ${point.x}: ${point.y}",
              focusable: true,
              role: "img",
              tabIndex: 0,
            }}
          >
            <ChartMarker>
              <ChartDataLabel
                visible={true}
                position="Top"
                format="{value}"
                font={{
                  color: "#1A1A1A",
                  fontFamily: "Arial",
                  fontSize: "12px",
                  fontStyle: "Normal",
                  fontWeight: "Normal",
                  opacity: 1,
                }}
              />
            </ChartMarker>
          </ChartSeries>
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

## Common errors

### Direct child ariaLabel props

Incorrect without child API support:

```tsx
<ChartTitle ariaLabel="Sales title" />
<ChartLegend ariaLabel="Series legend" />
<ChartSeries ariaLabel="Sales series" />
```

Use verified accessibility objects for components that expose them.

### Wrong marker sizing

Incorrect:

```tsx
<ChartMarker
  size={{ width: 15, height: 15 }}
/>
```

Correct:

```tsx
<ChartMarker
  width={15}
  height={15}
/>
```

### Wrong selection pattern

Use `Crosshatch`, not `CrossHatch`.

### Wrong data-label font property

Use `fontSize`, not `size`.

### Unverified RTL prop

Do not add `enableRtl` to `Chart` when it is absent from the current root API. Use a `dir="rtl"` wrapper and test the resulting layout.

### Compliance guarantees

Do not mark every WCAG criterion as passed based only on component documentation. Record results from actual automated and manual testing against the application's target conformance level.

## Validation checklist

Before returning an accessible chart implementation:

1. Provide a meaningful root `accessibility.ariaLabel`.
2. Choose an appropriate chart-level role.
3. Preserve logical focus order with `focusable` and `tabIndex`.
4. Keep a visible, high-contrast focus outline.
5. Use `ChartSeries.accessibility` for series descriptions.
6. Use `ChartLegend.accessibility` for legend labeling.
7. Do not invent direct `ariaLabel` props on child components.
8. Provide clear visible titles, axis titles, units, and labels.
9. Do not rely on color alone.
10. Use exact selection and highlight pattern names.
11. Use marker `width` and `height`, not a `size` object.
12. Use `fontSize` in Pure React font objects.
13. Avoid hover-only access to essential information.
14. Provide a text summary or data table for complex charts.
15. Verify text and non-text contrast using actual color pairs.
16. Test keyboard navigation and focus visibility.
17. Test with a screen reader used by the target audience.
18. Test browser zoom, mobile layout, touch input, and forced-colors mode.
19. Treat automated testing as a supplement to manual testing.
20. Do not claim WCAG compliance without application-level evidence.
21. Do not mix EJ2 configuration with Pure React component APIs.
22. Ensure every imported symbol is used.
23. Emit valid, unescaped TSX.

## Resources

- WCAG 2.2: https://www.w3.org/TR/WCAG22/
- WAI-ARIA Authoring Practices: https://www.w3.org/WAI/ARIA/apg/
- Section 508: https://www.section508.gov/
- WebAIM Contrast Checker: https://webaim.org/resources/contrastchecker/
- React accessibility guidance: https://react.dev/learn/accessibility
