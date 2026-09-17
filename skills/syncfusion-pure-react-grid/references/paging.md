---
name: paging
description: Pager configuration for the Syncfusion React Data Grid — enabling, page size, page count, custom pager template, programmatic navigation, paging in remote/custom data sources, and accessibility keyboard mapping. Load when the user asks to add pagination, change page size, build a custom pager, navigate pages programmatically, or understands server-driven paging.
---

# Paging

Paging is supported client-side (over an array) and server-side (over `DataManager` adaptors or `onDataRequest`). Default page size is **12** records.

## Enable

```tsx
import { Grid, PageSettings, SortSettings, FilterSettings, PagerModule, FilterModule } from '@syncfusion/react-grid';

const modules = { PagerModule, FilterModule };
const [sort]   = useState<SortSettings>({ enabled: true });
const [filter] = useState<FilterSettings>({ enabled: true });
const [page]   = useState<PageSettings>({ enabled: true, pageSize: 8, pageCount: 4 });

<Grid dataSource={data} sortSettings={sort} filterSettings={filter} pageSettings={page} modules={modules} />
```

`pageCount` is the number of pager buttons visible in the navigation (commonly 4, 8, 10). `pageSize` defaults to 12.

## `PageSettings` shape

```ts
{
  enabled: boolean;                         // default false
  pageSize?: number;                        // default 12
  currentPage?: number;                     // controlled initial page
  pageCount?: number;                       // number of pager buttons to render
  template?: (props: { currentPage: number; totalPages: number; totalRecordsCount: number }) => React.ReactElement;
  estimatedTotalRecordsCount?: number;      // hint for scrollbar size in Infinite scroll
}
```

When `dataSource` is a `DataManager` and paging is enabled, the grid automatically sends skip/take per page.

## Custom pager template

```tsx
import { NumericTextBox } from '@syncfusion/react-inputs';

const customTemplate = useCallback((props: Record<string, number>) => (
  <div className="custom-pager">
    <NumericTextBox
      min={1} max={props.totalPages}
      value={props.currentPage}
      onChange={(args) => gridRef.current?.goToPage(parseInt(args?.value as string, 10))}
    />
    <span>{props.currentPage} of {props.totalPages} pages ({props.totalRecordsCount} items)</span>
  </div>
), []);
```

Pass via `pageSettings.template`. The template receives only numeric props (no record-level context).

## Programmatic navigation

`gridRef.current?.goToPage(pageIndex: number)` — jumps to a 1-based page number.

```tsx
const jump = (inputValue: string) => gridRef.current?.goToPage(parseInt(inputValue, 10));
```

`gridRef.current?.pagerRef?.totalRecordsCount` is accessible when you need quota-style logic (e.g., reject adds when over 1000 rows).

## `onPageChange` event

`PageEvent`: `{ currentPage: number; previousPage: number }`. Fires after paging completes.

```tsx
const handlePage = useCallback((args: PageEvent) => {
  console.log(`Moved from page ${args.previousPage} to ${args.currentPage}`);
}, []);

<Grid onPageChange={handlePage} ... />
```

## Server-side paging (DataManager / onDataRequest)

With `DataManager`, paging is intrinsic — the grid adapts payload (OData `$skip/$top` or UrlAdaptor POST `{skip,take}`) and parses `{ result, count }` or `{ value, '@odata.count' }`.

With `onDataRequest`, your handler reads `args.skip`/`args.take` and must update `dataSource` to `{ result, count }`.

```tsx
// onDataRequest pattern
const handleDataRequest = useCallback((state: DataRequestEvent) => {
  setData(prev => prev); // trigger re-render with new data merged in
  fetch(`/api/data?skip=${state.skip}&take=${state.take}`)
    .then(r => r.json())
    .then(d => setData({ result: d.rows, count: d.total }));
}, []);
```

## Accessibility

The pager exposes a `navigation` ARIA role and supports keyboard:

- `←` / `PageUp` — previous page
- `→` / `PageDown` — next page
- `Home` (or `Ctrl+Alt+PageUp` / `Fn+←`) — first page
- `End` (or `Ctrl+Alt+PageDown` / `Fn+→`) — last page
- `Tab` / `Shift+Tab` — move focus across pager items
- `Enter` / `Space` — activate page

ARIA labels for the pager can be overridden through `L10n.load` keys `currentPageLabel`, `totalItemsLabel`, `firstPageTooltip`, `lastPageTooltip`, `nextPageTooltip`, `previousPageTooltip` (see `references/globalization.md`).

## Common patterns

| Need | Configuration |
|---|---|
| Default 12/page | `pageSettings={{ enabled: true }}` |
| Smaller pages | `pageSettings={{ enabled: true, pageSize: 8 }}` |
| Custom toolbar pager | `pageSettings={{ template: customTemplate }}` |
| Server paging | `DataManager` adapter or `onDataRequest` |
| Conditional reject-add when over quota | `gridRef.current?.pagerRef?.totalRecordsCount > 1000` |

## Constraints

- Pager listens after data + sort + filter have been applied. The actual rows visible are min(pageSize, total - skip).
- `pageSettings` does not affect virtualization — that's controlled by `virtualizationSettings`.
- When using `DataManager`, server response must include `count` for `pageCount` to render accurately. Use `estimatedTotalRecordsCount` as a hint.

## Cross-references

- For accessibility keyboard mapping & ARIA, see `references/accessibility.md`.
- For globalization keys (currentPageLabel etc.), see `references/globalization.md`.
- For server-side data wiring, see `references/data-binding.md`.