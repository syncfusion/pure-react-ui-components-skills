# SplitButton Migration

This section explains how to migrate the `SplitButton` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<SplitButton>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `cssClass` | `className` | Standard rename. |
| `iconCss` | `icon` | Accepts string or React node. |
| `cssClass="e-small"` | `size={Size.Small}` | Enum. |
| `cssClass="e-large"` | `size={Size.Large}` | Enum. |
| `cssClass="e-flat"` / `"e-outline"` | `variant={Variant.Standard}` / `Variant.Outlined` | Enum. |

`items`, `iconPosition`, `popupWidth`, `itemTemplate`, `disabled` carry
over.

## Methods

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `toggle()` (method) | Implemented in Pure React only | Use typed ref + `instance.toggle()`. |
| `destroy()` / `destroyed` callback | `useEffect` cleanup | |

## Events

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `close` | `onClose` | |
| `open` | `onOpen` | |
| `select` | `onSelect` | |
| `toggle` (event) | Not applicable | EJ2-only. |