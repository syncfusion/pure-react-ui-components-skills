---
name: data-binding
description: Data-binding patterns for the Syncfusion React Data Grid — local arrays, remote services via DataManager with ODataV4/WebApi/URL adaptors, custom `onDataRequest` for arbitrary backends, and the empty-record template. Load when wiring up the grid to a backend, configuring paging/sort/filter delegation, building a debounced live search, showing a no-data message, or integrating OData/WebApi adaptors.
---

# Data binding

The grid supports three binding modes: **local** (in-memory array), **remote** via `DataManager` adaptors (OData/WebApi/URL), and **custom** via the `onDataRequest` callback. Every mode uses the same `dataSource` prop on `<Grid>`.

## Local data

```tsx
import { Grid } from '@syncfusion/react-grid';

const [data, setData] = useState<Employee[]>([...]);

<Grid dataSource={data} />
```

When you need a state-driven lifecycle (re-fetch on dep change), use `useEffect`:

```tsx
useEffect(() => {
  fetch('/api/employees')
    .then(r => r.json())
    .then(setData);
}, []);
```

Lifts `dataSource` from JSX to effect-driven state, but the grid just receives the array — it doesn't care how the array was obtained.

## Remote data via `DataManager`

```tsx
import { DataManager, ODataV4Adaptor, Query } from '@syncfusion/react-data';

const data = new DataManager({
  url: 'https://services.odata.org/V4/Northwind/Northwind.svc/Orders',
  adaptor: new ODataV4Adaptor(),
  crossDomain: true,
  headers: [{ 'Syncfusion': 'true' }],
});

const query = new Query().addParams('dataCount', '100000');

<Grid<EmployeeServerData>
  dataSource={data}
  query={query}
  height="100%"
  pageSettings={{ enabled: true, pageSize: 50 }}
  filterSettings={{ enabled: true, type: 'Excel' }}
  sortSettings={{ enabled: true }}
  virtualizationSettings={{ enabled: true, scrollMode: ScrollMode.Virtual, viewPortBuffer: { rows: 5, columns: 5 } }}
  clipMode={ClipMode.EllipsisWithTooltip}
/>
```

Built-in adaptors:

| Class | Constructor | Server contract |
|---|---|---|
| `UrlAdaptor` | `new UrlAdaptor()` | POST body `{ skip, take, sorted, filters, requiresCounts, group, ... }` → response `{ result, count }` |
| `ODataV4Adaptor` | `new ODataV4Adaptor()` | GET with `$skip/$top/$orderby/$filter/$count=true` → `{ value, '@odata.count': "..." }` |
| `WebApiAdaptor` | `new WebApiAdaptor()` | OData V4-compatible contract (covers Web API endpoints) |

`DataManager` options: `url`, `adaptor`, `crossDomain`, `headers` (`Array<{[k:string]:string}>`), `data` (payload template), `requestType` (e.g., `'jsonp'`).

> For nested data (`field='Customer.CustomerID'`), call `Query.expand('Customer')` so the join is loaded alongside the parent rows.

### URL-adaptor skeleton

```js
POST /YourEndpoint
  { skip: 0, take: 12, sorted: [{ field: 'OrderID', direction: 'Ascending' }],
    filters: [{ field: 'CustomerID', operator: 'equal', value: 'VINET' }],
    requiresCounts: true, group: [], ... }
→ 200 OK { result: [/* rows */], count: 2155 }
```

### OData V4 skeleton

```
GET /Endpoint?$skip=0&$top=12&$orderby=OrderID asc&$filter=CustomerID eq 'VINET'&$count=true
→ 200 OK { value: [...], "@odata.count": "2155" }
```

## Custom backend via `onDataRequest`

For non-standard APIs (or any backend that the built-in `ODataV4Adaptor` / `WebApiAdaptor` / `UrlAdaptor` doesn’t satisfy), bind the grid through the `onDataRequest` event.

**Server response shape** (required):

```ts
{ result: Array<object>; count: number }
```

- `result` — records for the current page slice (e.g., 10 records when `take=10`).
- `count` — total records in the dataset across all pages; drives the pager’s total.

