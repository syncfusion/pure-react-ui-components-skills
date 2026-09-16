---
name: searching
description: Toolbar search configuration for the Syncfusion React Data Grid — enable, fields restriction, operator (contains/startswith/endswith/wildcard/like/equal), case sensitivity, accent insensitivity, predefined value, programmatic search, onSearch lifecycle, and how search interacts with custom-data and remote DataManager backends. Load when adding a search box, building a global "find", debouncing search-as-you-type, or supporting wildcard/LIKE syntax in search.
---

# Searching

The grid ships a built-in toolbar search box. It searches across **visible and invisible** columns by default, with field-level constraints, multiple operators, sensitivity toggles, and programmable initial value.

## Enable

```tsx
import { Grid, SearchSettings, SortSettings, FilterSettings, PageSettings, FilterModule, PagerModule, SearchModule } from '@syncfusion/react-grid';

const modules = { FilterModule, PagerModule, SearchModule };
const [sort]   = useState<SortSettings>({ enabled: true });
const [filter] = useState<FilterSettings>({ enabled: true });
const [page]   = useState<PageSettings>({ enabled: true, pageSize: 8, pageCount: 4 });
const [search] = useState<SearchSettings>({ enabled: true });

<Grid
  dataSource={data}
  sortSettings={sort} filterSettings={filter} pageSettings={page} searchSettings={search}
  toolbar={['Search']}
  modules={modules}
/>
```

The `'Search'` toolbar item renders the search input on the toolbar. **Requires `SearchModule` in `modules`** (otherwise nothing renders).

## `SearchSettings` shape

```ts
{
  enabled: boolean;
  fields?: string[];                          // restrict to specific field names
  operator?: 'startswith' | 'endswith' | 'contains' | 'wildcard' | 'like' | 'equal';   // default 'contains'
  value?: string;                             // predefined search text at init
  ignoreCase?: boolean;                       // default true (case ignored unless caseSensitive)
  caseSensitive?: boolean;                    // default false
  ignoreAccent?: boolean;                     // default false
}
```

> Documentation examples mix lowercase (`contains`) and camelCase (`startsWith`) tokens depending on version. When passing strings, prefer the lowercase form. When in doubt, pass via the documented `SearchSettings.operator` typed field.

## Restrict fields

```tsx
searchSettings={{
  enabled: true,
  fields: ['BookID', 'Title', 'Author', 'Genre'],
}}
```

By default, search runs against all visible and invisible columns.

## Operators

| Operator | Description |
|---|---|
| `contains` (default) | Substring match. |
| `startswith` | Prefix match. |
| `endswith` | Suffix match. |
| `wildcard` | Use `*` patterns (`te*` matches "test", "team"). |
| `like` | Use `%` patterns (`%box` matches values ending with "box"). |
| `equal` | Exact match. |

## Accent insensitivity

```tsx
searchSettings: { enabled: true, ignoreAccent: true }
```

Treats "Stréét" as "Street". Useful for regional data.

## Predefined value at init

```tsx
searchSettings: { enabled: true, value: 'Dessert', operator: 'contains', caseSensitive: true, ignoreAccent: true }
```

The textbox initializes with `'Dessert'` and immediately applies the filter.

## Programmatic search and reading data

```tsx
const gridRef = useRef<GridRef>(null);

// Apply
gridRef.current?.search('VINET');

// Clear
gridRef.current?.search('');

// Read all currently visible data (current page)
gridRef.current?.getData();

// Read all records across pages (only after search/filter is applied)
gridRef.current?.getData(false);
```

`getData(true)` (default) returns the current page slice. `getData(false)` returns the full filtered dataset across pages.

## `onSearch` event

`SearchEvent`: `{ value: string }` — the active search string.

```tsx
const handleSearch = useCallback((args: SearchEvent) => {
  setIsSearching(!!args.value);
  lastSearchRef.current = args.value;
}, []);
```

## Common patterns

| Need | Configuration |
|---|---|
| Searchable grid with built-in toolbar | `searchSettings={{ enabled: true }}` + `toolbar={['Search']}` |
| Search only across visible fields | `searchSettings={{ fields: ['Title', 'Author'] }}` |
| Wildcard search | `searchSettings={{ operator: 'wildcard' }}` |
| Accent-insensitive | `searchSettings={{ ignoreAccent: true }}` |
| Search-as-you-type debounced | external hook + `gridRef.current?.search(value)` (`debounce` from `@syncfusion/react-base`) |
| Read all result data across pages | `gridRef.current?.getData(false)` |

## Constraints & integration

- Custom-search debounce should commit to the grid via `search()` (not `filterByColumn`) — they live in different stores.
- Server-side search via `DataManager`: the adaptor (URL/ODataV4/WebApi) translates search into a backend query automatically.
- Server-side search via `onDataRequest`: read `state.search` and forward to the backend.
- Search fields participate in AccessEvents (recorded in `onSearch`).
- After search/filter changes, `getData(false)` reflects the new dataset; `getData()` returns only the current page.

## Cross-references

- Toolbar integration (and other toolbar items): see `references/interactivity.md`.
- Custom-API search endpoint wiring: see `references/data-binding.md`.