# Globalization Reference

Use this guidance when a Pure React Chart must display dates, numbers, currencies, percentages, labels, and translated text according to the active locale. Keep locale handling consistent across axes, data labels, tooltips, annotations, summaries, and fallback content.

## Core rules

- Keep raw numeric values as numbers and temporal values as `Date` objects.
- Format values at the presentation boundary rather than storing formatted strings as chart data.
- Use one active locale across axis labels, data labels, tooltips, and nearby text.
- Use `Intl.NumberFormat` and `Intl.DateTimeFormat` when custom locale-aware output is required.
- Use the chart's `format`, `formatter`, and `skeleton` APIs where they satisfy the requirement.
- Allow translated labels enough width and configure wrapping, trimming, or intersection handling deliberately.
- Do not assume that changing the text direction also changes number, date, or currency formatting.

## Locale source

Keep the active locale in application state, context, routing, or the application's internationalization library.

```tsx
const locale = "en-IN";
```

Use valid BCP 47 locale tags such as:

- `en-IN`
- `ta-IN`
- `fr-FR`
- `de-DE`
- `ar-SA`

Do not hard-code different locales in separate chart elements unless the requirement explicitly calls for mixed-locale output.

## Number formatting

Use `Intl.NumberFormat` for locale-aware grouping, decimal separators, and digit rules.

```tsx
const numberFormatter = new Intl.NumberFormat("en-IN", {
  maximumFractionDigits: 1,
});

numberFormatter.format(125000.5);
```

Use the same formatter in chart callbacks and nearby summaries.

## Axis number formatting

`ChartAxisLabel` supports global format strings and a formatter callback. The documented formatter signature is:

```tsx
(value: number, text: string) => string | boolean
```

```tsx
const numberFormatter = new Intl.NumberFormat(locale, {
  maximumFractionDigits: 1,
});

<ChartPrimaryYAxis valueType="Double">
  <ChartAxisTitle text="Sales" />
  <ChartAxisLabel
    formatter={(value) => numberFormatter.format(value)}
  />
</ChartPrimaryYAxis>
```

Use the provided `text` instead when the chart's existing formatting must be retained and only a prefix or suffix is required.

```tsx
<ChartAxisLabel
  formatter={(value, text) => `${text} units`}
/>
```

## Currency formatting

Use an explicit currency code. A locale does not determine the business currency by itself.

```tsx
const currencyFormatter = new Intl.NumberFormat("en-IN", {
  style: "currency",
  currency: "INR",
  maximumFractionDigits: 0,
});

<ChartPrimaryYAxis valueType="Double">
  <ChartAxisTitle text="Revenue" />
  <ChartAxisLabel
    formatter={(value) => currencyFormatter.format(value)}
  />
</ChartPrimaryYAxis>
```

Do not build currency text by concatenating a symbol to a number. Symbol position, spacing, grouping, and decimal rules vary by locale.

## Percentage formatting

`Intl.NumberFormat` with `style: "percent"` expects fractional values.

```tsx
const percentageFormatter = new Intl.NumberFormat(locale, {
  style: "percent",
  maximumFractionDigits: 1,
});

percentageFormatter.format(0.425); // 42.5% in a compatible locale
```

If the source stores `42.5` to mean `42.5%`, divide by 100 before passing the value to a percent formatter or use a deliberate text formatter.

```tsx
<ChartAxisLabel
  formatter={(value) => percentageFormatter.format(value / 100)}
/>
```

## DateTime data

Bind real `Date` objects when the X-axis represents time.

```tsx
const data = [
  { date: new Date(2026, 0, 1), sales: 120 },
  { date: new Date(2026, 1, 1), sales: 145 },
  { date: new Date(2026, 2, 1), sales: 138 },
];

<ChartPrimaryXAxis
  valueType="DateTime"
  interval={1}
  intervalType="Months"
/>
```

Do not use preformatted date strings as category values when the chart should preserve real time spacing.

## Date axis labels

Use `ChartAxisLabel.format` or `skeleton` for supported built-in date formatting. Use the formatter callback when application-controlled locale output is required.

```tsx
const dateFormatter = new Intl.DateTimeFormat(locale, {
  month: "short",
  year: "numeric",
});

<ChartPrimaryXAxis
  valueType="DateTime"
  interval={1}
  intervalType="Months"
>
  <ChartAxisLabel
    formatter={(value) =>
      dateFormatter.format(new Date(value))
    }
  />
</ChartPrimaryXAxis>
```

The axis formatter receives a numeric value. Convert it to `Date` only for a DateTime axis.

## Data-label formatting

`ChartDataLabel` supports `format` and a formatter callback. Its documented formatter signature is:

```tsx
(index: number, text: string) => string | boolean
```

