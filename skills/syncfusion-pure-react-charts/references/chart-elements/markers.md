# Markers Reference

Use markers to show individual data points in supported Cartesian series such as line, spline, area, and scatter charts. Configure markers with `ChartMarker` inside the owning `ChartSeries`.

## Required component hierarchy

```tsx
import {
  Chart,
  ChartMarker,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

<Chart>
  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="month"
      yField="value"
      type="Line"
    >
      <ChartMarker visible={true} />
    </ChartSeries>
  </ChartSeriesCollection>
</Chart>
```

Do not place `ChartMarker` directly under `Chart` or `ChartSeriesCollection`.

## Verified marker properties

The official `ChartMarkerProps` API documents:

- `visible: boolean`, default `false`
- `shape: ChartMarkerShape | null`, default `null`
- `width: number`, default `5`
- `height: number`, default `5`
- `fill: string | null`, default `""`
- `filled: boolean`, default `false`
- `border: ChartBorderProps`, default `{ color: "", width: 2, dashArray: "" }`
- `opacity: number`, default `1`
- `offset: ChartLocationProps`, default `{ x: 0, y: 0 }`
- `imageUrl: string`, default `""`
- `highlightable: boolean`, default `true`

`fill` accepts a valid CSS color and falls back to the series color when no explicit value is supplied. `opacity` accepts values from `0` through `1`. 

## Basic markers

Markers are hidden by default. Set `visible={true}` to render them.

```tsx
<ChartSeries
  dataSource={data}
  xField="month"
  yField="value"
  type="Line"
>
  <ChartMarker visible={true} />
</ChartSeries>
```

Use markers when individual points need to remain distinguishable. Avoid enabling dense markers automatically when they would obscure the series.

## Marker shapes

The official API documents these exact shape values:

- `Circle`
- `Rectangle`
- `Triangle`
- `Diamond`
- `Cross`
- `Plus`
- `HorizontalLine`
- `VerticalLine`
- `Pentagon`
- `InvertedTriangle`
- `Image`
- `Star`
- `None`

```tsx
<ChartMarker
  visible={true}
  shape="Circle"
  width={10}
  height={10}
/>
```

Use exact casing. Set `shape="None"` to disable the marker shape when that is preferable to removing the marker component. 

## Marker size

Use `width` and `height` directly on `ChartMarker`.

```tsx
<ChartMarker
  visible={true}
  width={12}
  height={12}
/>
```

Do not use a nested `size` object.

Incorrect:

```tsx
<ChartMarker
  size={{ width: 12, height: 12 }}
/>
```

Correct:

```tsx
<ChartMarker
  width={12}
  height={12}
/>
```

The documented defaults are `5` pixels for both width and height. 

## Fill and border

Use `fill` for marker color and `border` for its outline.

```tsx
<ChartMarker
  visible={true}
  shape="Circle"
  fill="#FF5733"
  border={{
    color: "#FFFFFF",
    width: 2,
    dashArray: "",
  }}
/>
```

The documented border default is:

```tsx
{
  color: "",
  width: 2,
  dashArray: "",
}
```

Use a valid CSS color. Use a comma-separated SVG-style string for `dashArray`. 

## Filled and hollow markers

Use `filled` to control whether the marker is filled using the corresponding series color.

```tsx
<ChartMarker
  visible={true}
  shape="Circle"
  filled={true}
/>
```

For a hollow-style marker, set `filled={false}` and configure the border as needed.

```tsx
<ChartMarker
  visible={true}
  shape="Circle"
  filled={false}
  border={{
    color: "#1565C0",
    width: 2,
    dashArray: "",
  }}
/>
```

The documented default of `filled` is `false`. 

## Marker opacity

Use `opacity` from `0` through `1`.

```tsx
<ChartMarker
  visible={true}
  opacity={0.8}
/>
```

- `0` is fully transparent.
- `1` is fully opaque.
- The documented default is `1`. 

## Marker offset

Use `offset` to move a marker relative to its data point.

