# Striplines Reference

Use striplines to draw horizontal or vertical reference bands and lines across the chart plot area. Striplines are axis-owned elements, so configure each stripline inside the axis whose values define its range.

## Required component hierarchy

Use `ChartStripLines` (capital L) as a child of `ChartPrimaryXAxis`, `ChartPrimaryYAxis`, or a named `ChartAxis`, and place each `ChartStripLine` inside it.

```tsx
import {
  Chart,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
  ChartStripLine,
  ChartStripLines,
} from "@syncfusion/react-charts";

<Chart>
  <ChartPrimaryXAxis valueType="Category" />

  <ChartPrimaryYAxis valueType="Double">
    <ChartStripLines>
      <ChartStripLine
        visible={true}
        range={{ start: 100, end: 120 }}
        style={{ color: "#FFF3E0" }}
        text={{ content: "Target range" }}
      />
    </ChartStripLines>
  </ChartPrimaryYAxis>

  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="category"
      yField="value"
      type="Column"
    />
  </ChartSeriesCollection>
</Chart>
```

Props placed directly on the `ChartStripLines` wrapper are ignored. Each stripline must be a `ChartStripLine` child — the chart harvests only `ChartStripLine` children of the collection.

## Horizontal and vertical striplines

The owning axis determines the stripline direction:

- A stripline configured on the Y-axis renders horizontally across the plot area.
- A stripline configured on the X-axis renders vertically across the plot area.

### Horizontal stripline

```tsx
<ChartPrimaryYAxis valueType="Double">
  <ChartStripLines>
    <ChartStripLine
      visible={true}
      range={{ start: 80, end: 100 }}
      style={{ color: "#E8F5E9", opacity: 0.5 }}
      text={{ content: "Acceptable range" }}
    />
  </ChartStripLines>
</ChartPrimaryYAxis>
```

### Vertical stripline

```tsx
<ChartPrimaryXAxis valueType="DateTime">
  <ChartStripLines>
    <ChartStripLine
      visible={true}
      range={{
        start: new Date(2026, 0, 1),
        end: new Date(2026, 2, 31),
      }}
      style={{ color: "#E3F2FD", opacity: 0.5 }}
      text={{ content: "First quarter" }}
    />
  </ChartStripLines>
</ChartPrimaryXAxis>
```

Use values compatible with the owning axis type.

## Verified properties

The Pure React `ChartStripLineProps` API documents these nested objects:

- `visible: boolean`, default `false`
- `range: StripLineRangeProps`
  - `start`: starting axis value (string, number, or Date); ignored if `size` is set
  - `end`: ending axis value; ignored if `size` is set
  - `size`: stripline width or height calculated from `start`
  - `sizeType`: `'Auto' | 'Pixel' | 'Years' | 'Months' | 'Days' | 'Hours' | 'Minutes' | 'Seconds'`
  - `shouldStartFromAxis: boolean`: render from the axis origin (zero)
- `style: StripLineStyleProps`
  - `color`: band background color, default `'#808080'`
  - `opacity`: 0 through 1, default `1`
  - `dashArray`: border dash pattern string
  - `imageUrl`: background image URL
  - `border: ChartBorderProps` (`{ color, width, dashArray }`)
  - `zIndex`: `'Behind' | 'Over'`, default `'Behind'`
- `text: StripLineTextProps`
  - `content`: label text
  - `font: ChartFontProps` (`color`, `fontFamily`, `fontSize`, `fontStyle`, `fontWeight`, `opacity`)
  - `rotation`: text rotation in degrees
  - `hAlign`: `'Left' | 'Center' | 'Right'`, default `'Center'`
  - `vAlign`: `'Top' | 'Center' | 'Bottom'`, default `'Center'`
- `repeat: StripLineRepeatProps`
  - `enable: boolean`, default `false`
  - `every`: interval value (string, number, or Date)
  - `until`: repeat end value (string, number, or Date)
- `segment: StripLineSegmentProps`
  - `enable: boolean`, default `false`
  - `axisName`: target axis name
  - `start` / `end`: visibility range on the target axis

## Range configuration

### Start and end range

```tsx
<ChartStripLine
  visible={true}
  range={{ start: 100, end: 200 }}
/>
```

Use `start < end` for an ordinary increasing numeric or DateTime axis. The values must fall within or intersect the visible axis range to appear.

### Start and size

Use `size` when the stripline should extend a specified amount from `start`.

```tsx
<ChartStripLine
  visible={true}
  range={{
    start: 100,
    size: 25,
    sizeType: "Auto",
  }}
/>
```