For infinite scrolling, also include `hasMore: boolean` (custom APIs) or rely on the grid’s auto-detected tokens (`@odata.nextLink`, `continuationToken`, `nextCursor`).

### Why custom binding?

- Backend API contract isn't OData V4 / WebApi / URL (no `$skip`/`$top`/`$filter`).
- Auth/auth headers / JWT / CSRF flow that’s bespoke.
- Multi-source aggregation (joins across services) the grid needs to honor.
- Lazy streaming with custom end-of-data cue.

### End-to-end `Fetch`-based wiring

This is the canonical `<Grid>` + `onDataRequest` + `useEffect` initial-load pattern from the docs. It uses `Fetch` from `@syncfusion/react-base` (uniform `send()` API for HTTP requests). Drop it into `App.tsx` as-is and adapt `BASE_URL` + the helper builders.

```tsx
import { useEffect, useState } from 'react';
import {
  Grid, Columns, Column,
  FilterSettings, SortSettings, PageSettings,
  DataRequestEvent, FilterBarType, ClipMode, TextAlign,
  SortDescriptor,
  FilterModule, PagerModule,
} from '@syncfusion/react-grid';
import { Fetch } from '@syncfusion/react-base';

const modules = { FilterModule, PagerModule };

interface GridData { result: Array<object>; count: number; }

export default function App() {
  const [data, setData] = useState<GridData>({ result: [], count: 0 });
  const [filterSettings] = useState<FilterSettings>({ enabled: true, type: 'FilterBar' });
  const [sortSettings]   = useState<SortSettings>({ enabled: true });
  const [pageSettings]   = useState<PageSettings>({ enabled: true, pageCount: 4, pageSize: 10 });

  const BASE_URL = 'https://services.odata.org/V4/Northwind/Northwind.svc/Orders';

  // Initial load: grid does NOT auto-fire onDataRequest on mount.
  usecallback/render-time pattern: call onDataRequest({skip:0, take:10}) once.
  useEffect(() => { renderComplete(); }, []);

  function renderComplete() {
    const state: DataRequestEvent = { skip: 0, take: 10 };
    onDataRequest(state);
  }

  // Called by the grid on paging/sort/filter, and by renderComplete() at mount.
  const onDataRequest = (state: DataRequestEvent | undefined) => {
    if (state) {
      const dataState: DataRequestEvent = {
        skip: state.skip ?? 0,
        take: state.take ?? 10,
        sort: state.sort
          ?.filter((s): s is { field: string; direction: string } => !!s.field)
          .map((s) => ({ field: s.field!, direction: s.direction })),
        where: state.where,
      };
      execute(dataState).then(setData);
    }
  };

  function execute(state: DataRequestEvent) { return getData(state); }

  return (
    <Grid dataSource={data}
          sortSettings={sortSettings} filterSettings={filterSettings} pageSettings={pageSettings}
          onDataRequest={onDataRequest} height="579px"
          modules={modules} enableDevMode={false}>
      <Columns>
        <Column field="OrderID"   headerText="Order ID"   width={100} filter={{ filterBarType: FilterBarType.NumericTextBox }} textAlign={TextAlign.Right} />
        <Column field="CustomerID" headerText="Customer ID" width={100} />
        <Column field="ShipName"  headerText="Ship Name"  width={150} clipMode={ClipMode.EllipsisWithTooltip} />
        <Column field="ShipCity"  headerText="Ship City"  width={120} />
        <Column field="ShipCountry" headerText="Ship Country" width={110} />
        <Column field="Freight"   headerText="Freight Charges" width={130} filter={{ filterBarType: FilterBarType.NumericTextBox }} textAlign={TextAlign.Right} format="C2" />
      </Columns>
    </Grid>
  );
}
```

### Initial data load — required `useEffect` hack

`onDataRequest` **does not auto-fire** when the grid mounts. To render data on first paint, call the handler manually inside `useEffect`:

```tsx
useEffect(() => { onDataRequest({ skip: 0, take: 10 }); }, []);
```

### `Fetch` helper (unified HTTP request)