```tsx
<ChartMarker
  visible={true}
  offset={{
    x: 0,
    y: -6,
  }}
/>
```

The documented default is `{ x: 0, y: 0 }`. Offset changes marker placement only; it does not change the point's data value. 

## Image markers

Set `shape="Image"` and provide `imageUrl`.

```tsx
<ChartMarker
  visible={true}
  shape="Image"
  imageUrl="/images/target-marker.png"
  width={16}
  height={16}
/>
```

`imageUrl` requires `shape="Image"`. Ensure the image URL is accessible to the application and that the marker has explicit dimensions appropriate for the chart. turn45search188

Do not set `imageUrl` with another marker shape and assume that the image will render.

## Highlight participation

Use `highlightable` to control whether markers participate in marker emphasis during hover or selection behavior.

```tsx
<ChartMarker
  visible={true}
  highlightable={true}
/>
```

The documented default is `true`. Set it to `false` only when markers should not receive that visual emphasis. 

## Data labels inside markers

Place `ChartDataLabel` inside `ChartMarker`.

```tsx
import {
  ChartDataLabel,
  ChartMarker,
} from "@syncfusion/react-charts";

<ChartSeries
  dataSource={data}
  xField="month"
  yField="value"
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
    />
  </ChartMarker>
</ChartSeries>
```

The required hierarchy is:

```text
ChartSeries
└── ChartMarker
    └── ChartDataLabel
```

Do not place `ChartDataLabel` alongside `ChartMarker` or directly under `ChartSeries`.

## Different markers for multiple series

Configure one marker for each series.

```tsx
<ChartSeriesCollection>
  <ChartSeries
    dataSource={actualData}
    xField="month"
    yField="value"
    type="Line"
    name="Actual"
  >
    <ChartMarker
      visible={true}
      shape="Circle"
      width={8}
      height={8}
      fill="#1565C0"
    />
  </ChartSeries>

  <ChartSeries
    dataSource={forecastData}
    xField="month"
    yField="value"
    type="Line"
    name="Forecast"
  >
    <ChartMarker
      visible={true}
      shape="Diamond"
      width={9}
      height={9}
      fill="#E65100"
    />
  </ChartSeries>
</ChartSeriesCollection>
```

Use shape and color together when series need clear visual distinction.

## Conditional marker customization

Do not attach `onPointRender` to `ChartSeries` unless the current Pure React API explicitly documents it there. The official root `Chart` API exposes `pointRender`, and its callback customizes individual rendered points. 

A verified root-level pattern is:

```tsx
import type { PointRenderProps } from "@syncfusion/react-charts";

const handlePointRender = (
  args: PointRenderProps,
): string => {
  const point = data[args.pointIndex];

  if (point?.value > 500) {
    return "#D32F2F";
  }

  return "#1565C0";
};

<Chart pointRender={handlePointRender}>
  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="category"
      yField="value"
      type="Line"
    >
      <ChartMarker visible={true} />
    </ChartSeries>
  </ChartSeriesCollection>
</Chart>
```

The official root API describes `pointRender` as a callback for customizing individual point color and shows a `PointRenderProps` argument type. Do not assume that the callback accepts mutation such as `args.marker = { ... }`; use the documented return contract. 

Incorrect without API verification:

```tsx
const handlePointRender = (args) => {
  args.marker = {
    size: { width: 15, height: 15 },
    fill: "#FF0000",
  };
};

<ChartSeries onPointRender={handlePointRender} />
```

If independently changing marker size per point is required, verify a dedicated point-marker field or event setting in the current API before generating code. Do not invent an `args.marker.size` object.

## Scatter markers

Scatter points are marker-driven. Configure their size and appearance with `ChartMarker`.

```tsx
<ChartSeries
  dataSource={data}
  xField="x"
  yField="y"
  type="Scatter"
>
  <ChartMarker
    visible={true}
    shape="Circle"
    width={10}
    height={10}
    fill="#00897B"
    border={{
      color: "#004D40",
      width: 1,
      dashArray: "",
    }}
  />
</ChartSeries>
```

