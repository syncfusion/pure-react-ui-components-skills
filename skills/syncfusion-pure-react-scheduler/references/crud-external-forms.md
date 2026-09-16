# CRUD Actions, Drag-and-Drop, Resizing, and External Forms

Events (also called appointments) are the primary interactive elements. Create, update, and remove appointments via the editor dialog, quick info popup, drag-and-drop and resize gestures, or programmatically through scheduler methods — or replace the built-in UI entirely with an external form.

## Add new events

Users create appointments by clicking a scheduler cell (quick info popup) or double-clicking (detailed editor dialog). Prevent new event creation with `allowAdding={false}` on `eventSettings`.

## Edit existing events

Users modify appointments via the detailed editor dialog (double-click) or quick info popup (single-click); changes take effect immediately upon save. Prevent editing with `allowEditing={false}`.

## Delete events

Users remove appointments from the quick info popup (single-click) or by pressing the Delete key, with a confirmation prompt. Prevent deletion with `allowDeleting={false}`.

All three CRUD toggles live on `eventSettings`:

```tsx
import { useMemo, useRef, useState } from 'react';
import { DayView, WeekView, WorkWeekView, MonthView, EventSettings, Scheduler } from '@syncfusion/react-scheduler';

const eventSettings: EventSettings = {
    dataSource: events,
    allowAdding: true,
    allowEditing: true,
    allowDeleting: true,
    fields: { id: 'Id', subject: 'Subject', startTime: 'StartTime', endTime: 'EndTime', isAllDay: 'IsAllDay' }
};
```

## Drag-and-drop

Reschedule appointments by dragging and dropping them to new time slots. Drag-and-drop is enabled by default (`eventDrag={true}`); set `eventDrag={false}` to disable it. On mobile devices, users tap and hold an event, then drag it to the desired location:

```tsx
import { SchedulerDataChangeEvent } from '@syncfusion/react-scheduler';

<Scheduler eventDrag={eventDrag} eventResize={false} onDataChangeStart={onDataChangeStart}>
    <DayView />
    <WeekView />
    <WorkWeekView />
    <MonthView />
</Scheduler>

// Apply the user's drag result back to local state
const onDataChangeStart = (args: SchedulerDataChangeEvent) => {
    if (args) {
        setSchedulerData((data: Record<string, any>[]) => data
            .filter((item) => args.deletedRecords?.find((current) => current.Id === item.Id) === undefined)
            .map((item) => args.changedRecords?.find((current) => current.Id === item.Id) || item)
            .concat(args.addedRecords?.map((item) => item) || [])
        );
    }
};
```

### Drag lifecycle callbacks

Three callbacks manage the drag lifecycle (see also [events-api.md](events-api.md)):

- **`onDragStart`**: fires when dragging begins — cancel the drag, enforce restrictions, or show visual feedback.
- **`onDrag`**: fires continuously during dragging — real-time validation or updating dependent UI.
- **`onDragStop`**: fires when dragging ends — final validation or persisting changes to the data source.

## Event resizing

Adjust appointment duration and timing by resizing from edge handles (top, bottom, left, right) available across all views. Vertical resizing adjusts duration (extends or shortens the time slot); horizontal resizing shifts the appointment position in Week and WorkWeek views. Handlers appear automatically on desktop; on mobile, long-press an appointment to reveal touch-friendly resize dot handles.

`eventResize` accepts a **boolean** or an **EventResizeProps** configuration object:

```tsx
// Default behavior
<Scheduler eventResize={true}>...</Scheduler>
```

### Resize configuration

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `enable` | `Boolean` | `true` | Enable or disable event resizing. |
| `interval` | `Number` | `30` | Time interval (in minutes) for snapping resize operations — e.g. `interval: 15` snaps to 15-minute increments. |
| `resizeToZero` | `Boolean` | `false` | Allows resizing events to zero duration (start equals end). Useful for point-in-time events. |
| `startResizable` | `Boolean` | `true` | Resizing from the top/left edge of events (adjust event start time). |
| `endResizable` | `Boolean` | `true` | Resizing from the bottom/right edge of events (adjust event end time). |
| `scroll.enable` | `Boolean` | `true` | Auto-scrolls during resizing when the cursor reaches the scheduler edges. |
| `scroll.scrollBy` | `Number` | `10` | Pixels to scroll per auto-scroll trigger. |
| `scroll.timeDelay` | `Number` | `100` | Delay in milliseconds before auto-scrolling begins. |

