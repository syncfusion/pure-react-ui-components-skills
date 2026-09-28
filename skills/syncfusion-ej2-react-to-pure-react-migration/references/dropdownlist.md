# DropDownList Migration

This section explains how to migrate the `DropDownList` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<DropDownList>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `allowFiltering` | `filterable` | Renamed. |
| `cssClass` | `className` | Standard rename. |
| `enableRtl` | `dir` on `<Provider>` | RTL moves to wrapper. |
| `filterBarPlaceholder` | `filterPlaceholder` | Renamed. |
| `floatLabelType` | `labelMode` | Renamed. |
| `locale` | `locale` on `<Provider>` | Locale moves to wrapper. |
| `popupHeight` / `popupWidth` / `zIndex` | `popupSettings={{ height, width, zIndex }}` | Consolidated. |
| `showClearButton` | `clearButton` | Renamed. |
| `cssClass="e-small"` | `size={Size.Small}` | Enum. |
| `cssClass="e-large"` | `size={Size.Large}` | Enum. |
| `cssClass="e-flat"` / `"e-outline"` | `variant={Variant.Standard}` / `Variant.Outlined` | Enum. |

`dataSource`, `allowObjectBinding`, `fields`, `headerTemplate`,
`footerTemplate`, `itemTemplate`, `groupTemplate`, `noRecordsTemplate`,
`valueTemplate`, `ignoreAccent`, `ignoreCase`, `placeholder`, `query`,
`readOnly`, `sortOrder`, `value`, `debounceDelay`, `filterType` carry
over (use `SortOrder` enum).

## Methods

| EJ2 React | Pure React |
| --- | --- |
| `destroy()` | `useEffect` cleanup |

## Events

| EJ2 React | Pure React |
| --- | --- |
| `close` | `onClose` |
| `open` | `onOpen` |
| `select` | `onSelect` |