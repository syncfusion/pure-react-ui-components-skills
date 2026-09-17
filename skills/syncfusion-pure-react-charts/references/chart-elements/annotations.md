# Annotations Reference

Use chart annotations to place text, HTML content, or content referenced by a DOM element ID at a data point or pixel position.

## Component hierarchy

Place `ChartAnnotation` inside `ChartAnnotationCollection`, and place the collection directly inside `Chart`.

```tsx
import {
  Chart,
  ChartAnnotation,
  ChartAnnotationCollection,
  ChartPrimaryXAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { year: 2020, value: 35 },
  { year: 2021, value: 42 },
  { year: 2022, value: 50 },
];

<Chart>
  <ChartPrimaryXAxis valueType="Double" />

  <ChartAnnotationCollection>
    <ChartAnnotation
      x={2022}
      y={50}
      coordinateUnit="Point"
      content="Peak Value"
    />
  </ChartAnnotationCollection>

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="year"
      yField="value"
      type="Line"
    />
  </ChartSeriesCollection>
</Chart>
```

Do not place `ChartAnnotation` directly under `Chart`. Do not place annotations inside `ChartSeries`.

## Verified annotation properties

The official `ChartAnnotationProps` API documents:

- `accessibility`: `ChartAccessibilityProps`; default `{ ariaLabel: "", role: "img", focusable: false, tabIndex: 0 }`
- `content`: `string`; default `""`
- `coordinateUnit`: `"Point" | "Pixel"`; default `"Point"`
- `hAlign`: `"Left" | "Center" | "Right"`; default `"Center"`
- `vAlign`: `"Top" | "Center" | "Bottom"`; default `"Center"`
- `x`: `string | Date | number | null`; default `null`
- `xAxisName`: `string | null`; default `null`
- `y`: `string | number | null`; default `null`
- `yAxisName`: `string | null`; default `null`

Do not add `fill`, `border`, `font`, `style`, or another appearance property to `ChartAnnotation` unless the current official API explicitly adds that property. Style HTML annotation content through the referenced HTML itself.

## Coordinate units

### Point coordinates

Use `coordinateUnit="Point"` to position the annotation using axis values. `Point` is the documented default.

```tsx
<ChartAnnotation
  x={2022}
  y={50}
  coordinateUnit="Point"
  content="Peak Value"
/>
```

For point coordinates:

- `x` must be compatible with the selected X-axis.
- `y` must be compatible with the selected Y-axis.
- Use `xAxisName` or `yAxisName` when the annotation belongs to additional named axes.
- The annotation follows its data position when the visible chart range changes.

### Pixel coordinates

Use `coordinateUnit="Pixel"` to position the annotation with pixel offsets.

```tsx
<ChartAnnotation
  x={100}
  y={40}
  coordinateUnit="Pixel"
  content="Chart note"
/>
```

For pixel coordinates, `x` and `y` represent offsets rather than data values. Do not assign `xAxisName` or `yAxisName` unless the current API behavior explicitly requires axis binding for the requested scenario.

## Axis-compatible point values

### Numeric axis

```tsx
<ChartPrimaryXAxis valueType="Double" />

<ChartAnnotation
  coordinateUnit="Point"
  x={20}
  y={75}
  content="Target reached"
/>
```

### Category axis

Use a category value that exists in the mapped data.

```tsx
<ChartPrimaryXAxis valueType="Category" />

<ChartAnnotation
  coordinateUnit="Point"
  x="March"
  y={52}
  content="Highest month"
/>
```

### DateTime axis

Use a valid `Date` value for the X coordinate.

```tsx
<ChartPrimaryXAxis valueType="DateTime" />

<ChartAnnotation
  coordinateUnit="Point"
  x={new Date(2026, 2, 1)}
  y={52}
  content="March peak"
/>
```

Do not use an arbitrary date string when the axis and data use `Date` objects.

## Multiple annotations

Add one `ChartAnnotation` child for each annotation.

