# Multi-Level Labels Reference

Use multi-level labels to group axis labels under higher-level headings, for example months grouped into quarters, products grouped into categories, or days grouped into weeks. Each level is drawn as an extra band below (or beside) the normal axis labels.

## Component hierarchy

Place `ChartMultiLevelLabels` inside the axis it decorates (`ChartPrimaryXAxis`, `ChartPrimaryYAxis`, or a named `ChartAxis`). Each `ChartMultiLevelLabel` child is one level; its `categories` array defines the groups on that level.

```tsx
import {
  Chart,
  ChartMultiLevelLabel,
  ChartMultiLevelLabels,
  ChartPrimaryXAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

const data = [
  { month: "Jan", sales: 35 }, { month: "Feb", sales: 28 }, { month: "Mar", sales: 34 },
  { month: "Apr", sales: 32 }, { month: "May", sales: 40 }, { month: "Jun", sales: 32 },
  { month: "Jul", sales: 35 }, { month: "Aug", sales: 55 }, { month: "Sep", sales: 38 },
  { month: "Oct", sales: 30 }, { month: "Nov", sales: 25 }, { month: "Dec", sales: 32 },
];

export default function QuarterlyGrouping() {
  return (
    <Chart>
      <ChartPrimaryXAxis valueType="Category">
        <ChartMultiLevelLabels>
          {/* Level 1: quarters */}
          <ChartMultiLevelLabel
            border={{ width: 1, color: "#9CA3AF" }}
            categories={[
              { start: -0.5, end: 2.5, text: "Q1" },
              { start: 2.5, end: 5.5, text: "Q2" },
              { start: 5.5, end: 8.5, text: "Q3" },
              { start: 8.5, end: 11.5, text: "Q4" },
            ]}
          />
          {/* Level 2: half-years */}
          <ChartMultiLevelLabel
            textStyle={{ fontWeight: "600" }}
            categories={[
              { start: -0.5, end: 5.5, text: "H1 2026" },
              { start: 5.5, end: 11.5, text: "H2 2026" },
            ]}
          />
        </ChartMultiLevelLabels>
      </ChartPrimaryXAxis>

      <ChartSeriesCollection>
        <ChartSeries dataSource={data} xField="month" yField="sales" type="Column" name="Sales" />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

The first `ChartMultiLevelLabel` is drawn closest to the axis labels; each later level is drawn further out.

## `ChartMultiLevelLabel` props

- `categories: ChartMultiLevelLabelCategoryProps[]`: the groups on this level
- `alignment: "Left" | "Center" | "Right"`, default `"Center"`
- `overflow: "None" | "Wrap" | "Trim"`, default `"Wrap"`: how text longer than the group width is handled
- `textStyle: ChartFontProps`: `fontSize`, `fontWeight`, `fontStyle`, `fontFamily`, `color`, `opacity`
- `border: ChartBorderProps`, default `{ color: "", width: 0, dashArray: "" }`

## Category props

- `text: string`: label shown for the group
- `start: number | string | Date`: start value of the group on the axis
- `end: number | string | Date`: end value of the group on the axis
- `maximumTextWidth?: number`: maximum text width in pixels before `overflow` applies

`start` and `end` are axis values, which depend on the axis `valueType`:

| Axis `valueType` | `start` / `end` |
| --- | --- |
| `Category` | Point indexes. Use half-steps (`-0.5`, `2.5`, …) so the group spans the full width of its categories. |
| `Double` / `Logarithmic` | Numeric axis values |
| `DateTime` | `Date` objects or date strings (strings are parsed with `new Date(...)`) |

Do not pass category **names** (such as `"Jan"`) as `start`/`end` on a category axis; they are treated as date strings and the group will not render where expected.

## DateTime example

```tsx
<ChartPrimaryXAxis valueType="DateTime" intervalType="Months">
  <ChartMultiLevelLabels>
    <ChartMultiLevelLabel
      categories={[
        { start: new Date(2026, 0, 1), end: new Date(2026, 3, 1), text: "Q1" },
        { start: new Date(2026, 3, 1), end: new Date(2026, 6, 1), text: "Q2" },
      ]}
    />
  </ChartMultiLevelLabels>
</ChartPrimaryXAxis>
```

## Click events

Handle clicks on group labels with the root `onMultiLevelLabelClick` prop.

```tsx
import type { MultiLevelLabelClickEvent } from "@syncfusion/react-charts";

const handleGroupClick = (args: MultiLevelLabelClickEvent): void => {
  console.log(args.axisName, args.level, args.text, args.start, args.end);
};

<Chart onMultiLevelLabelClick={handleGroupClick}>{/* ... */}</Chart>
```

`MultiLevelLabelClickEvent` exposes `axisName`, `text`, `level`, `start`, and `end`. A common use is to zoom or filter to the clicked group.

## Common errors

- Placing `ChartMultiLevelLabel` directly in the axis without the `ChartMultiLevelLabels` wrapper.
- Placing `ChartMultiLevelLabels` under `Chart` instead of inside an axis.
- Using a `text` prop on `ChartMultiLevelLabel` itself; group text belongs inside each `categories` entry.
- Overlapping groups on the same level.

## Validation checklist

1. `ChartMultiLevelLabels` is inside the axis it labels.
2. Each level is a `ChartMultiLevelLabel` with a `categories` array.
3. `start`/`end` use the axis value scale (indexes for `Category`).
4. Groups on one level do not overlap.
5. Event handlers use `onMultiLevelLabelClick` on `Chart`.