Use only the exact `StripLineSizeUnit` literals: `Auto`, `Pixel`, `Years`, `Months`, `Days`, `Hours`, `Minutes`, and `Seconds`. In a numeric axis, `Auto` interprets `size` as a number; in a DateTime axis, as milliseconds.

### Start from axis origin

```tsx
<ChartStripLine
  visible={true}
  range={{
    start: 100,
    size: 20,
    shouldStartFromAxis: true,
  }}
/>
```

Use `shouldStartFromAxis` only when origin-relative rendering is part of the requested stripline design.

## Threshold line versus range band

Use a range with different start and end values for a band.

```tsx
<ChartStripLine
  visible={true}
  range={{ start: 100, end: 150 }}
  style={{ color: "#E8F5E9", opacity: 0.45 }}
/>
```

For a threshold line, use `range.start` with a small `size` in pixels:

```tsx
<ChartStripLine
  visible={true}
  range={{ start: 100, size: 1, sizeType: "Pixel" }}
  style={{ color: "#D32F2F" }}
  text={{ content: "Target: 100" }}
/>
```

## Appearance

All visual styling lives in the `style` object.

```tsx
<ChartStripLine
  visible={true}
  range={{ start: 100, end: 200 }}
  style={{
    color: "#FFE5E5",
    opacity: 0.55,
    border: { color: "#D32F2F", width: 1, dashArray: "4,4" },
  }}
/>
```

Use opacity values from `0` through `1`.

## Stripline text

Configure the label through the `text` object.

```tsx
<ChartStripLine
  visible={true}
  range={{ start: 100, end: 120 }}
  text={{
    content: "Target range",
    font: {
      color: "#7F1D1D",
      fontSize: "12px",
      fontWeight: "Bold",
      fontFamily: "Arial",
      fontStyle: "Normal",
      opacity: 1,
    },
    hAlign: "Right",
    vAlign: "Top",
    rotation: 0,
  }}
/>
```

Use `font.fontSize`, not `size`, and `font.fontWeight`, not `bold`. Use `hAlign` and `vAlign` for alignment within the stripline.

## Multiple striplines

Add multiple `ChartStripLine` children to the same `ChartStripLines` collection.

```tsx
<ChartPrimaryYAxis valueType="Double">
  <ChartStripLines>
    <ChartStripLine
      visible={true}
      range={{ start: 0, end: 50 }}
      style={{ color: "#FFEBEE", opacity: 0.45 }}
      text={{ content: "Low" }}
    />
    <ChartStripLine
      visible={true}
      range={{ start: 50, end: 100 }}
      style={{ color: "#FFF8E1", opacity: 0.45 }}
      text={{ content: "Medium" }}
    />
    <ChartStripLine
      visible={true}
      range={{ start: 100, end: 150 }}
      style={{ color: "#E8F5E9", opacity: 0.45 }}
      text={{ content: "High" }}
    />
  </ChartStripLines>
</ChartPrimaryYAxis>
```

Keep ranges ordered and non-overlapping unless overlapping bands are intentionally requested.

## Repeated striplines

Repeated striplines create evenly spaced bands or lines across an axis. Configure recurrence through the `repeat` object.

```tsx
<ChartStripLine
  visible={true}
  range={{
    start: new Date(2026, 0, 1),
    end: new Date(2026, 0, 2),
  }}
  repeat={{
    enable: true,
    every: 7,
    until: new Date(2026, 2, 31),
  }}
  style={{ color: "#E3F2FD", opacity: 0.3 }}
/>
```

Use repeat values compatible with the owning axis (`every` as days for a DateTime axis, numeric units for a numeric axis).

## Layering

Use `style.zIndex` to render the stripline behind or over the series.

Typical behavior:

- `Behind`: use for contextual ranges that should not obscure data.
- `Over`: use only for emphasized overlays with sufficient transparency.

```tsx
<ChartStripLine
  visible={true}
  range={{ start: 100, end: 120 }}
  style={{ color: "#E8F5E9", opacity: 0.4, zIndex: "Behind" }}
/>
```

Do not obscure series values with a fully opaque foreground band.

## Complete horizontal stripline example

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
  ChartStripLine,
  ChartStripLines,
} from "@syncfusion/react-charts";

const data = [
  { month: "Jan", value: 72 },
  { month: "Feb", value: 88 },
  { month: "Mar", value: 104 },
  { month: "Apr", value: 117 },
  { month: "May", value: 96 },
];

