# RTL Reference

Use this guidance when a Pure React Chart runs in a right-to-left interface. Enable RTL through the Syncfusion `Provider`, keep language and direction metadata aligned, and test axes, legends, tooltips, labels, controls, and surrounding layout at the final responsive sizes.

## Core rules

- Set `dir="rtl"` through the Syncfusion `Provider` for the chart subtree.
- Add the correct `lang` attribute to the surrounding semantic container.
- Let RTL mirror layout-sensitive chart elements before adding manual overrides.
- Review axis placement, axis direction, legend alignment, tooltip placement, annotations, and external controls.
- Keep numeric, date, currency, and percentage formatting tied to the active locale.
- Do not reverse the data array merely to imitate RTL.
- Do not use axis `inverted` unless the value scale itself must run in reverse.
- Test mixed-direction text, such as RTL labels containing Latin product names or numbers.

## Enable RTL

Syncfusion Pure React components enable RTL by setting the `dir` property to `rtl` in the `Provider` context. This adds the package RTL class and aligns supported components in the right-to-left direction. 

```tsx
import { Provider } from "@syncfusion/react-base";
import {
  Chart,
  ChartPrimaryXAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

export default function RtlChart() {
  return (
    <Provider dir="rtl">
      <section lang="ar" dir="rtl">
        <Chart>
          <ChartPrimaryXAxis valueType="Category" />

          <ChartSeriesCollection>
            <ChartSeries
              dataSource={data}
              xField="category"
              yField="value"
              type="Column"
              name="القيمة"
            />
          </ChartSeriesCollection>
        </Chart>
      </section>
    </Provider>
  );
}
```

The chart RTL layout is designed to flow axes, labels, legends, and tooltips from right to left. 

## Application-level direction

Place the provider at the smallest stable scope that matches the application's direction model.

```tsx
<Provider dir={isRtl ? "rtl" : "ltr"}>
  <main lang={locale} dir={isRtl ? "rtl" : "ltr"}>
    <App />
  </main>
</Provider>
```

Use:

- `Provider.dir` for Syncfusion component direction
- HTML `dir` for surrounding document layout and text direction
- HTML `lang` for the content language
- `Intl` formatters for locale-aware values

Direction and locale are related but separate concerns.

## Do not reverse the source data

RTL changes presentation direction. It does not necessarily change chronological, categorical, ranked, or numeric meaning.

Incorrect:

```tsx
const rtlData = [...data].reverse();
```

Correct:

```tsx
<Provider dir="rtl">
  <ChartSeries
    dataSource={data}
    xField="date"
    yField="value"
    type="Line"
  />
</Provider>
```

Reverse or sort data only when the analytical order requires it, not as an RTL workaround.

## Axis placement

Review primary and additional axes after RTL is enabled. Use `opposedPosition` only when the design specifically requires the axis on the opposite side. The axis API separately provides `opposedPosition` for side placement and `inverted` for reversing maximum-to-minimum scale direction. 

```tsx
<ChartPrimaryYAxis
  valueType="Double"
  opposedPosition={false}
>
  <ChartAxisTitle text="القيمة" />
  <ChartAxisLabel format="{value}" />
</ChartPrimaryYAxis>
```

Do not automatically set both `opposedPosition` and `inverted` for RTL.

```tsx
<ChartPrimaryYAxis
  valueType="Double"
  inverted={true}
/>
```

Use `inverted` only when a reversed scale is meaningful, such as rank where lower values are better.

## Category and DateTime order

Preserve semantic order:

- chronological DateTime data should remain chronological
- ranked data should follow the intended best-to-worst or worst-to-best order
- process stages should retain process sequence
- categories should remain in the order required by the analysis

Test how the RTL layout renders the first and last category. Adjust only with documented axis configuration when the rendered order does not match the intended reading.

## Legend placement and alignment

`ChartLegend` supports `Auto`, `Top`, `Left`, `Bottom`, `Right`, and `Custom` positions. Alignment depends on whether the legend is horizontal or vertical. 

```tsx
<ChartLegend
  visible={true}
  position="Bottom"
  align="Center"
  inversed={true}
  toggleVisibility={true}
/>
```

`inversed` changes the order of the shape and text within each legend item. `reverse` changes the sequence of legend items. Use these only when the final RTL reading order requires them. 

```tsx
<ChartLegend
  visible={true}
  position="Bottom"
  inversed={true}
  reverse={false}
/>
```

Do not set both properties automatically. Test the result with the actual series names and locale.

## Tooltip placement

The tooltip supports automatic placement, fixed `location`, shared display, templates, and text styling. 

