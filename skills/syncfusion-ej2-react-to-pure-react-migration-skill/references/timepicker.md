# TimePicker Migration

This section explains how to migrate the `TimePicker` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<TimePicker>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `allowEdit` | `editable` | Renamed. |
| `cssClass` | `className` | Standard rename. |
| `enableRtl` | `dir` on `<Provider>` | RTL moves to wrapper. |
| `enabled` | `disabled` (inverted) | Standard rename. |
| `floatLabelType` | `labelMode` | Renamed. |
| `locale` | `locale` on `<Provider>` | Locale moves to wrapper. |
| `max` | `maxTime` | Renamed. |
| `min` | `minTime` | Renamed. |
| `showClearButton` | `clearButton` | Renamed. |
| `enableMask` | `inputMask` | Renamed. |

`format`, `openOnFocus`, `placeholder`, `readOnly`, `step`, `strictMode`,
`value`, `zIndex`, `fullScreenMode`, `maskPlaceholder` carry over.

## Methods

| EJ2 React | Pure React |
| --- | --- |
| `destroy()` | `useEffect` cleanup |

## Events

| EJ2 React | Pure React |
| --- | --- |
| `change` | `onChange` |
| `open` | `onOpen` |
| `close` | `onClose` |