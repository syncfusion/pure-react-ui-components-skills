# TextBox Migration

This section explains how to migrate the `TextBox` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<TextBox>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `showClearButton` | `clearButton` | Renamed. |
| `cssClass` | `className` | Standard rename. |
| `cssClass="e-small"` | `size={Size.Small}` | Enum. |
| `cssClass="e-bigger"` | `size={Size.Large}` | Enum. |
| `cssClass="e-success"` / `"e-warning"` / `"e-danger"` | `color={Color.Success}` / `Warning` / `Danger` | Enum (note `Danger`, not `Error`). |
| `labelMode` | `floatLabelType` | Renamed (note: opposite direction from EJ2). |
| `prependTemplate` | `prefix` | Renamed. |
| `appendTemplate` | `suffix` | Renamed. |

`placeholder` carries over.

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