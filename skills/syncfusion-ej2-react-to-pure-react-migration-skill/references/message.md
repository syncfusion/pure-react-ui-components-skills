# Message Migration

This section explains how to migrate the `Message` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<Message>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `content` | `children` | Renamed — pass content as children. |
| `cssClass` | `className` | Standard rename. |
| `severity="Success"` | `severity={Severity.Success}` | Enum-only in Pure React. |
| `variant="text"` | `variant={Variant.Text}` | Enum-only in Pure React. |
| `showIcon` | `icon` | Supports custom SVG icons. |
| `showCloseIcon` | `closeIcon` | Supports custom SVG icons. |
| `visible` | `visible` | Same name. Or use conditional rendering. |
| `enableRtl` | `dir` on `<Provider>` | RTL moves to wrapper. |
| `locale` | `locale` on `<Provider>` | Locale moves to wrapper. |

## Methods

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `destroy()` | Not applicable | Conditional render unmounts; React handles cleanup. |

## Events

| EJ2 React | Pure React |
| --- | --- |
| `closed` | `onClose` |
| `created` | `useEffect` with no deps |
| `destroyed` | `useEffect` cleanup |