```tsx
<Scheduler
    eventResize={{
        enable: true,
        interval: 10,
        startResizable: true,
        endResizable: true,
        scroll: { enable: true, scrollBy: 10, timeDelay: 100 },
    }}
    startHour='04:00'
>
    <DayView /><WeekView /><WorkWeekView /><MonthView />
</Scheduler>
```

### Resize lifecycle callbacks

- **`onResizeStart`**: fires when the user begins resizing — validate or cancel the operation.
- **`onResizing`**: fires continuously while resizing — real-time duration validation or preview updates.
- **`onResizeStop`**: fires when resizing ends — final validation or persisting the new duration.

Each callback receives a `SchedulerResizeEvent`:

| Property | Type | Description |
|----------|------|-------------|
| `event` | `MouseEvent \| TouchEvent` | The mouse or touch event that triggered the resize action. |
| `cancel` | `Boolean` | Set to `true` to cancel the resize operation. |
| `startTime` | `Date` | The current start time of the event during the resize. |
| `endTime` | `Date` | The current end time of the event during the resize. |
| `data` | `EventModel` | The appointment object being resized — all event properties including `id`, `subject`, `startTime`, `endTime`, `location`, and custom fields. |

## Programmatic event operations

Scheduler methods (via a component ref of type `IScheduler`) perform CRUD from code:

- `addEvent(data)` — add a single appointment object or an array of appointments to the data source.
- `saveEvent(data)` — update an appointment; the modified object must carry a valid `id` existing in the data source.
- `deleteEvent(idOrEvent)` — delete by the event's `id` or the entire event object.
- `openEditor(action, data)` — open the editor for `'Add'` or `'Edit'`; for recurring events also `'EditOccurrence'`/`'EditSeries'`.
- `deleteEvent(event, 'DeleteOccurrence' | 'DeleteSeries')` — delete a single occurrence or the whole series.
- `closeQuickInfoPopup()` — close the quick popup.

```tsx
import { useRef } from 'react';
import { Scheduler, IScheduler } from '@syncfusion/react-scheduler';

const schedulerRef = useRef<IScheduler | null>(null);

schedulerRef.current?.addEvent({ Id: 101, Subject: 'New Event', StartTime: start, EndTime: end });
schedulerRef.current?.saveEvent({ Id: 1, Subject: 'Updated', ... });
schedulerRef.current?.deleteEvent(101);
```

## External form editing

Replace the built-in dialog/quick popup with your own form wired to the scheduler methods. The pattern:

- Set `args.cancel = true` in `onCellClick`, `onCellDoubleClick`, `onEventClick`, and `onEventDoubleClick` to suppress the built-in popups and instead populate the external form.
- Click an appointment to populate the form (Subject, Location, Start/End dates, All-day status).
- **Save** updates an existing event (`saveEvent`) or creates a new one (`addEvent`); **Delete** removes the selected event; **Cancel** clears the form.

