# Calendar Migration

This section explains how to migrate the `Calendar` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<Calendar>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `cssClass` | `className` | Standard rename. |
| `calendarMode` | `calendarType` | Renamed. Accepts `'gregorian'` / `'islamic'`. |
| `dayHeaderFormat` | `weekDaysFormat` | Renamed. |
| `depth` | `depth` (use `CalendarView` enum) | Same name; take enum values. |
| `enableRtl` | `dir` on `<Provider>` | RTL moves to wrapper. |
| `enabled` | `disabled` (inverted) | Standard rename. |
| `isMultiSelection` | `multiSelect` | Renamed. |
| `locale` | `locale` on `<Provider>` | Locale moves to wrapper. |
| `max` | `maxDate` | Renamed. |
| `min` | `minDate` | Renamed. |
| `fullScreenMode` | `pickerVariant` | `PickerVariant.Auto`. |

Other props (`firstDayOfWeek`, `showTodayButton`, `start`, `value`,
`weekNumber`, `weekRule`) carry over with the same names. Use the `WeekRule`
enum for `weekRule`.

## Events

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `change` | `onChange` | Standard rename. |
| `navigated` | `onViewChange` | Renamed. |
| `renderDayCell` | `cellTemplate` (prop) | Switched from event handler to a template prop. |
| `destroy` | `useEffect` cleanup | Imperative method replaced with hook cleanup. |