# Tooltip Migration

This section explains how to migrate the `Tooltip` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<Tooltip>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `cssClass` | `className` | Standard rename. |
| `showTipPointer` (boolean) | `arrow` | Renamed. |
| `showTipPointer="start"` (position string) | `arrowPosition="start"` | Renamed with new key. |
| `isSticky` | `sticky` | Renamed. |
| `mouseTrail` | `followCursor` | Renamed. |

`animation`, `closeDelay`, `openDelay`, `open`, `opensOn`,
`windowCollision`, `content`, `offsetX`, `offsetY` carry over.

## Methods

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `destroy()` | `useEffect` cleanup |
| `close()` | `closeTooltip()` | Renamed. |
| `open()` | `openTooltip()` | Renamed. |
| `refresh()` | `refresh()` | Same. |

## Events

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `beforeOpen` | `onOpen` | Pre-open logic moves here. |
| `beforeClose` | `onClose` | Pre-close logic moves here. |