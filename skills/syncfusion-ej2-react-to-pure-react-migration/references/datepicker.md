# DatePicker Migration

This section explains how to migrate the `DatePicker` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<DatePicker>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `allowEdit` | `editable` | Renamed. |
| `cssClass` | `className` | Standard rename. |
| `calendarMode` | `calendarType` | Accepts `'gregorian'` / `'islamic'`. |
| `dayHeaderFormat` | `weekDaysFormat` | Renamed. |
| `depth` | `depth` (`CalendarView` enum) | Enum for month/year. |
| `enableRtl` | `dir` on `<Provider>` | RTL moves to wrapper. |
| `enabled` | `disabled` (inverted) | Standard rename. |
| `floatLabelType` | `labelMode` | Renamed. |
| `locale` | `locale` on `<Provider>` | Locale moves to wrapper. |
| `max` | `maxDate` | Renamed. |
| `min` | `minDate` | Renamed. |
| `readonly` | `readOnly` | Standard React prop. |
| `showClearButton` | `clearButton` | Renamed. |
| `fullScreenMode` | `pickerVariant` | `PickerVariant.Auto`. |
| `enableMask` | `inputMask` | Renamed. Drives masked input via `format`. |

`firstDayOfWeek`, `openOnFocus`, `placeholder`, `showTodayButton`,
`strictMode`, `start`, `value`, `weekNumber`, `weekRule`, `zIndex`,
`maskPlaceholder`, `inputFormats` carry over (use `WeekRule` enum for
`weekRule`).

## Methods

| EJ2 React | Pure React |
| --- | --- |
| `destroy()` | `useEffect` cleanup |

## Events

| EJ2 React | Pure React |
| --- | --- |
| `change` | `onChange` |
| `close` | `onClose` |
| `navigated` | `onViewChange` |
| `open` | `onOpen` |
| `renderDayCell` | `cellTemplate` (prop) |