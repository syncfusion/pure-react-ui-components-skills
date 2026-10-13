---
name: formulas
description: Formula cells and the formula engine inside the Syncfusion React Data Grid — `allowFormula`/`formulaSettings`, spreadsheet-style A1/B2/R[1]C[0] notation, cell-range references (A1:B2), built-in functions (SUM, AVERAGE, IF, CONCAT, TODAY, …), custom functions via `customFunctions`, the formula editor that converts expression ↔ A1 notation while editing, and the error codes (`#INVALID_REFERENCE`, `#CIRCREF!`, `#PARSE!`, `#NAME?`). Load when wiring spreadsheet-style computed cells, building a calculation footer, or exposing a formula input editor.
---

# Formulas

The formula engine turns cells into spreadsheet-style computed values. Cells whose column opts in (`allowFormula={true}`) evaluate a stored expression against the data source and update when dependencies change.

`FormulaModule` is required.

## Enabling formulas

```tsx
import { Grid, Columns, Columns, FormulaModule, EditModule, PagerModule } from '@syncfusion/react-grid';

const modules = { EditModule, PagerModule, FormulaModule };

<Grid
  dataSource={dairyProductList}
  height="400px"
  rowNumberSettings={{ enabled: true }}
  selectionSettings={{ type: 'Cell' }}
  formulaSettings={{ enabled: true }}
  editSettings={{ allowEdit: true, mode: 'Cell' }}
  modules={modules}
  enableDevMode={false}
>
  <Columns>
    <Column field="id"       headerText="ID"        isPrimaryKey />
    <Column field="product"  headerText="Product" />
    <Column field="price"    headerText="Price"     edit={{ type: EditType.NumericTextBox }} format='C2' />
    <Column field="quantity" headerText="Quantity"  edit={{ type: EditType.NumericTextBox }} format='N2' />
    <Column field="subtotal" headerText="Subtotal"  allowFormula={true} format='C2' />
    <Column field="tax"      headerText="Tax"       allowFormula={true} format='C2' />
    <Column field="total"    headerText="Total"     allowFormula={true} format='C2' />
  </Columns>
</Grid>
```

The formula expression lives in the data source row itself:

```ts
{
  id: 1,
  price: 3.5,  quantity: 2,
  subtotal: '=REF(COLUMN("price"),ROW(1))*REF(COLUMN("quantity"),ROW(1))',
  tax:      '=REF(COLUMN("subtotal"),ROW(1))*0.07',
  total:    '=REF(COLUMN("subtotal"),ROW(1))+REF(COLUMN("tax"),ROW(1))',
}
```

The engine evaluates these expressions; outputs surface as numeric / string values in the cell. When dependencies change, dependents recompute automatically.

## Cell reference notation

| Form | Example | Meaning |
|---|---|---|
| Single cell | `A1`, `B3`, `AA10` | One cell. |
| Range | `A1:B2`, `AA1:AB20` | Contiguous rectangular range (inclusive endpoints). |

Column labels run A → Z, then AA → AB → AC, etc. Row numbers start at 1.

`ROW(1)` and `COLUMN("price")` are programmatic-witness helpers: they return the current cell's row index / the column index of the named field. `REF(COLUMN("price"), ROW(1))` therefore resolves to the value of the "price" column on the current row.

## Operators

| Operator | Description |
|---|---|
| `+`, `-`, `*`, `/` | Arithmetic (incl. unary minus). `/` by zero raises `#PARSE!`. |
| `=`, `<>` | Equality comparisons. |
| `<`, `>`, `<=`, `>=` | Ordering comparisons. |

Exponentiation and text concat go through `POWER` and `CONCAT` rather than `^` or `&`.

## Built-in functions

### Aggregate & statistical
`SUM`, `AVERAGE`, `COUNT`, `COUNTA`, `COUNTBLANK`, `COUNTIF`, `SUMIF`, `MEDIAN`, `MIN`, `MAX`, `PRODUCT`.

### Math
`ABS`, `ROUND`, `MOD`, `POWER`, `SQRT`, `RAND`/`RANDOM`/`RAN` (all three names refer to the same random-number function).

### Logical
`IF(condition, valueIfTrue, valueIfFalse)`.

### Text
`CONCAT` (alias `CONCATENATE`).

### Date
`TODAY`, `NOW` (alias of each other).

### Custom functions

Pass a `customFunctions` map via `formulaSettings`:

```tsx
type CustomFormulaArgs = { values: unknown[] };

const customFunctions = {
  CUSTOMSUM: ({ values }: CustomFormulaArgs) =>
    values.reduce<number>((sum, v) => sum + Number(v), 0),
};

const [formulaSettings] = useState({
  enabled: true,
  customFunctions,
});

<Grid formulaSettings={formulaSettings} modules={modules} ... />
```

Use `CUSTOMSUM(A1:A3)` inside any formula column.

## Formula editor

When editing is enabled on a formula-enabled column, the default editor shows the **expression in A1 notation** (`=D1*E1`), highlights referenced cells, and updates the reference when the user clicks another cell during edit. The value preview (the result) displays after editing.

To opt out of the editor on a formula column while still evaluating the cell, set `<Column allowFormula={true} allowEdit={false} />`.

```tsx
<Column field="subtotal" headerText="Subtotal" allowFormula={true} allowEdit={false} format='C2' />
```

## Error codes

| Code | Cause |
|---|---|
| `#INVALID_REFERENCE` | Reference points outside the available data, or refers to a column not enabled for formulas. |
| `#CIRCREF!` | Formula refers back to its own cell directly or transitively. |
| `#PARSE!` | Expression cannot be tokenized / parsed, or division by zero. |
| `#NAME?` | Function name is unknown (not built-in and not registered). |

Surface these in templates to drive red-cell / tooltip UX:

```ts
const CellWithError = useCallback((props?: ColumnTemplateProps<T>): React.ReactElement => {
  const value = props?.column ? (props.data as T)[props.column.field as keyof T] : undefined;
  return typeof value === 'string' && value.startsWith('#') ? (
    <span className="sf-formula-error">{String(value)}</span>
  ) : (
    <span>{Number(value).toLocaleString()}</span>
  );
}, []);
```

## Best practices & guardrails

- **Store expressions in the data source**, not in column definitions. The formula module reads expressions from each row's payload — this is what lets server-side data flow in formula values without re-parsing templates.
- **Mark exactly the columns that hold expressions** with `allowFormula={true}`. Non-formula columns can still be referenced from formulas.
- **`isPrimaryKey`** is required because formulas can join across rows by computed ranges; without a key, edits may mis-target.
- **Performance — large datasets**: formulas evaluate on every cell whose `allowFormula` is true and on every dependent. For thousands of formula rows, batch the source mutation (so dependent cells only recompute once at end of batch) — or use `setCellValueAsync` to stagger dependent updates server-side.
- **Custom functions receive resolved values**, including range-expanded arrays. The signature is `({ values, args, row, col }) => unknown`.
- **Templates + formulas**: `template` runs over the *resolved* value (numeric, not the raw expression), so DOM renderers see the result. Use `args.data[field]` only when you need the original expression.

## Cross-references

- For cross-row / cross-column programmatic updates, see `references/editing-modes.md` (`setCellValue`, `setCellValueAsync`) and `references/data-binding.md` (`onDataRequest` for server-driven expressions).
- For aggregate footers vs. formula columns: aggregates compute unweighted summaries; formula columns recompute per dependency. Choose aggregates for stable column summaries (`footerTemplate`), formulas for cascading computation.
