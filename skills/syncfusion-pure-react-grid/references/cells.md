---
name: cells
description: Cell-level rendering, behavior, and styling for the Syncfusion React Data Grid — clip mode, text wrap, styling, grid lines, value accessor for display-only expressions, and HTML rendering. Load when the user asks about cell overflow, ellipsis/tooltip, wrapping, conditional CSS classes, computed display values, or rendering HTML inside cells.
---

# Cells

Cell-level rendering and styling for `<Column>`. All cell-level features pivot on these props and the grid-level `gridLines`/`clipMode`/`textWrapSettings`.

## Clip mode

Controls overflow when cell content is wider than the cell:

| Value | Behavior |
|---|---|
| `ClipMode.Clip` | Hard-cut overflow (no indicator) |
| `ClipMode.Ellipsis` | Append `...` to indicate truncation |
| `ClipMode.EllipsisWithTooltip` | Append `...` and show full text on hover |

Apply at grid, column, or both. Column-level wins.

```tsx
<Grid clipMode={ClipMode.EllipsisWithTooltip}>
  <Columns>
    <Column field="Description" clipMode={ClipMode.EllipsisWithTooltip} />
    <Column field="Amount" textAlign={TextAlign.Right} />
  </Columns>
</Grid>
```

Pair with `Column.textWrapSettings` to switch strategy.

## Text wrap

```tsx
const [wrap] = useState<TextWrapSettings>({ enabled: true, wrapMode: WrapMode.Content });

<Grid textWrapSettings={wrap} />
<p>Wraps content cells into multiple lines.</p>
```

`WrapMode` enum:

| Value | Headers | Content |
|---|---|---|
| `WrapMode.Both` | ✓ | ✓ |
| `WrapMode.Header` | ✓ | ✗ |
| `WrapMode.Content` | ✗ | ✓ |

Headers require `width` defined — `width='auto'` rows can't wrap predictably. Wrap interacts with virtualization: row-height changes (due to wrap) affect scroll math. With `textWrapSettings.enabled`, set a fixed `rowHeight` for the best scroll stability.

## Cell class

Two ways to style cells:

### Global CSS

```css
.sf-grid .sf-grid-content-row .sf-cell { font-weight: 500; font-size: 14px; }
```

### Per-cell via Column

`Column.cellClass` accepts a static string or a callback:

```tsx
const getStatusClass = (props?: CellClassProps): string => {
  if (props?.cellType !== CellType.Content) return '';          // skip headers
  const status = (props?.data as Record)?.status;
  switch (status) {
    case 'Active':   return 'badge-green';
    case 'Pending':  return 'badge-yellow';
    case 'Blocked':  return 'badge-red';
    default:         return '';
  }
};

<Column field="status" headerText="Status" cellClass={getStatusClass} />
```

`CellClassProps`:

```ts
{ data: object; column: { field?: string }; rowIndex: number; cellType: CellType }
```

`CellType` values used here: `Header`, `Content`, `Summary`, `GroupCaption`, `Detail`. One callback can branch across cell types.

**Performance:** function-based `cellClass` runs on every render. For high-frequency grids, prefer `template` or sticky CSS via `headerTemplate`.

## Grid lines

`gridLines` (grid-level):

| `GridLine` value | Horizontal | Vertical |
|---|---|---|
| `GridLine.Default` | ✓ | ✗ |
| `GridLine.None` | ✗ | ✗ |
| `GridLine.Both` | ✓ | ✓ |
| `GridLine.Horizontal` | ✓ | ✗ |
| `GridLine.Vertical` | ✗ | ✓ |

```tsx
const [lines] = useState<GridLine | string>(GridLine.Default);
<Grid gridLines={lines} />
```

String form also accepted.

## Value accessor (`valueAccessor`)

Computes the displayed cell value from row data — display-only. Sort/filter/edit operate on the raw value, so choose this when you want formatting that should not affect query logic.

```tsx
const percent = useCallback((props?: ValueAccessorProps): string => {
  const score = (props?.data as Grade)?.score ?? 0;
  return `${(score * 100).toFixed(2)}%`;
}, []);

<Column field="score" headerText="Score" valueAccessor={percent} />
<Column field="hireDate"
        valueAccessor={(props?: ValueAccessorProps) => {
          const days = (new Date().getTime() - new Date((props?.data as Emp)?.hireDate).getTime()) / 86400000;
          return days < 1 ? 'Today' : days < 2 ? 'Yesterday' : `${Math.floor(days)} days ago`;
        }} />
```

Common uses:
- Formatted currency/percent strings.
- Letter grades computed from raw scores.
- Concatenations (`"John (US)"`).
- Date-relative terms (`Today`, `Tomorrow`, `5 days ago`).
- Weighted aggregates (midterm*0.25 + project*0.30 + …).

Always memoize with `useCallback` and reference `data` from `ValueAccessorProps.data`. Wrap the `<Grid>` JSX in `useMemo` to keep the column render stable.

## Disable HTML encoding

The grid HTML-encodes cells/header strings for XSS safety by default. To render raw HTML markup:

```tsx
<Column field="feedback" headerText="<strong>Feedback</strong>" disableHtmlEncode />
```

Recommended pattern: pass `disableHtmlEncode={true}` only when you control the input. For dynamic graphical content, prefer a `template` returning `React.ReactElement` — templates bypass encoding entirely.

## Cell selection styling

`.sf-cell.sf-cell-selected` styles background/foreground. Define your own theme overrides through your CSS-variable pack, or add a CSS rule:

```css
.sf-grid .sf-cell.sf-cell-selected { background: var(--sf-color-primary-light); color: var(--sf-color-primary-text-color); }
```

## Cell-level hooks summary

| Hook | API surface |
|---|---|
| Overflow | `clipMode` (grid + column) |
| Wrap | `textWrapSettings` (`WrapMode` enum) |
| Styling | `cellClass` (column) + CSS |
| Lines | `gridLines` (grid) — see `cells.md` section "Grid lines" |
| Computed display | `valueAccessor` (column) |
| Raw HTML | `disableHtmlEncode` (column) + `template` (preferred for dynamic content) |

## Constraints

- Function-based callbacks on `cellClass`/`valueAccessor` execute on every render. Memoize, keep them pure, return strings (avoid `React.ReactElement`s when sorting needs to be preserved — display-only is fine but increases render time).
- `cellClass` should branch on `cellType` to skip applying content styles to headers.
- `valueAccessor` is purely display logic — don't use it for filtering/validation logic that should affect data.
- `disableHtmlEncode` should only be set `true` when input is fully trusted (XSS risk otherwise).