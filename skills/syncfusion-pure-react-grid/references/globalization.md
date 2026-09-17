---
name: globalization
description: Localization (L10n), culture-sensitive data formatting, CLDR data loading, RTL/Provider configuration, and per-grid locale loadouts for the Syncfusion React Data Grid. Load when targeting a non-en-US app, supporting RTL languages, customizing labels (button text, confirm dialog, pager), or wiring date/number formats per-culture.
---

# Globalization

The grid adopts the Syncfusion base globalization stack: `L10n.load`, `loadCldr` from `@syncfusion/react-base`, plus the `<Provider locale={...}>` wrapper. The grid's own `enableRtl` and `locale` props coordinate with these.

## Package imports

```tsx
import { L10n, loadCldr, Provider } from '@syncfusion/react-base';

// CLDR data — per-culture
import * as enAllData from '@syncfusion/react-cldr-data/main/en/all.json';
import * as deAllData from '@syncfusion/react-cldr-data/main/de/all.json';
import * as frAllData from '@syncfusion/react-cldr-data/main/fr/all.json';
import * as arAllData from '@syncfusion/react-cldr-data/main/ar/all.json';
import * as zhAllData from '@syncfusion/react-cldr-data/main/zh/all.json';

// Supplemental — once globally
import * as likelySubtags from '@syncfusion/react-cldr-data/supplemental/likelySubtags.json';
import * as numberingSystemData from '@syncfusion/react-cldr-data/supplemental/numberingSystems.json';
import * as currencyData from '@syncfusion/react-cldr-data/supplemental/currencyData.json';

// Locale text strings — per-culture
import * as en from '@syncfusion/react-locale/src/en-US.json';
import * as de from '@syncfusion/react-locale/src/de.json';
import * as fr from '@syncfusion/react-locale/src/fr.json';
```

`npm install @syncfusion/react-cldr-data @syncfusion/react-locale --save`.

## Loading pattern

```tsx
import { useEffect } from 'react';
import { L10n, loadCldr } from '@syncfusion/react-base';

useEffect(() => {
  L10n.load({ ...en, ...de, ...fr });
  loadCldr(
    enAllData, deAllData, frAllData,
    numberingSystemData,
    currencyData,
    likelySubtags
  );
}, []);
```

Load once at app startup, in the entry points (e.g., `src/App.tsx`, `src/main.tsx`).

## Per-grid locale

```tsx
<Provider locale="de">
  <Grid enableRtl={false} dataSource={data} ... />
</Provider>
```

`enableRtl` is set per grid (not per Provider). When the locale is `ar` (or any RTL language), set `enableRtl={true}`.

```tsx
<Provider locale="ar">
  <Grid enableRtl dataSource={data} ... />
</Provider>
```

Multi-grid apps: wrap each grid in its own `<Provider>` for independent locales — they don't share state.

## Custom labels

```ts
L10n.load({
  'en-custom': {
    grid: {
      emptyRecord: 'Nothing here yet!',
      confirmDeleteMessage: 'Are you sure you want to delete this record?',
      okButtonLabel: 'YES',
      cancelButtonLabel: 'Discard',
      addButtonLabel: 'New',
      editButtonLabel: 'Modify',
      updateButtonLabel: 'Save',
      deleteButtonLabel: 'Remove',
      currentPageLabel: '{0} of {1}',
    },
  },
});

<Provider locale="en-custom">
  <Grid ... />
</Provider>
```

## Localization keyword catalog (grid properties to localize)

### Data rendering

| Key | Default | Notes |
|---|---|---|
| `emptyRecord` / `noRecordsMessage` | "No records to display" | Empty-record template |

### Columns

| Key | Default |
|---|---|
| `booleanTrueLabel` | "true" |
| `booleanFalseLabel` | "false" |

### Editing / toolbar

| Key | Default |
|---|---|
| `addButtonLabel` / `add` | "Add" |
| `editButtonLabel` / `edit` | "Edit" |
| `cancelButtonLabel` / `cancel` | "Cancel" |
| `updateButtonLabel` / `update` | "Update" |
| `deleteButtonLabel` / `delete` | "Delete" |
| `confirmDeleteMessage` | "Are you sure you want to delete the record?" |

### Delete-via-checkbox dialog

