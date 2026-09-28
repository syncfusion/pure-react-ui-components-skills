# DropDownButton Migration

This section explains how to migrate the `DropDownButton` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<DropDownButton>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `cssClass` | `className` | Standard rename. |
| `iconCss` | `icon` | Accepts string or React node. |
| `cssClass="e-primary"` / `"e-success"` / `"e-info"` / `"e-warning"` / `"e-danger"` | `color={Color.Primary}` / `Success` / `Info` / `Warning` / `Error` | Enum replaces CSS string. |
| `cssClass="e-small"` | `size={Size.Small}` | Enum. |
| `cssClass="e-large"` | `size={Size.Large}` | Enum. |
| `cssClass="e-flat"` / `"e-outline"` | `variant={Variant.Standard}` / `Variant.Outlined` | Enum. |

`items`, `iconPosition`, `itemTemplate`, `popupWidth`, `disabled` carry over.

## Methods

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `toggle()` (event) | Not applicable | The `toggle` *event* is EJ2-only. |
| `destroy()` | `useEffect` cleanup |
| `toggle()` (method) | Implemented in Pure React only | Use `ref` + `instance.toggle()`. |

## Events

| EJ2 React | Pure React |
| --- | --- |
| `close` | `onClose` |
| `open` | `onOpen` |
| `select` | `onSelect` |
| `toggle` (event) | Not applicable | EJ2-only. |
| `destroyed` | `useEffect` cleanup |