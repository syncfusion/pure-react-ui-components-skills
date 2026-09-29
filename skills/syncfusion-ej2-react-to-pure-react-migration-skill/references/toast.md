# Toast Migration

This section explains how to migrate the `Toast` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<Toast>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `cssClass` | `className` | Standard rename. |
| `showProgressBar` | `progressBar` | Renamed. |
| `timeOut` (used with `show()` method) | `timeOut` (declarative) | Pure React reads `timeOut` directly without imperative `show()` for static config. |

`content`, `position`, `buttons`, `extendedTimeout`, `newestOnTop` carry
over.

## Methods

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `show()` | `show()` on the typed ref | Same signature. |
| `hide()` | `hide()` on the typed ref | Same signature. |
| `destroy()` | `useEffect` cleanup |

## Events

| EJ2 React | Pure React |
| --- | --- |
| `click` | `onClick` |
| `close` | `onClose` |
| `open` | `onOpen` |
| `destroyed` | `useEffect` cleanup |