| Key | Notes |
|---|---|
| `chooseRecordsToDelete` | Trigger label |
| `deleteSelectedRecordsOnPage` | `{0}` placeholder = count, `{1}` = pluralization |
| `deleteSelectedRecordsOnPageDescription` | |
| `deleteAllSelectedRecordsAcrossPages` | |
| `deleteAllSelectedRecordsAcrossPagesDescription` | |
| `deleteAllSelectedRecordsAcrossPagesDescriptionNoCurrentPage` | |
| `deleteAllSelectedRecordsFromLoadedPages` | |
| `deleteAllSelectedRecordsFromLoadedPagesDescription` | |
| `deleteAllSelectedRecordsFromLoadedPagesDescriptionNoCurrentPage` | |
| `okButtonLabel` | |
| `cancelButtonLabel` | |

### Pager

| Key | Default template / string |
|---|---|
| `currentPageLabel` | `"{0} of {1} pages"` |
| `totalItemsLabel` | `"({0} items)"` |
| `firstPageTooltip` | "Go to first page" |
| `lastPageTooltip` | "Go to last page" |
| `nextPageTooltip` | "Go to next page" |
| `previousPageTooltip` | "Go to previous page" |
| `nextPageGroupTooltip` | "Go to next page group" |
| `previousPageGroupTooltip` | "Go to previous page group" |
| `pagerStatusMessage` | "Pager external message" |
| `pagerOfLabel` | " of " |

### Accessibility

| Key | Default |
|---|---|
| `ariaSortAscending`, `ariaSortDescending`, `ariaSortNone` | ARIA announcements for sort direction |
| `ariaSelected`, `ariaBusy`, `columnheader`, `gridCell` | ARIA roles |

## Number, date, currency formats

`Column.format` follows locale-specific format strings: `'yMd'`, `'C2'`, `'N2'`, `'P0'`, `'P'`, `'MM/dd/yyyy HH:mm'`. Skeletons: `'short'`, `'medium'`, `'long'`, `'full'`. Custom number specifiers: `'0000'`, `'####'`, `'$#,##0.00'`. See `references/cells.md` for the full symbol table.

Pair locale + `format` for culture-correct display:

```tsx
<Provider locale="de">
  <Grid dataSource={data}>
    <Columns>
      <Column field="price" format="C2" />     {/* €1.234,57 */}
      <Column field="date" format="dd.MM.yyyy" type={ColumnType.Date} />
    </Columns>
  </Grid>
</Provider>
```

## Culture-aware sorting

`sortComparer` + `<Provider locale={...}>` + `loadCldr(...)`:

```tsx
import { Provider } from '@syncfusion/react-base';

const collator = new Intl.Collator('de');
const compareDe = useCallback(
  (a: string, b: string) => collator.compare(a, b), []);

<Column field="Name" sortComparer={compareDe} />
```

`Intl.Collator` lives in the runtime; the grid does not need its own culture tables for sorting.

## Built-in localizations (`@syncfusion/react-locale`)

Locales ship as JSON files: `en-US.json`, `de.json`, `fr.json`, `ar.json`, `zh.json`, … Load every locale you intend to use, and merge into `L10n.load`.

## Common patterns

| Need | Configuration |
|---|---|
| English (default) | No wrapping needed; grid defaults |
| App-level locale switch | `Provider` at the root, `loadCldr` once |
| RTL | `enableRtl` on `<Grid>` + `Provider` with an RTL locale |
| Custom confirm dialog | `L10n.load({ 'en-custom': { grid: { confirmDeleteMessage } } })` |
| Pluralized delete labels | Use `{0}` / `{1}` placeholders |

## Constraints & guardrails

- **`Provider`** must wrap the grid — otherwise `<Grid>` falls back to its default culture.
- **`enableRtl`** is a `<Grid>` prop (not a `<Provider>` prop). Apply per grid.
- **`loadCldr`** should be called **once** at app startup, not per render — wrap in a module-scope guard or `useEffect(() => { ... }, [])` at the root component.
- **`L10n.load`** is **idempotent per key** — re-loading the same key replaces the bundle. Avoid loading multiple locales with overlapping keys unless you intend the override.
- **`Intl.Collator`** does not require CLDR data — but `<Provider>` does. For sorting + locale only, you can skip CLDR if `<Provider>` isn't wrapping the grid.
- **Locale-aware formatting**: confirm `Column.type` is set explicitly when the first cell could be null/empty.
- **High-risk operation**: switching `Provider`'s `locale` mid-session can re-render sensitive inputs — for a security context, ensure no form data is in-flight before flipping.
- **Integration**: when shipping a multi-locale grid, also update exports (`L10n` keys flow into PDF/Excel/Print headers/footers) and confirm dialog labels.