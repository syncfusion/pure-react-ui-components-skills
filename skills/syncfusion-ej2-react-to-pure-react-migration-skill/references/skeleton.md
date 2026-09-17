# Skeleton Migration

This section explains how to migrate the `Skeleton` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<Skeleton>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `shape="Circle"` | `variant={Variants.Circle}` | Renamed; enum replaces string. |
| `cssClass` | `className` | Standard rename. |
| `shimmerEffect="Pulse"` | `animation={AnimationType.Pulse}` | Renamed; enum replaces string. |

## Methods

| EJ2 React | Pure React |
| --- | --- |
| `destroy()` | `useEffect` cleanup |