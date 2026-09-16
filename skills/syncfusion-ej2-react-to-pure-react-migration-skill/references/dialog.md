# Dialog Migration

This section explains how to migrate the `Dialog` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<Dialog>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `animationSettings` | `animation` | Renamed; same shape. |
| `showCloseIcon` | `closeIcon` | Renamed. Also accepts `ReactNode`. |
| `cssClass` | `className` | Standard rename. |
| `allowDragging` | `draggable` | Renamed. |
| `isModal` | `modal` | Renamed. |
| `enableResize` | `resizable` | Renamed. |
| `footerTemplate` | `footer` | Renamed. |

`position`, `resizeHandles`, `header`, `target` carry over.

## Methods

| EJ2 React | Pure React |
| --- | --- |
| `destroy()` | `useEffect` cleanup |

## Events

| EJ2 React | Pure React |
| --- | --- |
| `drag` | `onDrag` |
| `dragStart` | `onDragStart` |
| `dragStop` | `onDragStop` |
| `resizing` | `onResize` |
| `resizeStart` | `onResizeStart` |
| `dragStop` (resize end) | `onResizeStop` |
| `close` | `onClose` |