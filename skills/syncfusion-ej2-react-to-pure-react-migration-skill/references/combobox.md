# ComboBox Migration

This section explains how to migrate the `ComboBox` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<ComboBox>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `allowCustom` | `customValue` | Renamed. |
| `allowFiltering` | `filterable` | Renamed. |
| `cssClass` | `className` | Standard rename. |
| `enabled` | `disabled` (inverted) | Standard rename. |
| `enableRtl` | `dir` on `<Provider>` | RTL moves to wrapper. |
| `enableVirtualization` | `virtualization` (object: `{ itemHeight }`) | Renamed and retyped. |
| `floatLabelType` | `labelMode` | Renamed. |
| `htmlAttributes` | `inputProps` | Renamed. |
| `locale` | `locale` on `<Provider>` | Locale moves to wrapper. |
| `popupHeight` / `popupWidth` | `popupSettings={{ height, width }}` | Consolidated. |
| `readonly` | `readOnly` | Standard React prop. |
| `showClearButton` | `clearButton` | Renamed. |

`autofill`, `allowObjectBinding`, `dataSource`, `debounceDelay`, `fields`,
`filterType`, `footerTemplate`, `groupTemplate`, `headerTemplate`,
`ignoreAccent`, `ignoreCase`, `itemTemplate`, `noRecordsTemplate`,
`placeholder`, `required`, `sortOrder`, `value` carry over with the same
names (`sortOrder` uses `SortOrder` enum).

## Events

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `actionBegin` | `onDataRequest` | Fire on data request. |
| `actionComplete` | `onDataLoad` | Fire on data load. |
| `actionFailure` | `onError` | Fire on data load error. |
| `change` | `onChange` | Value change. |
| `select` | `onChange` | Item chosen. |
| `open` | `onOpen` | Popup open. |
| `close` | `onClose` | Popup close. |
| `customValueSpecifier` | `onCustomValueSelect` | Custom value committed (passes raw value). |
| `filtering` | `onFilter` | During typing-based filtering. |