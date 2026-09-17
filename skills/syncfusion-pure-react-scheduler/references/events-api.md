# Scheduler Events API

The scheduler offers event hooks for initialization, rendering, user interactions, data operations, and errors, letting you track the component's lifecycle in real time. Use them to validate input, show UI feedback, and persist changes as users click, drag/resize, or navigate between views.

## Event hooks overview

| Event Name | Description |
|--------------------|-----------------------------------------------|
| `onCellClick` | Fired when a scheduler cell is clicked (or tapped on mobile devices). Useful for custom cell selection logic or triggering event creation dialogs. |
| `onCellDoubleClick` | Fired when a cell is double-clicked (or double-tapped on mobile devices). Commonly used to open the editor for creation. |
| `onDataChangeComplete` | Fired after a scheduler data modification successfully completes. Useful for refreshing dependent components or displaying success confirmations. |
| `onDataChangeStart` | Fired at the start of any scheduler data modification (add, edit, or delete). Can validate changes or display loading indicators. |
| `onDataRequest` | Fired before event data is loaded from the data source or remote server. Useful for preprocessing or validating data before it renders (also the basis of load-on-demand). |
| `onDrag` | Fired continuously while an event is being dragged across time slots. Useful for real-time validation or updating dependent UI elements. |
| `onDragStart` | Fired when the user begins dragging an event. Can cancel the drag, apply restrictions, or trigger visual feedback. |
| `onDragStop` | Fired when the user releases an event after dragging. Can apply final validation or persist changes to the data source. |
| `onError` | Fired when an error occurs during a scheduler operation (e.g. data loading, event validation). Useful for logging errors or displaying error messages. |
| `onEventClick` | Fired when an event is clicked (or tapped on mobile devices). Use this to display event details or trigger custom event workflows. |
| `onEventDoubleClick` | Fired when an event is double-clicked (or double-tapped on mobile devices). Typically opens the event editor for viewing or modifying event details. |
| `onMoreEventsClick` | Fired when the user clicks the "+n more events" indicator in Month view. Useful for opening the agenda view to show all events for a specific date. |
| `onResizeStart` | Fired when the user grabs an event resize handle to begin resizing. Can validate or cancel the resize operation. |
| `onResizeStop` | Fired when the user releases the resize handle after adjusting an event's duration. Can apply final validation or persist the new duration. |
| `onResizing` | Fired continuously while the user is dragging an event resize handle. Useful for real-time duration validation or preview updates. |
| `onSelectedDateChange` | Fired when the user changes the active date. Use this to synchronize the scheduler with other components. |
| `onViewChange` | Fired when the user switches between view types (Day, Week, Month, etc.). Useful for updating UI state or loading view-specific data. |

## Wiring up the hooks

The full callback set is attached directly on `<Scheduler>`. A typical pattern: log every interaction, and keep `view`/`selectedDate` state in sync for controlled mode:

```tsx
import { useCallback, useMemo, useState } from 'react';
import { DayView, WeekView, WorkWeekView, MonthView, Scheduler, SchedulerDateChangeEvent, SchedulerViewChangeEvent, SchedulerDataChangeEvent, SchedulerEventClickEvent, SchedulerCellClickEvent } from '@syncfusion/react-scheduler';

export default function App() {
    const [selectedDate, setSelectedDate] = useState(new Date('2025-01-12'));
    const [view, setView] = useState('Week');

    const onCellClick = useCallback((args: SchedulerCellClickEvent) => { /* custom cell logic */ }, []);
    const onCellDoubleClick = useCallback(() => { /* e.g. open external form */ }, []);
    const onEventClick = useCallback((args: SchedulerEventClickEvent) => { /* show event details */ }, []);
    const onEventDoubleClick = useCallback(() => { /* typically opens the editor */ }, []);
    const onMoreEventsClick = useCallback(() => { /* open agenda for that date */ }, []);

    const onDragStart = useCallback(() => { /* validate / cancel drag */ }, []);
    const onDrag = useCallback(() => { /* real-time validation */ }, []);
    const onDragStop = useCallback(() => { /* final validation / persist */ }, []);

    const onResizeStart = useCallback(() => { /* validate or cancel resize */ }, []);
    const onResizing = useCallback(() => { /* live duration preview */ }, []);
    const onResizeStop = useCallback(() => { /* persist new duration */ }, []);

    const onSelectedDateChange = useCallback((args: SchedulerDateChangeEvent) => {
        setSelectedDate(args.value);
    }, []);

    const onViewChange = useCallback((args: SchedulerViewChangeEvent) => {
        setView(args.value);
    }, []);

    const onDataChangeStart = useCallback((args: SchedulerDataChangeEvent) => {
        // validate/persist; set args.cancel = true to revert (see events-data.md)
    }, []);

    const onDataChangeComplete = useCallback(() => { /* refresh dependent UI */ }, []);
    const onError = useCallback(() => { /* log or display errors */ }, []);

    const scheduler = useMemo(() => (
        <Scheduler
            height='34.375rem'
            view={view}
            startHour="08:00"
            selectedDate={selectedDate}
            eventSettings={{ dataSource: defaultData }}
            onCellClick={onCellClick}
            onCellDoubleClick={onCellDoubleClick}
            onEventClick={onEventClick}
            onEventDoubleClick={onEventDoubleClick}
            onMoreEventsClick={onMoreEventsClick}
            onDragStart={onDragStart}
            onDrag={onDrag}
            onDragStop={onDragStop}
            onResizeStart={onResizeStart}
            onResizing={onResizing}
            onResizeStop={onResizeStop}
            onSelectedDateChange={onSelectedDateChange}
            onViewChange={onViewChange}
            onDataChangeStart={onDataChangeStart}
            onDataChangeComplete={onDataChangeComplete}
            onError={onError}
        >
            <DayView />
            <WeekView />
            <WorkWeekView />
            <MonthView />
        </Scheduler>
    ), [
        view, selectedDate,
        onCellClick, onCellDoubleClick, onEventClick, onEventDoubleClick, onMoreEventsClick,
        onDragStart, onDrag, onDragStop,
        onResizeStart, onResizing, onResizeStop,
        onSelectedDateChange, onViewChange,
        onDataChangeStart, onDataChangeComplete, onError
    ]);

    return <>{scheduler}</>;
}
```

