# MultiSelect Migration

This section explains how to migrate the `MultiSelect` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<MultiSelect>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `allowCustomValue` | `customValue` | Renamed. |
| `cssClass` | `className` | Standard rename. |
| `enabled` | `disabled` (inverted) | Standard rename. |
| `floatLabelType` | `labelMode` | Renamed. |
| `enableRtl` | `dir` on `<Provider>` | RTL moves to wrapper. |
| `htmlAttributes` | `inputProps` | Renamed. |
| `locale` | `locale` on `<Provider>` | Locale moves to wrapper. |
| `readonly` | `readOnly` | Standard React prop. |
| `showClearButton` | `clearButton` | Renamed. |
| `width` | `style={{ width }}` | Use `style`. |

`addTagOnBlur`, `allowObjectBinding`, `dataSource`, `debounceDelay`,
`delimiterChar`, `fields`, `hideSelectedItem`, `ignoreAccent`, `ignoreCase`,
`maximumSelectionLength`, `mode` (`DisplayMode.Box`), `openOnClick`,
`placeholder`, `selectAllText`, `showSelectAll`, `unSelectAllText`,
`value` carry over.

## Events

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `change` | `onChange` | Selected values change. |
| `chipSelection` | `onChipClick` | Fires when a chip is clicked. |
| `removed` | `onChipDelete` | Receives the deleted chip item. |
| `customValueSelection` | `onCustomValueSelect` | Custom value committed via Enter. |
| `selectedAll` | `onSelectAll` | Select-all toggle. |