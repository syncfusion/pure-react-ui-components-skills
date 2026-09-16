# Remote Data Binding Reference

Use remote data binding when chart records must be loaded from an API, OData service, Web API, or another server-side source. Bind normalized records to `ChartSeries.dataSource`, or use Syncfusion `DataManager` with an adaptor supported by the service protocol.

## Supported approaches

The official Pure React Chart documentation supports two primary remote-data approaches:

1. Fetch records with React `useEffect` and the native Fetch API, then bind the resulting array to `ChartSeries.dataSource`.
2. Bind a Syncfusion `DataManager` configured with a service URL and an appropriate adaptor. 

Choose Fetch when the application needs direct control over request lifecycle, authentication, cancellation, response normalization, loading state, and error state. Choose `DataManager` when its adaptor model matches the backend protocol and query requirements.

## Required chart hierarchy

Remote loading does not change the chart hierarchy. Place every `ChartSeries` inside `ChartSeriesCollection`.

```tsx
import {
  Chart,
  ChartPrimaryXAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

<Chart>
  <ChartPrimaryXAxis valueType="Category" />

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

The root and series API remain the same after remote records are converted into a local array. The series still requires valid `dataSource`, `xField`, `yField`, and `type` values. 

## Fetch API pattern

Use `useEffect` to request the data, `useState` to store the normalized records, and `AbortController` to cancel the request when the component unmounts or the request dependency changes.

```tsx
import { useEffect, useState } from "react";
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

type ApiSalesRecord = {
  month?: unknown;
  sales?: unknown;
};

type SalesPoint = {
  month: string;
  sales: number;
};

function normalizeSalesRecord(record: ApiSalesRecord): SalesPoint | null {
  const month = typeof record.month === "string"
    ? record.month.trim()
    : "";
  const sales = Number(record.sales);

  if (!month || !Number.isFinite(sales)) {
    return null;
  }

  return { month, sales };
}

