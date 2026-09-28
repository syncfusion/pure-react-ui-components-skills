# Autocomplete Migration

This section explains how to migrate the `Autocomplete` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<Autocomplete>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `cssClass` | `className` | Standard rename. |
| `allowCustom` | `customValue` | Renamed. |
| `highlight` | `autoHighlight` | Renamed. |
| `showClearButton` | `clearButton` | Renamed. |
| `suggestionCount` | `maxSuggestions` | Renamed. |
| `floatLabelType` | `labelMode` | Renamed. |
| `enableRtl` | `dir` on `<Provider>` | RTL moves to wrapper. |
| `locale` | `locale` on `<Provider>` | Locale moves to wrapper. |
| `readonly` | `readOnly` | Standard React prop. |
| `enabled` | `disabled` (inverted) | Standard rename. |
| `width` | `style` | Use `style={{ width }}`. |
| `headerTemplate`, `footerTemplate`, `itemTemplate`, `noRecordsTemplate`, `valueTemplate`, `groupTemplate` | same names | Accept React nodes / render functions. |

Other props (`autofill`, `dataSource`, `fields`, `minLength`, `placeholder`,
`filterType`, `ignoreCase`, `ignoreAccent`, `allowObjectBinding`, `value`,
`debounceDelay`, `sortOrder`, `required`) carry over with the same names
(use the `SortOrder` enum for `sortOrder`).

## Events

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `change` | `onChange` | Fires on value change. |
| `select` | `onChange` | Fires when an item is selected. |
| `customValueSpecifier` | `onCustomValueSelect` | Custom value committed; value param is the raw value. |
| `filtering` | `onFilter` | During typing-based filtering. |
| `open` | `onOpen` | Popup open. |
| `close` | `onClose` | Popup close. |
| `focus` | `onFocus` | Focus event. |
| `blur` | `onBlur` | Blur event. |