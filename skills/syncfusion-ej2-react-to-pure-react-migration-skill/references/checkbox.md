# Checkbox Migration

This section explains how to migrate the `CheckBox` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<CheckBox>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `checked` | `defaultChecked` | Initial-state switch to React idiom. Combine with `onChange` for controlled use. |
| `cssClass` | `className` | Standard rename. |
| `labelPosition` | `labelPlacement` | Renamed. Use `Position.Left` / `Position.Right` enum. |
| `enabled` (imperative toggle) | `disabled` (inverted) | Imperative toggle not available — manage state with `checked` + `onChange`. |
| `cssClass="e-small"` | `size={Size.Small}` | Enum replaces CSS string. |
| `cssClass="e-large"` | `size={Size.Large}` | Enum replaces CSS string. |

`disabled`, `indeterminate`, `label` carry over with the same names.

## Methods

| EJ2 React | Pure React |
| --- | --- |
| `destroy()` | `useEffect` cleanup |
| `toggle()` | Not available — manage state via `checked` + `onChange` |

## Events

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `onCheckedChange` | `onChange` | Renamed. |
| `focusIn` | `onFocus` | Renamed. |
| `focusOut` | `onBlur` | Renamed. |