```tsx
<ChartAnnotationCollection>
  <ChartAnnotation
    x={2020}
    y={50}
    coordinateUnit="Point"
    content="Peak"
  />
  <ChartAnnotation
    x={2022}
    y={30}
    coordinateUnit="Point"
    content="Trough"
  />
  <ChartAnnotation
    x={100}
    y={40}
    coordinateUnit="Pixel"
    content="Chart note"
  />
</ChartAnnotationCollection>
```

Keep every annotation's coordinate values compatible with its own `coordinateUnit`.

## Alignment

Use `hAlign` and `vAlign` to position the annotation relative to its anchor.

Horizontal values:

- `Left`
- `Center`
- `Right`

Vertical values:

- `Top`
- `Center`
- `Bottom`

```tsx
<ChartAnnotation
  x={2020}
  y={50}
  coordinateUnit="Point"
  content="Peak Sales"
  hAlign="Center"
  vAlign="Top"
/>
```

Alignment does not change the anchor coordinate. It changes how annotation content is aligned relative to that anchor.

## Named-axis binding

When `coordinateUnit="Point"` and the chart uses additional named axes, bind the annotation using `xAxisName` and `yAxisName`.

```tsx
<ChartAxes>
  <ChartAxis name="growthAxis" opposedPosition={true} />
</ChartAxes>

<ChartAnnotationCollection>
  <ChartAnnotation
    coordinateUnit="Point"
    x="Q4"
    y={18}
    yAxisName="growthAxis"
    content="Highest growth"
  />
</ChartAnnotationCollection>
```

The annotation axis name must match the corresponding `ChartAxis.name` exactly, including casing.

For an additional X-axis:

```tsx
<ChartAnnotation
  coordinateUnit="Point"
  x={new Date(2026, 8, 1)}
  y={42}
  xAxisName="dateAxis"
  content="September"
/>
```

## Plain text content

`content` accepts a string.

```tsx
<ChartAnnotation
  x="Apr"
  y={51}
  coordinateUnit="Point"
  content="Peak Value"
/>
```

Do not pass a JSX element directly to `content` when its documented type is `string`.

Incorrect:

```tsx
<ChartAnnotation
  content={<strong>Peak Value</strong>}
/>
```

Use a plain text string, an HTML string, or a DOM element ID as documented by the API.

## HTML string content

The API permits an HTML string.

```tsx
const annotationContent = `
  <div style="padding:6px;background:#f0f0f0;border-radius:4px">
    <strong>Peak Value</strong>
    <div>$50k</div>
  </div>
`;

<ChartAnnotation
  x={2020}
  y={50}
  coordinateUnit="Point"
  content={annotationContent}
/>
```

Do not insert untrusted user-provided HTML without sanitization. Prefer plain text when HTML formatting is unnecessary.

## DOM element ID content

The API also permits a string that references a DOM element ID.

```tsx
<div id="peak-annotation" style={{ display: "none" }}>
  <strong>Peak Value</strong>
  <div>$50k</div>
</div>

<ChartAnnotation
  x={2020}
  y={50}
  coordinateUnit="Point"
  content="peak-annotation"
/>
```

Ensure that:

- The referenced element exists when the chart renders.
- The ID is unique in the document.
- The content does not rely on APIs unsupported by the chart annotation renderer.

## Styling annotation content

Because `ChartAnnotation` does not document direct `fill`, `border`, or `font` props, apply styling within HTML content or the referenced DOM element.

```tsx
const styledContent = `
  <span
    style="display:inline-block;padding:4px 8px;background:#FFD700;
           border:1px solid #FF6B6B;color:#333;font-size:14px;
           font-weight:700;border-radius:4px"
  >
    Maximum
  </span>
`;

<ChartAnnotation
  x={2020}
  y={50}
  coordinateUnit="Point"
  content={styledContent}
/>
```

Do not transfer text-style properties from titles, labels, or other chart components to `ChartAnnotation`.

## Accessibility

Use the annotation `accessibility` object when the annotation needs a specific accessible label or focus behavior.