```tsx
import { useMemo, useRef, useState } from "react";
import { Scheduler, DayView, WeekView, WorkWeekView, MonthView, IScheduler, EventSettings, SchedulerCellClickEvent, SchedulerEventClickEvent } from "@syncfusion/react-scheduler";
import { Form, FormField, TextBox, TextBoxChangeEvent } from "@syncfusion/react-inputs";
import { Button, Variant, Checkbox, CheckboxChangeEvent } from "@syncfusion/react-buttons";
import { DatePicker, TimePicker } from "@syncfusion/react-calendars";

export default function App() {
  const eventsRef = useRef<EventItem[]>([
    { Id: 1, Subject: "Design", Location: "Room 1", StartTime: new Date(2026, 5, 15, 9, 0), EndTime: new Date(2026, 5, 15, 10, 0) },
    { Id: 2, Subject: "Development", Location: "Room 2", StartTime: new Date(2026, 5, 15, 11, 0), EndTime: new Date(2026, 5, 15, 12, 30) },
  ]);
  const schedulerRef = useRef<IScheduler | null>(null);

  const [formData, setFormData] = useState<{
    Id?: number; Subject: string; Location: string;
    StartTime?: Date; EndTime?: Date; IsAllDay: boolean; isEditing: boolean;
  }>({ Subject: "Add title", Location: "", StartTime: new Date(2026, 5, 15, 9, 0), EndTime: new Date(2026, 5, 15, 10, 0), IsAllDay: false, isEditing: false });

  const nextId = useMemo(() => {
    let n = Math.max(0, ...eventsRef.current.map((e) => e.Id));
    return () => (n = n + 1);
  }, []);

  const clearForm = () => setFormData({ Subject: "Add title", Location: "", StartTime: new Date(2026, 5, 15, 9, 0), EndTime: new Date(2026, 5, 15, 10, 0), IsAllDay: false, isEditing: false });

  // Click an appointment -> populate the form (suppress built-in popup)
  const onEventClick = (args: SchedulerEventClickEvent) => {
    args.cancel = true;
    setFormData({
      Id: args.data.id as number,
      Subject: (args.data.subject as string) ?? "",
      Location: (args.data.location as string) ?? "",
      StartTime: args.data.startTime ? new Date(args.data.startTime) : undefined,
      EndTime: args.data.endTime ? new Date(args.data.endTime) : undefined,
      IsAllDay: Boolean(args.data.isAllDay),
      isEditing: true,
    });
  };

  // Click a cell -> prepare a new event form
  const onCellClick = (args: SchedulerCellClickEvent) => {
    (args as any).cancel = true;
    setFormData({
      Id: undefined, Subject: "Add title", Location: "",
      StartTime: args.startTime ? new Date(args.startTime) : undefined,
      EndTime: args.endTime ? new Date(args.endTime) : undefined,
      IsAllDay: false, isEditing: false,
    });
  };

  const onEventDoubleClick = (args: SchedulerEventClickEvent) => { args.cancel = true; };
  const onCellDoubleClick = (args: SchedulerCellClickEvent) => { (args as any).cancel = true; };

  const eventSettings: EventSettings = {
    dataSource: eventsRef.current,
    fields: { id: "Id", subject: "Subject", location: "Location", startTime: "StartTime", endTime: "EndTime", isAllDay: "IsAllDay" },
  };

  // Combine date-part and time-part pickers into full Date values
  const setDatePart = (base: Date | undefined, d: Date | null) => {
    if (!d) return base;
    const b = base ?? d;
    const res = new Date(b);
    res.setFullYear(d.getFullYear(), d.getMonth(), d.getDate());
    return res;
  };
  const setTimePart = (base: Date | undefined, t: Date | null) => {
    if (!t) return base;
    const b = base ?? t;
    const res = new Date(b);
    res.setHours(t.getHours(), t.getMinutes(), 0, 0);
    return res;
  };

  const onSave = () => {
    const data = {
      Id: formData.Id ?? nextId(),
      Subject: formData.Subject?.trim() || "Add title",
      Location: formData.Location?.trim() || "",
      StartTime: formData.StartTime ?? new Date(),
      EndTime: formData.EndTime ?? new Date(),
      IsAllDay: !!formData.IsAllDay,
    } as unknown as Record<string, unknown>;

    if (formData.isEditing && formData.Id != null) {
      schedulerRef.current?.saveEvent(data);
    } else {
      schedulerRef.current?.addEvent(data);
    }
    clearForm();
  };

  const onDelete = () => {
    if (formData.isEditing && formData.Id != null) {
      schedulerRef.current?.deleteEvent(formData.Id);
      clearForm();
    }
  };

  return (
    <div>
      {/* Your external form: TextBoxes for Subject/Location, DatePicker+TimePicker pairs
          for Start/End, a Checkbox for IsAllDay, and Delete/Save/Cancel Buttons.
          The skeleton:
        <Form rules={{ Subject: {}, Location: {}, StartTime: {}, EndTime: {} }}>
          ...FormField name="Subject"><TextBox value={formData.Subject} .../></FormField...
          ...Buttons with onClick={onSave} / onDelete / onCancel...
        </Form>
      */}
      <Scheduler
        ref={schedulerRef}
        height="40.625rem"
        defaultSelectedDate={new Date(2026, 5, 15)}
        eventSettings={eventSettings}
        onEventClick={onEventClick}
        onCellClick={onCellClick}
        onEventDoubleClick={onEventDoubleClick}
        onCellDoubleClick={onCellDoubleClick}
      >
        <DayView /><WeekView /><WorkWeekView /><MonthView />
      </Scheduler>
    </div>
  );
}
```

## Handling data changes

On every add, edit, or delete, the scheduler invokes `onDataChangeStart` and `onDataChangeComplete` with `addedRecords`, `changedRecords`, and `deletedRecords` collections (and `cancel` to abort). This is where you sync React state, show messages, persist to a server, or revert on failure — full details and recipes are in [events-data.md](events-data.md) under "Handling data changes".