```ts
import { Fetch } from '@syncfusion/react-base';

const fetchApi = Fetch(url, 'GET', 'application/json');
if (fetchApi && typeof fetchApi.send === 'function') {
  return fetchApi.send().then(async (response: Response) => { /* … */ });
}
```

`Fetch(url, method, contentType)` returns an object with `.send(): Promise<Response>`. Use it for consistency or substitute `fetch(url)` if you prefer the browser-native API.

### Reusable translators — `buildPageQuery` / `buildSortQuery` / `buildFilterQuery`

The OData-flavoured helpers below map the grid’s state into query strings. They’re reusable across `onDataRequest` and the initial `useEffect` call.

```ts
function buildPageQuery(state: DataRequestEvent): string {
  return `$skip=${state.skip}&$top=${state.take}`;
}

function buildSortQuery(state: DataRequestEvent): string {
  if (state.sort?.length) {
    return (
      `&$orderby=` + state.sort
        .map((obj: SortDescriptor) =>
          obj.direction?.toLowerCase() === 'descending' ? `${obj.field} desc` : obj.field)
        .reverse()
        .join(',')
    );
  }
  return '';
}

function buildFilterQuery(state: DataRequestEvent): string {
  if (state.where?.length) {
    const rawPredicates = state.where[0].predicates;
    // predicates can be array OR a single predicate object — guard both
    const predicates = Array.isArray(rawPredicates) ? rawPredicates : [rawPredicates];
    return (
      `&$filter=` + predicates.map((col) => {
        const value: string = (typeof col.value === 'string' ? `'${col.value}'` : col.value) as string;
        if (col.operator === 'startsWith') return `startswith(tolower(${col.field}), ${value.toLowerCase()})`;
        if (col.operator === 'equal')      return `${col.field} eq ${value}`;
        if (col.operator === 'contains')   return `contains(tolower(${col.field}), ${value.toLowerCase()})`;
        return '';
      }).join(' and ')
    );
  }
  return '';
}

function getData(state: DataRequestEvent): Promise<GridData> {
  const url = `${BASE_URL}?${buildPageQuery(state)}${buildSortQuery(state)}${buildFilterQuery(state)}&$count=true`;
  const fetchApi = Fetch(url, 'GET', 'application/json');
  if (fetchApi && typeof fetchApi.send === 'function') {
    return fetchApi.send().then(async (response: Response) => {
      const v = response as unknown as { value: Array<object>; '@odata.count': string };
      return { result: v.value, count: parseInt(v['@odata.count'], 10) };
    });
  }
  return Promise.resolve({ result: [], count: 0 });
}
```

`$count=true` (or `requiresCounts: true` on the request) is what tells OData services to return the unpaginated total so the pager has a real `count`.

### `DataRequestEvent` shape (full)

```ts
{
  skip?: number;
  take?: number;
  requiresCounts?: boolean;                       // pager count is authoritative when true
  sort?: Array<{ field: string; direction: 'Ascending' | 'Descending' }>;
  where?: Array<{
    isComplex?: boolean;
    condition?: 'and' | 'or';
    predicates: Array<{
      field: string;
      operator: 'equal' | 'startsWith' | 'contains' | 'greaterThan' | ...;
      value: unknown;
    }> | {
      field: string;
      operator: string;
      value: unknown;
    };
  }>;
  search?: Array<{ fields?: string[]; value: string; operator?: string }>;
  select?: string[];
  distinct?: any[];
  distinctCounts?: boolean;
  requestType?: string;                           // e.g., 'filterChoiceRequest' for Excel filter combobox
}
```

> Predicate shape: `state.where[0].predicates` can be either a single `{field, operator, value}` object **or** an array of those objects — guard with `Array.isArray(...)` before iterating (see `buildFilterQuery` above).

### Debounced live search (combined mode)

```tsx
import { debounce } from '@syncfusion/react-base';

const debouncedFilter = useMemo(
  () => debounce((value: string) => {
    gridRef.current?.filterByColumn('Product', 'contains', value);
  }, 300),
  [gridRef]
);
useEffect(() => { debouncedFilter?.(searchTerm); }, [searchTerm]);
```

## Empty record template