Wrap handlers in `useCallback` (and the scheduler JSX in `useMemo` with those handlers as dependencies) so re-renders don't churn the scheduler unnecessarily.

## Event argument shapes

- **`SchedulerDataChangeEvent`** (`onDataChangeStart`, `onDataChangeComplete`): carries `addedRecords`, `changedRecords`, `deletedRecords`, and `cancel` — full recipes in [events-data.md](events-data.md) ("Handling data changes").
- **`SchedulerDataRequestEvent`** (`onDataRequest`): carries `startDate`, `endDate`, and assignable `result` for load-on-demand — see [events-data.md](events-data.md) ("Load on demand").
- **`SchedulerDateChangeEvent` / `SchedulerViewChangeEvent`** (`onSelectedDateChange` / `onViewChange`): expose `value` (the new date or view name) — the backbone of controlled mode.
- **`SchedulerEventClickEvent`** (`onEventClick`, `onEventDoubleClick`): exposes `data` (the `EventModel`) and `cancel` — set `args.cancel = true` to suppress built-in popups (the external-form pattern in [crud-external-forms.md](crud-external-forms.md)).
- **`SchedulerCellClickEvent`** (`onCellClick`, `onCellDoubleClick`): exposes `startTime`, `endTime`, `isAllDay`, and `cancel`.
- **`SchedulerResizeEvent`** (`onResizeStart`, `onResizing`, `onResizeStop`): exposes `event`, `cancel`, `startTime`, `endTime`, and `data` — details in [crud-external-forms.md](crud-external-forms.md) ("Event resizing").
- **`SchedulerEditorSubmitEvent`** (`onEditorSubmit`): exposes `data` and `cancel` for save-time validation, and lets you merge custom form values into `args.data` — see [editor-and-popups.md](editor-and-popups.md).

## Which callbacks pair with which feature

| Feature | Relevant callbacks | Reference |
|---|---|---|
| Controlled date/view state | `onSelectedDateChange`, `onViewChange` | [getting-started.md](getting-started.md) |
| Local state sync after CRUD | `onDataChangeStart`, `onDataChangeComplete` | [events-data.md](events-data.md) |
| Load on demand | `onDataRequest` | [events-data.md](events-data.md) |
| Business-rule validation on edits | `onDataChangeStart` (via `cancel`) | [events-data.md](events-data.md), [solutions.md](solutions.md) |
| Drag / resize validation | `onDragStart`/`onDrag`/`onDragStop`, `onResizeStart`/`onResizing`/`onResizeStop` | [crud-external-forms.md](crud-external-forms.md) |
| Suppressing built-in popups (external forms) | `onCellClick`, `onCellDoubleClick`, `onEventClick`, `onEventDoubleClick` (via `cancel`) | [crud-external-forms.md](crud-external-forms.md) |
| Custom editor save validation | `onEditorSubmit` | [editor-and-popups.md](editor-and-popups.md) |
| "+n more" indicator | `onMoreEventsClick` | views (Month) |