```tsx
<ChartAnnotation
  x="Apr"
  y={51}
  coordinateUnit="Point"
  content="Peak sales"
  accessibility={{
    ariaLabel: "Peak sales value of 51 in April",
    role: "img",
    focusable: false,
    tabIndex: 0,
  }}
/>
```

The documented default is:

```tsx
{
  ariaLabel: "",
  role: "img",
  focusable: false,
  tabIndex: 0,
}
```

Set `focusable` deliberately. Avoid making decorative annotations keyboard-focusable.

## Complete annotation example

```tsx
import {
  Chart,
  ChartAnnotation,
  ChartAnnotationCollection,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { month: "Jan", sales: 35 },
  { month: "Feb", sales: 42 },
  { month: "Mar", sales: 38 },
  { month: "Apr", sales: 51 },
  { month: "May", sales: 47 },
];

const peakContent = `
  <div style="padding:6px 8px;background:#fff3cd;border:1px solid #e0a800;
              border-radius:4px;color:#333;font-weight:600">
    Peak: 51
  </div>
`;

export default function AnnotationChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis valueType="Category">
        <ChartAxisTitle text="Month" />
        <ChartAxisLabel edgeLabelPlacement="Shift" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double">
        <ChartAxisTitle text="Sales" />
        <ChartAxisLabel format="{value}" />
      </ChartPrimaryYAxis>

      <ChartAnnotationCollection>
        <ChartAnnotation
          x="Apr"
          y={51}
          coordinateUnit="Point"
          content={peakContent}
          hAlign="Center"
          vAlign="Top"
          accessibility={{
            ariaLabel: "Peak sales value of 51 in April",
            role: "img",
            focusable: false,
            tabIndex: 0,
          }}
        />
      </ChartAnnotationCollection>

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="sales"
          type="Line"
          name="Sales"
          width={2}
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Common errors

### Passing JSX to `content`

Incorrect:

```tsx
<ChartAnnotation content={<div>Peak</div>} />
```

Correct:

```tsx
<ChartAnnotation content="Peak" />
```

or provide an HTML string or DOM element ID.

### Using unsupported style props

Incorrect:

```tsx
<ChartAnnotation
  fill="#FFD700"
  border={{ color: "#333", width: 1 }}
  font={{ size: "14px" }}
/>
```

Style the HTML content instead.

### Using the wrong coordinate-unit casing

Incorrect:

```tsx
<ChartAnnotation coordinateUnit="point" />
```

Correct:

```tsx
<ChartAnnotation coordinateUnit="Point" />
```

### Using incompatible point coordinates

Do not use a numeric year such as `2020` when the X-axis is a Category axis containing labels such as `"FY 2020"`. The annotation X value must match the axis value.

### Name mismatch on additional axes

Incorrect:

```tsx
<ChartAxis name="growthAxis" />
<ChartAnnotation yAxisName="GrowthAxis" />
```

Correct:

```tsx
<ChartAxis name="growthAxis" />
<ChartAnnotation yAxisName="growthAxis" />
```

## Validation checklist

Before returning an annotation implementation:

1. Import `ChartAnnotationCollection` and `ChartAnnotation` from `@syncfusion/react-charts`.
2. Place `ChartAnnotation` inside `ChartAnnotationCollection`.
3. Place the collection directly inside `Chart`.
4. Use only `Point` or `Pixel` for `coordinateUnit`.
5. Match Point coordinates to the configured axis value types.
6. Use pixel offsets for Pixel coordinates.
7. Use only `Left`, `Center`, or `Right` for `hAlign`.
8. Use only `Top`, `Center`, or `Bottom` for `vAlign`.
9. Treat `content` as a string.
10. Use a plain text string, HTML string, or DOM element ID.
11. Do not pass JSX directly to `content`.
12. Do not add unsupported `fill`, `border`, or `font` props to `ChartAnnotation`.
13. Match `xAxisName` and `yAxisName` exactly with named axes when Point coordinates use additional axes.
14. Avoid unsafe unsanitized HTML.
15. Ensure all imported components are used.
16. Emit valid, unescaped TSX.
