# Chart Migration

This section explains how to migrate the `Chart` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<Chart>` and the chart sub-components
(`<ChartSeries>`, `<ChartPrimaryXAxis>`, `<ChartLegend>`, `<ChartTooltip>`,
`<ChartTitle>`, …).

## Root props

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `focusBorderWidth` / `focusBorderColor` / `focusBorderMargin` | `focusOutline={{ width, color, offset }}` | Consolidated to a single object prop. |
| `isTransposed` | `transposed` | Renamed. |
| `chartArea={{...}}` | `<ChartArea />` child | Inline object → nested component. |

## Title

`<ChartComponent titleStyle={{...}}>` becomes a `<ChartTitle>` child with
direct props: `align`, `background`, `border`, `color`, `fontFamily`,
`fontSize`, `fontStyle`, `fontWeight`, `opacity`, `position`, `textOverflow`,
`x`, `y`, `text`. `titleStyle.textAlignment` → `align`, `titleStyle.size` →
`fontSize`, `titleStyle.position` → `position`, etc.

## Axes

`<PrimaryXAxis />` becomes `<ChartPrimaryXAxis />`. Inline axis objects
(grid lines, tick lines, label style, label, title, strip lines) become
nested components:

- `<ChartPrimaryXAxis>` → `<ChartMajorGridLines />`, `<ChartMajorTickLines />`,
  `<ChartMinorGridLines />`, `<ChartMinorTickLines />`,
  `<ChartAxisLabel />`, `<ChartAxisTitle />`, `<ChartStripLines>` + `<ChartStripLine />`,
  `<ChartScrollbar />`, `<ChartCrosshairTooltip />`,
  `<ChartMultiLevelLabels>` + `<ChartMultiLevelLabel />`.
- `crossesAt` / `crossesInAxis` / `placeNextToAxisLine` → `crossAt={{ value, axis, allowOverlap }}`.
- `primaryXAxis.isIndexed` → `indexed`, `primaryXAxis.isInversed` →
  `inverted`, `primaryXAxis.maximumLabels` → `maxLabelDensity`.
- `primaryXAxis.labelStyle.textAlignment` → `<ChartAxisLabel align="...">`,
  `labelStyle.size` → `fontSize`, `labelFormat` → `format`, `labelRotation`
  → `rotationAngle`, `labelPadding` → `padding`, `labelPosition` → `position`,
  `titlePadding` → `padding`, `titleRotation` → `rotationAngle`.

## Series

`<SeriesCollectionDirective>` / `<SeriesDirective>` → `<ChartSeriesCollection>`
/ `<ChartSeries>`.

Field-name renames on series:

| EJ2 React | Pure React |
| --- | --- |
| `colorName` | `colorField` |
| `size` | `sizeField` |
| `tooltipMappingName` | `tooltipField` |
| `xName` | `xField` |
| `yName` | `yField` |

Settings objects:

- `showNormalDistribution`, `binInterval` → `histogramSettings.*`
- `showMean`, `boxPlotMode`, `showOutliers` → `boxAndWhiskerSettings.*`
- `negativeFillColor`, `summaryFillColor` → `waterfallSettings.negativeColor`,
  `positiveColor`, `subTotalColor`, `totalColor`
- `connector` → `waterfallSettings.connectorLine`
- `intermediateSumIndexes`, `sumIndexes` → `waterfallSettings.*`
- `marker={{...}}` → `<ChartMarker />` child (`isFilled` → `filled`,
  `allowHighlight` → `highlightable`)
- `dataLabel={{...}}` → `<ChartDataLabel />` child (`rx`/`ry` →
  `borderRadius={{x, y}}`, `labelIntersectAction` → `intersectMode`, `name` →
  `labelField`, `angle` → `rotationAngle`, `alignment` → `textAlign`)
- `labelSettings={{...}}` → `<ChartSeriesLabel />` child
- `paretoOptions={{...}}` → `<ChartParetoOptions />` child
- `lastValueLabel={{...}}` → `<ChartLastValueLabel />` child
  (`lineColor`/`lineWidth`/`dashArray` → `lineStyle`)
- `polar drawType`/`type` mappings consolidated into a single `type`
  (`PolarStackingColumn`, `RadarLine`, …)

## Rows / columns / axes

- `rows: [...]` → `<ChartRows>` + `<ChartRow />`
- `columns: [...]` → `<ChartColumns>` + `<ChartColumn />`
- `axes: [...]` → `<ChartAxes>` + `<ChartAxis />`

## Legend

`legendSettings={{...}}` → `<ChartLegend />`. Property renames inside
(`alignment` → `align`, `isInversed` → `inversed`, `titleStyle.textAlignment`
→ `titleAlign`, `titleStyle.textOverflow` → `titleOverflow`,
`titleStyle.size` → `fontSize`).

## Tooltip

`tooltip={{...}}` → `<ChartTooltip />`. `header` → `headerText`,
`enableMarker` → `showMarker`, `textStyle.size` → `fontSize`.

## Zoom settings

`zoomSettings={{...}}` → `<ChartZoomSettings />`. `enablePan` → `pan`,
`showToolbar` + `toolbarItems` → `toolbar={{ visible, items }}`,
`enableMouseWheelZooming` → `mouseWheelZoom`, `enablePinchZooming` →
`pinchZoom`, `enableSelectionZooming` → `selectionZoom`.

## Stack labels / trendlines / indicators

- `stackLabels={{...}}` → `<ChartStackLabels />`. `rx`/`ry` → `borderRadius`,
  `font.textAlignment` → `align`, `font.size` → `fontSize`, `angle` →
  `rotationAngle`.
- `<TrendlinesDirective>` / `<TrendlineDirective>` →
  `<ChartTrendlineCollection>` / `<ChartTrendline />`. `fill` → `stroke`.
- `<IndicatorsDirective>` / `<IndicatorDirective>` →
  `<ChartIndicatorCollection>` / `<ChartIndicator />`.

## Scrollbar

`scrollbarSettings={{...}}` → `<ChartScrollbar />`. `height` → `thickness`,
`scrollbarRadius` → `thumbRadius`, `scrollbarColor` → `thumbColor`. Global
`zoomSettings.enableScrollbar` migrates to `<ChartZoomSettings enableScrollbar />`.

## Annotations

`<AnnotationsDirective>` / `<AnnotationDirective>` →
`<ChartAnnotationCollection>` / `<ChartAnnotation />`. `coordinateUnits` →
`coordinateUnit`, `horizontalAlignment` → `hAlign`, `verticalAlignment` →
`vAlign`.

## Crosshair / crosshair tooltip

`crosshair={{ enable, color, width, dashArray }}` → `<ChartCrosshair />`
with `enable`, `lineStyle`. `crosshairTooltip.enable` →
`<ChartCrosshairTooltip enable />`.

## Error bar / cross axis / selection / highlight

- `errorBar={{...}}` → `<ChartErrorBar />` (`errorBarColorMapping` →
  `errorBarColorField`).
- `crossesAt` / `crossesInAxis` / `placeNextToAxisLine` → `crossAt={{ value,
  axis, allowOverlap }}`.
- `selectionMode` / `allowMultiSelection` / `selectedDataIndexes` /
  `selectionPattern` → `<ChartSelection mode, allowMultiSelection,
  selectedDataIndexes, pattern />`.
- `highlightMode` / `highlightColor` / `highlightPattern` →
  `<ChartHighlight mode, fill, pattern />`.

## Events

All event props gain an `on` prefix and lose any `chart*` / `*Click` prefix
on the chart component itself:

| EJ2 React | Pure React |
| --- | --- |
| `axisLabelClick` | `onAxisLabelClick` |
| `chartMouseClick` | `onClick` |
| `legendClick` | `onLegendClick` |
| `chartMouseLeave` | `onMouseLeave` |
| `chartMouseMove` | `onMouseMove` |
| `pointClick` | `onPointClick` |
| `resized` | `onResize` |
| `zoomComplete` | `onZoomEnd` |
| `onZooming` | `onZoomStart` |