Do not use `ChartSeries.size={{ width, height }}` for scatter marker dimensions. Use `ChartMarker.width` and `ChartMarker.height`.

## Complete example

```tsx
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartDataLabel,
  ChartMarker,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { month: "Jan", value: 35 },
  { month: "Feb", value: 42 },
  { month: "Mar", value: 38 },
  { month: "Apr", value: 51 },
  { month: "May", value: 47 },
];

export default function MarkersChart() {
  return (
    <Chart>
      <ChartPrimaryXAxis valueType="Category">
        <ChartAxisTitle text="Month" />
        <ChartAxisLabel edgeLabelPlacement="Shift" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double">
        <ChartAxisTitle text="Value" />
        <ChartAxisLabel format="{value}" />
      </ChartPrimaryYAxis>

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="value"
          type="Line"
          name="Value"
          width={2}
        >
          <ChartMarker
            visible={true}
            shape="Circle"
            width={10}
            height={10}
            fill="#1565C0"
            filled={true}
            border={{
              color: "#FFFFFF",
              width: 2,
              dashArray: "",
            }}
            opacity={1}
            offset={{ x: 0, y: 0 }}
            highlightable={true}
          >
            <ChartDataLabel
              visible={true}
              position="Top"
              format="{value}"
              fill="#FFFFFF"
              border={{
                color: "#D0D0D0",
                width: 1,
                dashArray: "",
              }}
              font={{
                fontFamily: "Arial",
                fontSize: "12px",
                fontStyle: "Normal",
                fontWeight: "Bold",
                color: "#222222",
                opacity: 1,
              }}
            />
          </ChartMarker>
        </ChartSeries>
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Common errors

### Wrong hierarchy

Incorrect:

```tsx
<Chart>
  <ChartMarker visible={true} />
</Chart>
```

Correct:

```tsx
<ChartSeries>
  <ChartMarker visible={true} />
</ChartSeries>
```

### Using a size object

Incorrect:

```tsx
<ChartMarker size={{ width: 10, height: 10 }} />
```

Correct:

```tsx
<ChartMarker width={10} height={10} />
```

### Wrong shape casing

Incorrect:

```tsx
<ChartMarker shape="circle" />
```

Correct:

```tsx
<ChartMarker shape="Circle" />
```

### Image URL without image shape

Incorrect:

```tsx
<ChartMarker imageUrl="/marker.png" />
```

Correct:

```tsx
<ChartMarker
  shape="Image"
  imageUrl="/marker.png"
/>
```

### Wrong data-label hierarchy

Incorrect:

```tsx
<ChartSeries>
  <ChartMarker visible={true} />
  <ChartDataLabel visible={true} />
</ChartSeries>
```

Correct:

```tsx
<ChartSeries>
  <ChartMarker visible={true}>
    <ChartDataLabel visible={true} />
  </ChartMarker>
</ChartSeries>
```

### Unverified series event and marker mutation

Do not use `ChartSeries.onPointRender` or assign `args.marker.size` unless the current Pure React API explicitly documents those members. The verified root-level callback is `Chart.pointRender`, with the return contract shown by the root API. 

## Validation checklist

Before returning a marker implementation:

1. Import `ChartMarker` from `@syncfusion/react-charts`.
2. Place `ChartMarker` inside the owning `ChartSeries`.
3. Set `visible={true}` when markers are requested.
4. Use only documented marker shapes with exact casing.
5. Use direct `width` and `height` props, not a nested `size` object.
6. Use `fill`, `filled`, `border`, and `opacity` according to their documented roles.
7. Keep opacity between `0` and `1`.
8. Use `offset={{ x, y }}` only to adjust visual placement.
9. Set `shape="Image"` when using `imageUrl`.
10. Place `ChartDataLabel` inside `ChartMarker`.
11. Configure a separate marker for each series that needs one.
12. Use the root `Chart.pointRender` event only according to its documented signature and return value.
13. Do not invent per-point marker-size mutation APIs.
14. Ensure every imported symbol is used.
15. Emit valid, unescaped TSX.