```tsx
<ChartTooltip
  enable={true}
  shared={true}
  format="${series.name}: ${point.y}"
/>
```

Prefer automatic placement first. Use `location` only when a fixed tooltip position is required and has been tested in both directions.

```tsx
<ChartTooltip
  enable={true}
  location={{ x: 24, y: 24 }}
/>
```

A fixed X coordinate is physical, not logical. Recalculate or provide direction-specific configuration when the tooltip must align to the logical start or end.

## Tooltip templates

Set direction and alignment explicitly inside custom templates when mixed-direction content is possible.

```tsx
const tooltipTemplate = (props: ChartTooltipTemplateProps) => (
  <div
    dir="rtl"
    lang="ar"
    style={{
      padding: "8px",
      textAlign: "start",
    }}
  >
    <strong>{String(props.x)}</strong>
    <div>{numberFormatter.format(props.y)}</div>
  </div>
);

<ChartTooltip
  enable={true}
  template={tooltipTemplate}
/>
```

Use `textAlign: "start"` instead of hard-coding `left` or `right` when logical alignment is intended.

## Locale-aware formatting

Keep plotted values numeric and format display text with the active locale.

```tsx
const numberFormatter = new Intl.NumberFormat("ar-SA", {
  maximumFractionDigits: 1,
});

const currencyFormatter = new Intl.NumberFormat("ar-SA", {
  style: "currency",
  currency: "SAR",
  maximumFractionDigits: 0,
});

const dateFormatter = new Intl.DateTimeFormat("ar-SA", {
  dateStyle: "medium",
});
```

Apply the same locale to axis labels, data labels, tooltips, summaries, and accessible descriptions.

```tsx
<ChartAxisLabel
  formatter={(value) => numberFormatter.format(value)}
/>
```

Do not concatenate currency symbols manually because symbol order and spacing are locale dependent.

## Mixed-direction labels

RTL labels may contain Latin names, model numbers, URLs, dates, or units. Isolate mixed-direction fragments when rendering custom HTML.

```tsx
<span dir="rtl">
  المنتج <bdi>Surface Pro 11</bdi>
</span>
```

For plain chart strings, test the rendered output with actual localized content. Avoid embedding directional control characters unless the localization system manages them deliberately.

## External controls

Mirror surrounding controls with logical CSS properties.

```css
.chart-toolbar {
  display: flex;
  justify-content: flex-start;
  gap: 8px;
  margin-block-end: 12px;
}

.chart-card {
  padding-inline: 16px;
  padding-block: 16px;
}
```

Prefer:

- `margin-inline-start` and `margin-inline-end`
- `padding-inline`
- `inset-inline-start` and `inset-inline-end`
- `text-align: start`

Avoid maintaining separate physical `left` and `right` rules when logical properties can support both directions.

## Responsive RTL example

```tsx
import { Provider } from "@syncfusion/react-base";
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartLegend,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
  ChartTitle,
  ChartTooltip,
} from "@syncfusion/react-charts";

const data = [
  { category: "الفئة الأولى", value: 42 },
  { category: "الفئة الثانية", value: 35 },
  { category: "الفئة الثالثة", value: 51 },
  { category: "الفئة الرابعة", value: 47 },
];

export default function ResponsiveRtlChart() {
  const numberFormatter = new Intl.NumberFormat("ar-SA");

  return (
    <Provider dir="rtl">
      <section
        className="chart-card"
        lang="ar"
        dir="rtl"
        aria-labelledby="chart-heading"
      >
        <h2 id="chart-heading">نظرة عامة</h2>

        <div className="chart-canvas">
          <Chart
            width="100%"
            height="100%"
            accessibility={{
              ariaLabel: "مخطط مقارنة الفئات",
              role: "img",
              focusable: true,
              tabIndex: 0,
            }}
          >
            <ChartTitle text="مقارنة الفئات" />

            <ChartPrimaryXAxis valueType="Category">
              <ChartAxisTitle text="الفئة" />
              <ChartAxisLabel
                enableWrap={true}
                maxLabelWidth={90}
                intersectAction="Wrap"
                edgeLabelPlacement="Shift"
              />
            </ChartPrimaryXAxis>

            <ChartPrimaryYAxis valueType="Double" minimum={0}>
              <ChartAxisTitle text="القيمة" />
              <ChartAxisLabel
                formatter={(value) =>
                  numberFormatter.format(value)
                }
              />
            </ChartPrimaryYAxis>

            <ChartLegend
              visible={true}
              position="Bottom"
              align="Center"
              inversed={true}
            />

            <ChartTooltip
              enable={true}
              shared={true}
            />

            <ChartSeriesCollection>
              <ChartSeries
                dataSource={data}
                xField="category"
                yField="value"
                type="Column"
                name="القيمة"
              />
            </ChartSeriesCollection>
          </Chart>
        </div>
      </section>
    </Provider>
  );
}
```

