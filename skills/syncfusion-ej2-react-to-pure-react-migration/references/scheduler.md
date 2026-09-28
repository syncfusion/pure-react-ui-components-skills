# Scheduler Migration

This section explains how to migrate the `Scheduler` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<Scheduler>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `cssClass` | `className` | Standard rename. |
| `allowDragAndDrop` | `eventDrag` | Renamed. |
| `allowResizing` | `eventResize` | Renamed. |
| `cellTemplate` | `cell` | Suffix dropped. |
| `cellHeaderTemplate` | `cellHeader` | Suffix dropped. |
| `dateHeaderTemplate` | `dateHeader` | Suffix dropped. |
| `currentView` | `defaultView` (uncontrolled) / `view` (controlled) | Maps to either uncontrolled default or controlled value with `onViewChange`. |
| `editorTemplate` / `editorHeaderTemplate` / `editorFooterTemplate` | `editor` | Unified to a single prop returning `<SchedulerEditorPopup dialogProps={...} />`. |
| `enableRtl` | `dir` on `<Provider>` | RTL moves to wrapper. |
| `dateRangeTemplate` | `header.dateRangeTemplate` | Nested under `header`. |
| `eventDragArea` | `eventDrag.eventDragArea` | Nested under `eventDrag`. |
| `readonly` | `readOnly` | Standard React prop. |
| `resourceHeaderTemplate` | `resourceHeader` | Suffix dropped. |
| `selectedDate` | `defaultSelectedDate` or `selectedDate` (with `onSelectedDateChange`) | Both uncontrolled and controlled variants supported. |
| `showQuickInfo` | `showQuickInfoPopup` | Renamed. |

In EJ2 resources are configured via `<ResourcesDirective>` / `<ResourceDirective>`.
In Pure React, `resources` is a plain array passed as a prop.

`agendaDaysCount`, `dateFormat`, `enableRecurrenceValidation`, `group`,
`startHour`, `endHour`, `workHours`, `eventSettings`, `firstDayOfWeek`,
`height`, `hideEmptyAgendaDays` / `<AgendaView hideEmptyAgendaDays />`,
`rowAutoHeight`, `showTimeIndicator`, `showWeekend`, `timeFormat`,
`timezone`, `timezoneDataSource`, `weekRule`, `width`, `workDays` carry
over (use `WeekRule` enum for `weekRule`).

## Events

| EJ2 React | Pure React |
| --- | --- |
| `cellClick` | `onCellClick` |
| `cellDoubleClick` | `onCellDoubleClick` |
| `dataBinding` | `onDataRequest` |
| `dragStart` / `drag` / `dragStop` | `onDragStart` / `onDrag` / `onDragStop` |
| `resizeStart` / `resizing` / `resizeStop` | `onResizeStart` / `onResizing` / `onResizeStop` |
| `eventClick` / `eventDoubleClick` | `onEventClick` / `onEventDoubleClick` |
| `actionBegin` | `onDataChangeStart` |
| `actionComplete` | `onDataChangeComplete` |
| `tooltipOpen` | `onTooltipOpen` |
| `moreEventsClick` | `onMoreEventsClick` |
| `error` | `onError` |
| `editorSubmit` | `onEditorSubmit` |