```tsx
<ChartMarker visible={true}>
  <ChartDataLabel
    visible={true}
    formatter={(index, text) => text}
  />
</ChartMarker>
```

The data-label formatter does not receive the numeric value directly. When locale formatting requires the original point value, use the point index with the corresponding data array or prepare a locale-formatted label field before rendering.

```tsx
const displayData = data.map((item) => ({
  ...item,
  salesLabel: currencyFormatter.format(item.sales),
}));

<ChartMarker visible={true}>
  <ChartDataLabel
    visible={true}
    labelField="salesLabel"
  />
</ChartMarker>
```

Keep the original numeric field for plotting and use a separate display field for localized text.

## Tooltip formatting

`ChartTooltip` supports `format`, `formatter`, and `template`. Its formatter receives a string or string array, not the original point object.

```tsx
const formatTooltip = (
  text: string | string[],
): string | string[] => {
  return Array.isArray(text)
    ? text.map((item) => item)
    : text;
};

<ChartTooltip
  enable={true}
  shared={true}
  formatter={formatTooltip}
/>
```

For full locale control over original X and Y values, use a tooltip template.

```tsx
import type {
  ChartTooltipTemplateProps,
} from "@syncfusion/react-charts";

const tooltipTemplate = (
  props: ChartTooltipTemplateProps,
) => (
  <div>
    <strong>
      {dateFormatter.format(new Date(props.x))}
    </strong>
    <div>{currencyFormatter.format(props.y)}</div>
  </div>
);

<ChartTooltip
  enable={true}
  template={tooltipTemplate}
/>
```

`ChartTooltipTemplateProps.x` and `.y` are numeric in the current API. Interpret `x` as a timestamp only when the owning series uses a DateTime X-axis.

## Shared formatting utilities

Create formatters once per locale and currency, then reuse them.

```tsx
import { useMemo } from "react";

const formatters = useMemo(() => {
  return {
    currency: new Intl.NumberFormat(locale, {
      style: "currency",
      currency,
      maximumFractionDigits: 0,
    }),
    number: new Intl.NumberFormat(locale, {
      maximumFractionDigits: 1,
    }),
    date: new Intl.DateTimeFormat(locale, {
      month: "short",
      year: "numeric",
    }),
  };
}, [currency, locale]);
```

Recreate formatters when the active locale or currency changes.

## Complete localized chart

```tsx
import { useMemo } from "react";
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
  ChartTitle,
  ChartTooltip,
} from "@syncfusion/react-charts";
import type {
  ChartTooltipTemplateProps,
} from "@syncfusion/react-charts";

const data = [
  { date: new Date(2026, 0, 1), revenue: 125000 },
  { date: new Date(2026, 1, 1), revenue: 148500 },
  { date: new Date(2026, 2, 1), revenue: 139750 },
  { date: new Date(2026, 3, 1), revenue: 172300 },
];

interface LocalizedChartProps {
  locale: string;
  currency: string;
}

export default function LocalizedChart({
  locale,
  currency,
}: LocalizedChartProps) {
  const currencyFormatter = useMemo(
    () =>
      new Intl.NumberFormat(locale, {
        style: "currency",
        currency,
        maximumFractionDigits: 0,
      }),
    [currency, locale],
  );

  const dateFormatter = useMemo(
    () =>
      new Intl.DateTimeFormat(locale, {
        month: "short",
        year: "numeric",
      }),
    [locale],
  );

  const displayData = useMemo(
    () =>
      data.map((item) => ({
        ...item,
        revenueLabel: currencyFormatter.format(item.revenue),
      })),
    [currencyFormatter],
  );

  const tooltipTemplate = (
    props: ChartTooltipTemplateProps,
  ) => (
    <div style={{ padding: "8px" }}>
      <strong>
        {dateFormatter.format(new Date(props.x))}
      </strong>
      <div>{currencyFormatter.format(props.y)}</div>
    </div>
  );

  return (
    <Chart
      accessibility={{
        ariaLabel: "Monthly revenue chart",
        role: "img",
        focusable: true,
        tabIndex: 0,
      }}
    >
      <ChartTitle text="Monthly revenue" />

      <ChartPrimaryXAxis
        valueType="DateTime"
        interval={1}
        intervalType="Months"
      >
        <ChartAxisTitle text="Month" />
        <ChartAxisLabel
          formatter={(value) =>
            dateFormatter.format(new Date(value))
          }
        />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double" minimum={0}>
        <ChartAxisTitle text="Revenue" />
        <ChartAxisLabel
          formatter={(value) =>
            currencyFormatter.format(value)
          }
        />
      </ChartPrimaryYAxis>

      <ChartTooltip
        enable={true}
        template={tooltipTemplate}
      />

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={displayData}
          xField="date"
          yField="revenue"
          type="Line"
          name="Revenue"
        >
          <ChartMarker
            visible={true}
            shape="Circle"
            width={7}
            height={7}
          >
            <ChartDataLabel
              visible={true}
              labelField="revenueLabel"
              position="Top"
            />
          </ChartMarker>
        </ChartSeries>
      </ChartSeriesCollection>
    </Chart>
  );
}
```