```css
.chart-card {
  min-width: 0;
  padding-block: 16px;
  padding-inline: 16px;
}

.chart-card h2 {
  margin-block: 0 12px;
  text-align: start;
}

.chart-canvas {
  width: 100%;
  height: 420px;
  min-width: 0;
}

@media (max-width: 640px) {
  .chart-canvas {
    height: 340px;
  }
}
```

## Pie and donut charts

Wrap `PieChart` with the same provider and semantic direction container.

```tsx
<Provider dir="rtl">
  <section lang="ar" dir="rtl">
    <PieChart width="100%" height="420px">
      <PieChartLegend
        visible={true}
        position="Bottom"
        align="Center"
        inversed={true}
      />

      <PieChartTooltip enable={true} />

      <PieChartSeriesCollection>
        <PieChartSeries
          dataSource={data}
          xField="category"
          yField="value"
        />
      </PieChartSeriesCollection>
    </PieChart>
  </section>
</Provider>
```

Do not manually reverse slice angles or calculate custom geometry for RTL. Review slice order, labels, connectors, legend order, and tooltip alignment with the actual data.

## Accessibility

- Keep the natural keyboard focus order after mirroring the visual layout.
- Use `tabIndex: 0`, not positive values, for focusable chart elements.
- Provide localized accessibility labels and summaries.
- Maintain a visible focus outline.
- Do not rely only on tooltip hover or color.
- Confirm screen-reader reading order for mixed RTL and LTR text.
- Test external filters, legend controls, and export buttons with keyboard navigation.

RTL visual mirroring must not produce a contradictory DOM or focus order.

## Testing checklist

Test both `ltr` and `rtl` in the same application build:

1. Primary X- and Y-axis sides and reading direction.
2. Additional named axes and `opposedPosition` behavior.
3. Category and DateTime first-to-last order.
4. Legend position, item order, shape-text order, and paging.
5. Tooltip automatic and fixed placement.
6. Data labels, crosshair labels, annotations, and striplines.
7. Zoom toolbar, scrollbars, and external controls.
8. Long translated labels and mixed-direction text.
9. Locale-specific digits, dates, currencies, and percentages.
10. Keyboard focus order and visible focus styling.
11. Narrow mobile panels and browser zoom.
12. Printing and exported images or PDFs.

## Common errors

### Using only an HTML direction wrapper

An HTML `dir="rtl"` wrapper helps surrounding layout, but Syncfusion Pure React RTL should be enabled through `Provider dir="rtl"` so supported component internals receive the proper context. 

### Reversing the dataset

Do not reverse data automatically. Preserve semantic chronology and category order.

### Using `inverted` as an RTL switch

`inverted` reverses the value direction from maximum to minimum. It is not the chart RTL setting. 

### Hard-coded left and right positions

Use logical CSS properties and direction-aware configuration where possible.

### Reversing both legend controls

`inversed` and `reverse` affect different aspects of the legend. Test each independently before combining them. 

### Locale-insensitive numbers

RTL direction alone does not format numbers, dates, or currencies. Use the active locale through `Intl` or the documented chart formatting APIs.

## Validation checklist

Before returning an RTL implementation:

1. Enable direction with `Provider dir="rtl"`.
2. Set matching semantic `dir` and `lang` attributes around the content.
3. Keep raw data order unless the analysis requires sorting or reversal.
4. Do not use axis `inverted` as an RTL flag.
5. Use `opposedPosition` only for deliberate axis placement.
6. Test primary and additional axes in RTL.
7. Test legend `position`, `align`, `inversed`, and `reverse` independently.
8. Prefer automatic tooltip positioning before fixed coordinates.
9. Make custom tooltip templates direction aware.
10. Use one active locale across chart labels and surrounding UI.
11. Use `Intl` for locale-aware numbers, dates, currencies, and percentages.
12. Test mixed-direction text with real translations.
13. Use logical CSS properties for external layout.
14. Keep visual order and keyboard focus order consistent.
15. Test mobile, browser zoom, print, and export output.
16. Test both Cartesian and pie families when the application uses both.
17. Do not mix EJ2 RTL flags with Pure React Provider behavior.
18. Ensure every imported symbol is used.
19. Emit valid, unescaped TSX.