export default function HorizontalStriplineChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis valueType="Category">
        <ChartAxisTitle text="Month" />
        <ChartAxisLabel edgeLabelPlacement="Shift" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis
        valueType="Double"
        minimum={0}
        maximum={140}
        interval={20}
      >
        <ChartAxisTitle text="Performance" />
        <ChartAxisLabel format="{value}" />

        <ChartStripLines>
          <ChartStripLine
            visible={true}
            range={{ start: 100, end: 120 }}
            style={{
              color: "#E8F5E9",
              opacity: 0.5,
              border: {
                color: "#2E7D32",
                width: 1,
                dashArray: "4,4",
              },
            }}
            text={{
              content: "Target range",
              hAlign: "Right",
              vAlign: "Top",
              font: {
                color: "#1B5E20",
                fontSize: "12px",
                fontWeight: "Bold",
                fontFamily: "Arial",
                fontStyle: "Normal",
                opacity: 1,
              },
            }}
          />
        </ChartStripLines>
      </ChartPrimaryYAxis>

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="value"
          type="Line"
          name="Performance"
          width={2}
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Complete DateTime stripline example

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
  ChartStripLine,
  ChartStripLines,
} from "@syncfusion/react-charts";

const data = [
  { date: new Date(2026, 0, 1), value: 35 },
  { date: new Date(2026, 1, 1), value: 42 },
  { date: new Date(2026, 2, 1), value: 38 },
  { date: new Date(2026, 3, 1), value: 51 },
];

export default function DateTimeStriplineChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis
        valueType="DateTime"
        interval={1}
        intervalType="Months"
      >
        <ChartAxisTitle text="Month" />
        <ChartAxisLabel format="MMM" edgeLabelPlacement="Shift" />

        <ChartStripLines>
          <ChartStripLine
            visible={true}
            range={{
              start: new Date(2026, 0, 1),
              end: new Date(2026, 2, 31),
            }}
            style={{ color: "#E3F2FD", opacity: 0.45 }}
            text={{
              content: "First quarter",
              hAlign: "Center",
              vAlign: "Top",
              font: {
                color: "#0D47A1",
                fontSize: "12px",
                fontWeight: "Bold",
                fontFamily: "Arial",
                fontStyle: "Normal",
                opacity: 1,
              },
            }}
          />
        </ChartStripLines>
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double">
        <ChartAxisTitle text="Value" />
        <ChartAxisLabel format="{value}" />
      </ChartPrimaryYAxis>

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="date"
          yField="value"
          type="Line"
          name="Value"
          width={2}
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Common errors

### Wrong casing

Incorrect:

```tsx
<ChartStriplines>
  <ChartStripLine />
</ChartStriplines>
```

Correct:

```tsx
<ChartStripLines>
  <ChartStripLine />
</ChartStripLines>
```

### Props on the collection wrapper

Props placed directly on `ChartStripLines` are ignored. Move them onto each `ChartStripLine` child.

Incorrect:

```tsx
<ChartStripLines range={{ start: 100, end: 120 }} />
```

Correct:

```tsx
<ChartStripLines>
  <ChartStripLine range={{ start: 100, end: 120 }} />
</ChartStripLines>
```

### Flat EJ2-style properties

`color`, `opacity`, `border`, `dashArray`, `zIndex`, `hAlign`, `vAlign`, `rotation`, and `isRepeat`/`repeatEvery`/`repeatUntil` are not direct `ChartStripLine` props. They live in the nested `style`, `text`, and `repeat` objects.

Incorrect:

```tsx
<ChartStripLine color="#FFE5E5" opacity={0.5} hAlign="Right" />
```

Correct:

```tsx
<ChartStripLine
  style={{ color: "#FFE5E5", opacity: 0.5 }}
  text={{ content: "Target", hAlign: "Right" }}
/>
```

### Invalid sizeType

Incorrect:

```tsx
<ChartStripLine range={{ start: 100, size: 25, sizeType: "Axis" }} />
```

Correct:

```tsx
<ChartStripLine range={{ start: 100, size: 25, sizeType: "Auto" }} />
```

## Validation checklist

Before returning a stripline implementation:

1. Import `ChartStripLines` and `ChartStripLine` with correct casing.
2. Place `ChartStripLines` inside the owning axis.
3. Place every `ChartStripLine` inside the `ChartStripLines` collection.
4. Set `visible` on each `ChartStripLine` that should render.
5. Configure appearance through the `style` object, not flat props.
6. Configure labels through the `text` object with `content` and `font`.
7. Configure recurrence through the `repeat` object with `enable`, `every`, and `until`.
8. Use only verified `sizeType` and `zIndex` literals.
9. Use range values compatible with the owning axis type.
10. Ensure all imports are used.
11. Emit valid, unescaped TSX.