export default function RemoteSalesChart() {
  const [data, setData] = useState<SalesPoint[]>([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const controller = new AbortController();

    async function loadData() {
      setIsLoading(true);
      setError(null);

      try {
        const response = await fetch("/api/sales", {
          signal: controller.signal,
          headers: {
            Accept: "application/json",
          },
        });

        if (!response.ok) {
          throw new Error(
            `Request failed with status ${response.status}`,
          );
        }

        const payload: unknown = await response.json();

        if (!Array.isArray(payload)) {
          throw new Error("Expected the API response to be an array");
        }

        const normalized = payload
          .map((record) => normalizeSalesRecord(record as ApiSalesRecord))
          .filter((record): record is SalesPoint => record !== null);

        setData(normalized);
      } catch (requestError) {
        if (requestError instanceof DOMException &&
            requestError.name === "AbortError") {
          return;
        }

        setError(
          requestError instanceof Error
            ? requestError.message
            : "Unable to load chart data",
        );
      } finally {
        if (!controller.signal.aborted) {
          setIsLoading(false);
        }
      }
    }

    void loadData();

    return () => controller.abort();
  }, []);

  if (isLoading) {
    return <p>Loading chart data...</p>;
  }

  if (error) {
    return <p role="alert">Unable to load chart data: {error}</p>;
  }

  return (
    <Chart noDataTemplate="No sales data available">
      <ChartPrimaryXAxis valueType="Category">
        <ChartAxisTitle text="Month" />
        <ChartAxisLabel edgeLabelPlacement="Shift" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double" minimum={0}>
        <ChartAxisTitle text="Sales" />
        <ChartAxisLabel format="{value}" />
      </ChartPrimaryYAxis>

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="month"
          yField="sales"
          type="Column"
          name="Sales"
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

The official Pure React remote-data guide explicitly supports `useEffect` with the native Fetch API. 

## Loading, error, empty, and success states

Handle these states separately:

- Loading: the request has not completed.
- Error: the request or response validation failed.
- Empty: the request succeeded but no valid chart records remain.
- Success: normalized records are ready for binding.

Do not treat an empty successful response as a network error. Use the root chart's `noDataTemplate` for the empty chart state when required; the root API documents `noDataTemplate` for cases where all series data sources are empty. 

```tsx
<Chart noDataTemplate="No records available">
  <ChartSeriesCollection>
    <ChartSeries
      dataSource={data}
      xField="category"
      yField="value"
      type="Line"
    />
  </ChartSeriesCollection>
</Chart>
```

## Response normalization

Never bind an unknown response directly to `ChartSeries`. Extract the actual record array and convert fields into the types expected by the chart.

```tsx
type ApiRecord = {
  label?: unknown;
  amount?: unknown;
};

type ChartPoint = {
  category: string;
  value: number;
};

function normalizeRecord(record: ApiRecord): ChartPoint | null {
  const category = typeof record.label === "string"
    ? record.label.trim()
    : "";
  const value = Number(record.amount);

  if (!category || !Number.isFinite(value)) {
    return null;
  }

  return { category, value };
}
```

Bind the normalized names:

```tsx
<ChartSeries
  dataSource={data}
  xField="category"
  yField="value"
  type="Column"
/>
```

Do not keep `xField="label"` and `yField="amount"` after renaming those fields during normalization.

## Extracting wrapped response arrays

APIs often wrap records inside another object. Validate the wrapper before accessing it.

```tsx
type ApiEnvelope = {
  items?: unknown;
};

function extractItems(payload: unknown): unknown[] {
  if (Array.isArray(payload)) {
    return payload;
  }

  if (
    typeof payload === "object" &&
    payload !== null &&
    "items" in payload &&
    Array.isArray((payload as ApiEnvelope).items)
  ) {
    return (payload as ApiEnvelope).items as unknown[];
  }

  throw new Error("The API response does not contain a valid item array");
}
```

Do not assume every API uses `data`, `items`, `result`, or `value`. Use the response shape documented by the selected endpoint.

## DateTime data

Convert remote date values into valid `Date` objects when the X-axis uses `DateTime`.

```tsx
type ApiPoint = {
  timestamp?: unknown;
  reading?: unknown;
};

type TimePoint = {
  time: Date;
  value: number;
};

function normalizeTimePoint(record: ApiPoint): TimePoint | null {
  const time = new Date(String(record.timestamp ?? ""));
  const value = Number(record.reading);

  if (Number.isNaN(time.getTime()) || !Number.isFinite(value)) {
    return null;
  }

  return { time, value };
}
```

```tsx
<ChartPrimaryXAxis valueType="DateTime" />

<ChartSeries
  dataSource={data}
  xField="time"
  yField="value"
  type="Line"
/>
```

Use `Category` instead when date-looking labels should remain equally spaced and actual elapsed time should not affect position.

## Specialized series normalization

Remote response fields still must satisfy the selected series type.

### Range series

```tsx
type RangePoint = {
  day: string;
  low: number;
  high: number;
};

<ChartSeries
  dataSource={data}
  xField="day"
  low="low"
  high="high"
  type="RangeArea"
/>
```

### Financial series

```tsx
type FinancialPoint = {
  date: Date;
  open: number;
  high: number;
  low: number;
  close: number;
};

<ChartSeries
  dataSource={data}
  xField="date"
  open="open"
  high="high"
  low="low"
  close="close"
  type="Candle"
/>
```

### Bubble series

```tsx
type BubblePoint = {
  x: number;
  y: number;
  size: number;
};

<ChartSeries
  dataSource={data}
  xField="x"
  yField="y"
  size="size"
  type="Bubble"
/>
```

Validate all mapped numeric fields with `Number.isFinite`.

## Refetching when request inputs change

Put request inputs in the effect dependency list and cancel the previous request.

```tsx
useEffect(() => {
  const controller = new AbortController();

  async function loadData() {
    const response = await fetch(
      `/api/sales?region=${encodeURIComponent(region)}`,
      { signal: controller.signal },
    );

    if (!response.ok) {
      throw new Error(`Request failed with status ${response.status}`);
    }

    const payload = await response.json();
    setData(normalizeResponse(payload));
  }

  void loadData().catch((requestError) => {
    if (!(requestError instanceof DOMException &&
          requestError.name === "AbortError")) {
      setError(
        requestError instanceof Error
          ? requestError.message
          : "Unable to load chart data",
      );
    }
  });

  return () => controller.abort();
}, [region]);
```

Do not omit a changing request input from the dependency list.

## Authentication and headers

Pass only the headers required by the service.

```tsx
const response = await fetch("/api/sales", {
  signal: controller.signal,
  headers: {
    Accept: "application/json",
    Authorization: `Bearer ${accessToken}`,
  },
});
```

Do not hard-code secrets, access tokens, API keys, or credentials in the component source. Obtain credentials through the application's authentication and configuration flow.

## DataManager approach

The official Pure React Chart guide supports remote binding through Syncfusion `DataManager`. Configure a service `url` and an adaptor that matches the endpoint protocol. Built-in adaptor scenarios include custom URL/REST services, OData V4, and OData-based Web APIs.

Use imports from the current Pure React data package documented by the application version:

```tsx
import {
  DataManager,
  ODataV4Adaptor,
} from "@syncfusion/react-data";
```

```tsx
const manager = new DataManager({
  url: "https://service.example.com/odata/Orders",
  adaptor: new ODataV4Adaptor(),
});
```

Bind the manager to the series only when the current `ChartSeries.dataSource` type supports a `DataManager` instance:

```tsx
<ChartSeries
  dataSource={manager}
  xField="CustomerID"
  yField="Freight"
  type="Column"
/>
```

The official remote-data guide describes `@syncfusion/react-data` adaptors for this workflow. 

Do not import DataManager from legacy `@syncfusion/ej2-data` or use `@syncfusion/ej2-react-charts` in a Pure React sample unless the user explicitly requests the EJ2 wrapper architecture. The older EJ2 demo uses different packages and component names and must not be mixed with `@syncfusion/react-charts`.

## Choosing a DataManager adaptor

Select the adaptor according to the backend contract:

- URL adaptor: custom REST or service endpoints with the response shape expected by the adaptor.
- OData V4 adaptor: OData version 4 endpoints.
- Web API adaptor: compatible ASP.NET Web API or OData-oriented endpoints.
- Custom adaptor: service-specific request or response handling when built-in adaptors do not match.

The official documentation identifies DataManager adaptors as the communication layer between the chart and remote services.

Do not choose an adaptor based only on the URL suffix. Verify the service protocol and response format.

## DataManager response shape

The official URL-adaptor guidance expects a response containing `result` and `count` for server-driven operations. 

Conceptual response:

```json
{
  "result": [
    { "category": "A", "value": 42 },
    { "category": "B", "value": 58 }
  ],
  "count": 2
}
```

Do not wrap or rename the server response arbitrarily. Match the selected adaptor's documented contract.

## Fetch versus DataManager

Use Fetch when:

- The endpoint returns a custom JSON shape.
- The component must show explicit loading and error UI.
- The application controls cancellation, retries, headers, or authentication.
- Records need significant normalization before chart binding.

Use DataManager when:

- The backend matches a supported adaptor protocol.
- The application benefits from adaptor-generated requests and response processing.
- The chart should bind through the DataManager abstraction.

Do not use both approaches for the same series unless there is a specific architectural reason.

## Retry behavior

Retries must be bounded and deliberate. Do not create an unbounded retry loop.

```tsx
async function fetchWithRetry(
  url: string,
  signal: AbortSignal,
  attempts = 2,
): Promise<Response> {
  let lastError: unknown;

  for (let attempt = 1; attempt <= attempts; attempt += 1) {
    try {
      const response = await fetch(url, { signal });

      if (response.ok || response.status < 500) {
        return response;
      }

      lastError = new Error(
        `Request failed with status ${response.status}`,
      );
    } catch (error) {
      if (signal.aborted) {
        throw error;
      }

      lastError = error;
    }
  }

  throw lastError instanceof Error
    ? lastError
    : new Error("Unable to load chart data");
}
```

Retry only when appropriate for the service and request. Avoid automatically retrying authentication failures, validation errors, or other non-transient responses.

## Refreshing remote data

Use application-controlled refresh behavior rather than embedding an arbitrary polling interval into every sample.

```tsx
const [refreshKey, setRefreshKey] = useState(0);

useEffect(() => {
  // Load and normalize the remote records.
}, [refreshKey]);

<button
  type="button"
  onClick={() => setRefreshKey((value) => value + 1)}
>
  Refresh
</button>
```

For continuous monitoring, use a separate real-time chart pattern and clean up timers, subscriptions, or streams when the component unmounts.

## Complete normalized remote-data example

```tsx
import { useEffect, useState } from "react";
import {
  Chart,
  ChartAxisLabel,
  ChartAxisTitle,
  ChartLegend,
  ChartPrimaryXAxis,
  ChartPrimaryYAxis,
  ChartSeries,
  ChartSeriesCollection,
} from "@syncfusion/react-charts";

type ApiRecord = {
  period?: unknown;
  actual?: unknown;
  target?: unknown;
};

type ResultPoint = {
  period: string;
  actual: number;
  target: number;
};

function normalizeResponse(payload: unknown): ResultPoint[] {
  if (!Array.isArray(payload)) {
    throw new Error("Expected an array of result records");
  }

  return payload.flatMap((value) => {
    const record = value as ApiRecord;
    const period = typeof record.period === "string"
      ? record.period.trim()
      : "";
    const actual = Number(record.actual);
    const target = Number(record.target);

    if (
      !period ||
      !Number.isFinite(actual) ||
      !Number.isFinite(target)
    ) {
      return [];
    }

    return [{ period, actual, target }];
  });
}

export default function RemoteDataBindingChart() {
  const [data, setData] = useState<ResultPoint[]>([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const controller = new AbortController();

    async function loadResults() {
      setIsLoading(true);
      setError(null);

      try {
        const response = await fetch("/api/results", {
          signal: controller.signal,
          headers: {
            Accept: "application/json",
          },
        });

        if (!response.ok) {
          throw new Error(
            `Request failed with status ${response.status}`,
          );
        }

        const payload: unknown = await response.json();
        setData(normalizeResponse(payload));
      } catch (requestError) {
        if (requestError instanceof DOMException &&
            requestError.name === "AbortError") {
          return;
        }

        setError(
          requestError instanceof Error
            ? requestError.message
            : "Unable to load result data",
        );
      } finally {
        if (!controller.signal.aborted) {
          setIsLoading(false);
        }
      }
    }

    void loadResults();

    return () => controller.abort();
  }, []);

  if (isLoading) {
    return <p>Loading result data...</p>;
  }

  if (error) {
    return <p role="alert">{error}</p>;
  }

  return (
    <Chart noDataTemplate="No result data available">
      <ChartPrimaryXAxis valueType="Category">
        <ChartAxisTitle text="Period" />
        <ChartAxisLabel edgeLabelPlacement="Shift" />
      </ChartPrimaryXAxis>

      <ChartPrimaryYAxis valueType="Double" minimum={0}>
        <ChartAxisTitle text="Value" />
        <ChartAxisLabel format="{value}" />
      </ChartPrimaryYAxis>

      <ChartLegend
        visible={true}
        position="Bottom"
      />

      <ChartSeriesCollection>
        <ChartSeries
          dataSource={data}
          xField="period"
          yField="actual"
          type="Column"
          name="Actual"
        />
        <ChartSeries
          dataSource={data}
          xField="period"
          yField="target"
          type="Line"
          name="Target"
          width={2}
        />
      </ChartSeriesCollection>
    </Chart>
  );
}
```

## Common errors

### Binding an unresolved Promise

Incorrect:

```tsx
<ChartSeries
  dataSource={fetch("/api/sales")}
/>
```

Resolve and normalize the response first, or use a supported `DataManager` instance.

### Ignoring HTTP status

Incorrect:

```tsx
const payload = await fetch(url).then((response) => response.json());
```

Check `response.ok` before decoding a success payload.

### Binding unknown JSON directly

Incorrect:

```tsx
const payload = await response.json();
setData(payload);
```

Validate the response shape and convert field types before binding.

### DateTime axis with raw invalid strings

Do not map invalid or locale-ambiguous date strings to a `DateTime` axis. Normalize them into valid `Date` objects.

### Missing cancellation

Use `AbortController` when an effect-owned request may still be active during cleanup or dependency changes.

### Mixing Pure React and EJ2 APIs

Do not combine:

```tsx
import { Chart } from "@syncfusion/react-charts";
import { DataManager } from "@syncfusion/ej2-data";
import { ChartComponent } from "@syncfusion/ej2-react-charts";
```

Use the package family documented for the selected Pure React architecture. Older EJ2 examples use different components and mappings such as `ChartComponent`, `SeriesDirective`, `xName`, and `yName`.

### Hard-coded credentials

Do not include access tokens or secrets directly in source code.

## Validation checklist

Before returning a remote-data implementation:

1. Use either Fetch plus local state or a supported DataManager and adaptor.
2. Place every `ChartSeries` inside `ChartSeriesCollection`.
3. Handle loading, error, empty, and success states separately.
4. Check `response.ok` before processing a Fetch response.
5. Cancel effect-owned requests with `AbortController`.
6. Validate the top-level response shape.
7. Normalize remote field names and value types before chart binding.
8. Match every mapping to the normalized object shape.
9. Convert valid date strings into `Date` objects for DateTime axes.
10. Validate all numeric fields with `Number.isFinite`.
11. Supply all required specialized mappings for range, financial, bubble, and other series.
12. Keep changing request inputs in the effect dependency list.
13. Do not hard-code credentials or secrets.
14. Choose a DataManager adaptor that matches the backend protocol.
15. Do not mix Pure React and legacy EJ2 Chart APIs.
16. Keep retries bounded and appropriate to the failure type.
17. Clean up timers, streams, subscriptions, and requests.
18. Ensure all imported symbols are used.
19. Emit valid, unescaped TSX.
