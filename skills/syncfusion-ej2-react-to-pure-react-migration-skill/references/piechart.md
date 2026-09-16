# PieChart Migration

This section explains how to migrate the `PieChart` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<PieChart>` and the chart sub-components
(`<PieChartSeries>`, `<PieChartLegend>`, `<PieChartTooltip>`, …).

## Root props

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `enableSmartLabels` | `smartLabels` | Renamed. |
| `focusBorderWidth` / `focusBorderColor` / `focusBorderMargin` | `focusOutline={{ width, color, offset }}` | Consolidated. |
| `accessibility={{ accessibilityDescription, accessibilityRole, focusable, tabIndex }}` | `accessibility={{ ariaLabel, role, focusable, tabIndex }}` | Renamed key. |

## Title / subtitle

`<AccumulationChartComponent title="..." titleStyle={...}>` →
`<PieChartTitle>` child with `text`, `align`, etc. Subtitle works the
same way via `<PieChartSubtitle text="..." font={{...}} />`.

## Series

`<AccumulationSeriesCollectionDirective>` / `<AccumulationSeriesDirective>`
→ `<PieChartSeriesCollection>` / `<PieChartSeries>`. Field names:

| EJ2 React | Pure React |
| --- | --- |
| `xName` | `xField` |
| `yName` | `yField` |
| `pointColorMapping` | `colorField` |
| `tooltipMappingName` | `tooltipField` |

`enableBorderOnMouseMove` (component-level in EJ2) → `showBorderOnHover`
on the series.

## Data label

`dataLabel={{...}}` → `<PieChartDataLabel />` child.
`angle` → `rotationAngle`, `textRender` → `formatter(index, text)`,
`maxWidth` → `maxLabelWidth`.

## Legend

`legendSettings={{...}}` → `<PieChartLegend />`. `alignment` → `align`,
`titlePosition` → `titleAlign`, `maximumTitleWidth` → `maxTitleWidth`,
`maximumLabelWidth` → `maxLabelWidth`, `isInversed` → `inversed`,
`legendImageUrl` (series-level in EJ2) → `imageUrl` on legend, `legendShape`
→ `shape`.

## Tooltip

`tooltip={{...}}` → `<PieChartTooltip />`. `enableMarker` → `showMarker`,
`tooltipRender` → `formatter(text: string | string[])`.

## Center label

`centerLabel={{ text, font, hoverTextFormat }}` → `<PieChartCenterLabel />`
with `label={[{ text, textStyle: { fontSize } }]}` and `hoverTextFormat`.

## Annotations

`<AccumulationAnnotationsDirective>` / `<AccumulationAnnotationDirective>` →
`<PieChartAnnotationCollection>` / `<PieChartAnnotation />`. `coordinateUnits`
→ `coordinateUnit`, `horizontalAlignment` / `verticalAlignment` (deprecated
in EJ2) → `hAlign` / `vAlign`.

## Events

| EJ2 React | Pure React |
| --- | --- |
| `legendClick` | `onLegendClick` |
| `resized` | `onResize` |
| `pointClick` | `onPointClick` |
| `chartMouseClick` | `onClick` |
| `chartMouseMove` | `onMouseMove` |
| `chartMouseLeave` | `onMouseLeave` |