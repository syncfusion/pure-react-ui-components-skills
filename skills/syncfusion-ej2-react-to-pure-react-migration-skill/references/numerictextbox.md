# NumericTextBox Migration

This section explains how to migrate the `NumericTextBox` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<NumericTextBox>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `showClearButton` | `clearButton` | Renamed. |
| `cssClass` | `className` | Standard rename. |
| `maxValue` | `max` | Renamed. |
| `minValue` | `min` | Renamed. |
| `allowMouseWheel` | `mouseWheel` | Renamed. |
| `validateDecimalOnType` | `validateOnType` | Renamed. |
| `labelMode` | `floatLabelType` | Renamed (note: opposite direction from EJ2's `labelMode`). |
| `prependTemplate` | `prefix` | Renamed. |
| `appendTemplate` | `suffix` | Renamed. |

`currency`, `decimals`, `format`, `strictMode`, `placeholder`, `step`
carry over.

## Events

| EJ2 React | Pure React |
| --- | --- |
| `change` | `onChange` |