```tsx
<Grid
  dataSource={[]}
  emptyRecordTemplate={() => (
    <div className='sf-empty-record-template'>
      <img src={emptyImage} alt="No records" />
      <span>There is no data available to display.</span>
    </div>
  )}
  enableDevMode={false}
/>
```

Renders when `dataSource` resolves to an empty result. Override it for empty, no-results, error states, etc.

## Lifecycle events

| Event | Fires when |
|---|---|
| `onGridRenderStart` | Before row/column/header render begins |
| `onDataLoad` | After data is fetched and DOM initialized |
| `onGridRenderComplete` | After full DOM render |
| `onError` | On rendering/data error (args typed `Error`) |
| `onDataRequest` | Custom data request — paginate/sort/filter from your backend. **Not fired on mount**; trigger the first call from `useEffect`. |

## Common patterns

| Use case | Mode |
|---|---|
| Fully client-side grid | `dataSource={array}` |
| OData endpoint | `DataManager` + `ODataV4Adaptor` |
| ASP.NET WebApi | `DataManager` + `WebApiAdaptor` |
| RESTful mock | `DataManager` + `UrlAdaptor` + server-side POST handler |
| Arbitrary backend with JWT auth | `onDataRequest` handler with your own fetch |
| Streaming / infinite scroll | local large array + `infinite-scroll` settings (see `references/performance-and-scrolling.md`) |

## Custom-binding prerequisites (preflight)

Before shipping an `onDataRequest` integration, run through these checks — every one of them is a known silent-failure source.

1. **Initial-load hack**: confirm `useEffect(() => onDataRequest({skip:0, take:pageSize}), [])` exists. Missing it → empty grid on first render.
2. **`isPrimaryKey` column present** if the grid also accepts edits.
3. **Server response contract**: `{ result: T[], count: number }` (plus `hasMore` if using infinite). Mismatched shape → blank screen or wrong pager.
4. **Custom server search**: forward `state.search?.[0]?.value` to a `q` or `$search` parameter (the build helpers above don’t include `search`).
5. **`requiresCounts: true`** forwarded to the backend → otherwise pager total falls back to `result.length` × current-page estimate, which multiplies by `take` for total in some adaptors.
6. **Errors surfaced**: wrap `fetch`/`Fetch.send` in `try/catch` and call back into `onError`, not silently swallowed.
7. **`Fetch` + custom headers** are passed at request time. There’s no global header store on `Fetch`; for auth, build headers inline.
8. **Predicate shape guard**: `state.where[0].predicates` may be a single object or array. `Array.isArray(rawPredicates) ? rawPredicates : [rawPredicates]` is the canonical pattern.
9. **OData path**: `$count=true` is required for the pager to know total records; the OData response must include `@odata.count` for `parseInt` to succeed.
10. **Nested fields** on remote: even in custom binding, dotted `field` paths (`Name.FirstName`) imply the server is expected to return nested objects. Build unwrap logic for those server payloads.

## Constraints & guardrails

- **Detection gap**: missing `DataManager.crossDomain`, missing CSRF/JWT header, or missing `useEffect` initial-load are the three most common silent failures when wiring remote data. Confirm exact server contract first.
- **Integration check**: when wiring remote data, register the data-loading into the same at-mount + change-driven lifecycle as the rest of your app, and ensure errors surface through `onError` (don't swallow them).
- **Trigger event forwarding**: `setCellValue`/`setRowData` accept a fourth boolean to commit to the underlying source; default updates only the UI. With `DataManager`, prefer the `DataManager.saveChanges`/`remove`/`insert` method path instead.
- **Empty-record template**: don't use it as a controlled loading state (loading ≠ empty). Show a `Skeleton` overlay inside a `template` while `args.data === undefined`.
- **Nested paths on remote**: ensure the backend returns nested objects for dotted `field` columns; otherwise the column will render as `undefined`. (`Query.expand('Path')` is the `DataManager` flow; for `onDataRequest` you build the same expansion in your server payload.)
- **High-risk operations**: bulk remote deletes (`{isSelectAll: true, primaryKeys: []}`) and `clearFilter([])` should pass through confirmation dialogs before triggering server-side cascades.