# TextArea Migration

This section explains how to migrate the `TextArea` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<TextArea>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `showClearButton` | `clearButton` | Renamed. |
| `cssClass` | `className` | Standard rename. |
| `cssClass="e-small"` | `size={Size.Small}` | Enum. |
| `cssClass="e-bigger"` | `size={Size.Large}` | Enum. |
| `resizeMode="Both"` | `resizeMode={ResizeMode.Both}` | Enum replaces string. |
| `cssClass="e-filled"` | `variant={Variant.Filled}` | Enum. |
| `cssClass="e-outline"` | `variant={Variant.Outlined}` | Enum. |
| `labelMode` | `floatLabelType` | Renamed (note: opposite direction from EJ2). |
| `prependTemplate` | `prefix` | Renamed. |
| `appendTemplate` | `suffix` | Renamed. |

`cols`, `rows`, `maxLength`, `placeholder` carry over.

## Methods

| EJ2 React | Pure React |
| --- | --- |
| `destroy()` | `useEffect` cleanup |

## Events

| EJ2 React | Pure React |
| --- | --- |
| `change` | `onChange` |
| `focusIn` | `onFocus` |
| `focusOut` | `onBlur` |