Translate visible titles, axis titles, series names, summaries, and accessibility labels through the application's localization system. `Intl` formats values but does not translate application text.

## Long translated labels

`ChartAxisLabel` supports wrapping, trimming, width limits, rotation, and intersection handling.

```tsx
<ChartPrimaryXAxis valueType="Category">
  <ChartAxisLabel
    enableWrap={true}
    maxLabelWidth={90}
    intersectAction="Wrap"
    edgeLabelPlacement="Shift"
  />
</ChartPrimaryXAxis>
```

Available intersection strategies include `None`, `Hide`, `Trim`, `Wrap`, `MultipleRows`, `Rotate45`, and `Rotate90`.

Choose the least destructive strategy that keeps labels readable. Do not rely on trimming when the hidden text is essential to interpretation.

## Right-to-left languages

Use a semantic direction wrapper and test the rendered chart.

```tsx
<div dir={isRtl ? "rtl" : "ltr"} lang={locale}>
  <LocalizedChart
    locale={locale}
    currency={currency}
  />
</div>
```

Text direction and locale are separate concerns:

- `dir` controls reading and layout direction.
- `lang` identifies the content language.
- `Intl` applies locale-specific value formatting.

Do not assume a chart-level `enableRtl` prop exists when it is absent from the current Pure React `Chart` API.

## Locale changes at runtime

Keep locale-dependent labels and formatter objects derived from the active locale.

```tsx
const [locale, setLocale] = useState("en-IN");

<select
  value={locale}
  onChange={(event) => setLocale(event.target.value)}
>
  <option value="en-IN">English</option>
  <option value="ta-IN">தமிழ்</option>
  <option value="de-DE">Deutsch</option>
</select>
```

When the locale changes, update:

- visible titles and series names
- axis label formatters
- data-label display fields
- tooltip templates
- accessibility labels
- nearby summaries and tables

## Time zones

Locale and time zone are different settings. Specify a time zone when the chart must display dates consistently across users.

```tsx
const dateFormatter = new Intl.DateTimeFormat(locale, {
  dateStyle: "medium",
  timeStyle: "short",
  timeZone: "Asia/Kolkata",
});
```

Document whether timestamps represent UTC, browser-local time, or a business-specific time zone. Avoid applying accidental shifts when parsing date-only strings.

## Common errors

### Storing formatted numbers as Y values

Incorrect:

```tsx
{ month: "Jan", revenue: "₹1,25,000" }
```

Correct:

```tsx
{ month: "Jan", revenue: 125000 }
```

Plot raw numbers and format only the displayed text.

### Manually adding currency symbols

Incorrect:

```tsx
formatter={(value) => `₹${value}`}
```

Correct:

```tsx
formatter={(value) => currencyFormatter.format(value)}
```

### Assuming tooltip formatter receives point data

The tooltip formatter receives generated text. Use a tooltip template for direct `x` and `y` access.

### Mixing locales

Do not format axes with one locale and tooltips with another unless the design explicitly requires it.

### Unhandled long labels

Configure wrapping, trimming, multiple rows, or rotation and test with the longest supported translation.

## Validation checklist

Before returning a globalization implementation:

1. Identify the active locale and currency separately.
2. Keep plotted numbers numeric and dates as valid `Date` values.
3. Use one locale across axes, labels, tooltips, and surrounding text.
4. Use `Intl.NumberFormat` for custom number, currency, and percentage output.
5. Use `Intl.DateTimeFormat` for custom date and time output.
6. Set an explicit time zone when required by the product.
7. Use the documented axis formatter signature `(value, text)`.
8. Use the documented data-label formatter signature `(index, text)`.
9. Use a separate localized `labelField` when data labels need original values.
10. Handle both string and string-array tooltip formatter input.
11. Use a tooltip template when original point values are required.
12. Recreate locale-dependent formatters when locale or currency changes.
13. Translate titles, series names, summaries, and accessibility labels separately.
14. Configure long-label wrapping or intersection handling.
15. Use `lang` and `dir` appropriately for translated and RTL content.
16. Do not assume an undocumented chart-level RTL property.
17. Test zero, negative, large, fractional, missing, and invalid values.
18. Test the longest supported labels and narrow layouts.
19. Do not mix EJ2 globalization APIs with Pure React components.
20. Ensure every imported symbol is used.
21. Emit valid, unescaped TSX.
