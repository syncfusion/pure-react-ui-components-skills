# Event Types: All-Day, Spanned, Overlapping, Blocked, and Read-Only Events

Beyond normal timed events, the scheduler supports all-day events, spanned (multi-day) events, overlap control, blocked time ranges, and per-event or global read-only behavior.

## Normal events

Normal events are appointments with defined start and end times rendered in the time slots:

```tsx
import { DayView, WeekView, WorkWeekView, MonthView, EventSettings, Scheduler } from '@syncfusion/react-scheduler';

export default function App() {
    const schedulerData = [
        {
            Id: 1,
            Subject: 'Normal event',
            StartTime: new Date(2026, 5, 15, 2, 0),
            EndTime: new Date(2026, 5, 15, 5, 30),
        }
    ];
    const eventSettings: EventSettings = {
        dataSource: schedulerData,
        fields: { id: 'Id', subject: 'Subject', startTime: 'StartTime', endTime: 'EndTime' }
    };

    return (
        <Scheduler height='34.375rem' defaultSelectedDate={new Date(2026, 5, 15)} eventSettings={eventSettings}>
            <DayView /><WeekView /><WorkWeekView /><MonthView />
        </Scheduler>
    );
}
```

## Spanned events

Spanned events extend beyond a single day. Events **longer than 24 hours** appear in the all-day row; **cross-day events under 24 hours** are split and shown on each affected day. For example, an event from April 20, 2026 at 4:00 AM to April 21, 2026 at 2:30 AM (less than 24 hours) renders as two segments, one on each day.

```tsx
const schedulerData = [
    { Id: 1, Subject: 'Less Than 24 Hours', StartTime: new Date(2026, 3, 20, 4, 0), EndTime: new Date(2026, 3, 21, 2, 30) },
    { Id: 2, Subject: 'Greater Than 24 Hours', StartTime: new Date(2026, 3, 22, 0, 0), EndTime: new Date(2026, 3, 24, 0, 0) }
];
```

### Customizing spanned event rendering

By default, events longer than 24 hours render in the all-day row. Set `eventSettings.spannedEventPlacement="TimeSlot"` to render long events directly in the time grid instead:

```tsx
const eventSettings: EventSettings = {
    dataSource: schedulerData,
    spannedEventPlacement: 'TimeSlot',
    fields: { id: 'Id', subject: 'Subject', startTime: 'StartTime', endTime: 'EndTime' }
};
```

## All-day events

All-day events occupy the entire day (e.g. holidays) and appear in a dedicated all-day row below the date header. Set `isAllDay` to `true` to make an event all-day:

```tsx
import { DayView, WeekView, WorkWeekView, MonthView, EventSettings, Scheduler } from '@syncfusion/react-scheduler';

export default function App() {
    const schedulerData = [
        {
            Id: 1,
            Subject: 'All-day event',
            StartTime: new Date(2026, 5, 15, 2, 0),
            EndTime: new Date(2026, 5, 15, 4, 0),
            IsAllDay: true
        }
    ];
    const eventSettings: EventSettings = {
        dataSource: schedulerData,
        fields: { id: 'Id', subject: 'Subject', startTime: 'StartTime', endTime: 'EndTime', isAllDay: 'IsAllDay' }
    };

    return (
        <Scheduler defaultSelectedDate={new Date(2026, 5, 15)} eventSettings={eventSettings} eventDrag={false} eventResize={false}>
            <DayView /><WeekView /><WorkWeekView /><MonthView />
        </Scheduler>
    );
}
```

> All-day events are treated as date-based entries and are not timezone-converted (see [globalization-accessibility.md](globalization-accessibility.md)).

## Overlapping events

By default, overlapping appointments display side-by-side — each event shares the available width proportionally:

```tsx
const schedulerData = [
    { Id: 1, Subject: 'Client Meeting', StartTime: new Date(2026, 5, 15, 9, 0), EndTime: new Date(2026, 5, 15, 11, 0) },
    { Id: 2, Subject: 'Tech Symposium', StartTime: new Date(2026, 5, 15, 9, 30), EndTime: new Date(2026, 5, 15, 10, 30) },
    { Id: 3, Subject: 'Project Review', StartTime: new Date(2026, 5, 15, 10, 0), EndTime: new Date(2026, 5, 15, 11, 30) }
];
```

### Preventing overlaps

Set `eventOverlap={false}` to reject new or updated events that conflict with existing appointments, showing a conflict warning. On initial load, the scheduler arranges non-overlapping events by prioritizing duration and all-day status — longer and all-day events take precedence. When adding or editing, overlaps are detected and blocked with an alert:

```tsx
<Scheduler eventSettings={eventSettings} eventOverlap={false} eventDrag={true} eventResize={true}>
    <DayView />
    <WeekView startHour="08:00" endHour="20:00" />
    <WorkWeekView />
    <MonthView />
</Scheduler>
```

## Block dates and hours

Block specific dates or time ranges to prevent creating or modifying events during unavailable periods. Add an event with `startTime`/`endTime` and `isBlock: true`; those cells are marked unavailable and interactions within them are disallowed:

```tsx
import { DayView, WeekView, WorkWeekView, MonthView, EventSettings, Scheduler } from '@syncfusion/react-scheduler';

export default function App() {
    const schedulerData = [
        { Id: 1, Subject: 'Life on Mars', StartTime: new Date(2026, 5, 15, 9, 30), EndTime: new Date(2026, 5, 15, 10, 30), IsBlock: false },
        { Id: 2, Subject: 'Lifecycle of Ants', StartTime: new Date(2026, 5, 16, 10, 30), EndTime: new Date(2026, 5, 16, 11, 30), IsBlock: false },
        { Id: 3, Subject: 'Mysteries of Pyramid', StartTime: new Date(2026, 5, 17, 15, 30), EndTime: new Date(2026, 5, 17, 17, 30), IsBlock: false },
        { Id: 4, Subject: 'Rainbow Effect', StartTime: new Date(2026, 5, 18, 9, 45), EndTime: new Date(2026, 5, 18, 11, 0), IsBlock: false },
        { Id: 5, Subject: 'Not Available', StartTime: new Date(2026, 5, 14, 9, 0), EndTime: new Date(2026, 5, 14, 20, 0), IsBlock: true },
        { Id: 6, Subject: 'Not Available', StartTime: new Date(2026, 5, 20, 9, 0), EndTime: new Date(2026, 5, 20, 20, 0), IsBlock: true },
        { Id: 7, Subject: 'Lunch', StartTime: new Date(2026, 5, 15, 13, 0), EndTime: new Date(2026, 5, 15, 14, 0), IsBlock: true },
        { Id: 8, Subject: 'Lunch', StartTime: new Date(2026, 5, 16, 13, 0), EndTime: new Date(2026, 5, 16, 14, 0), IsBlock: true },
        { Id: 9, Subject: 'Lunch', StartTime: new Date(2026, 5, 17, 13, 0), EndTime: new Date(2026, 5, 17, 14, 0), IsBlock: true },
        { Id: 10, Subject: 'Lunch', StartTime: new Date(2026, 5, 18, 13, 0), EndTime: new Date(2026, 5, 18, 14, 0), IsBlock: true },
        { Id: 11, Subject: 'Lunch', StartTime: new Date(2026, 5, 19, 13, 0), EndTime: new Date(2026, 5, 19, 14, 0), IsBlock: true },
        { Id: 12, Subject: 'Not Available', StartTime: new Date(2026, 5, 17, 11, 0), EndTime: new Date(2026, 5, 17, 12, 0), IsBlock: true },
        { Id: 13, Subject: 'Not Available', StartTime: new Date(2026, 5, 19, 9, 30), EndTime: new Date(2026, 5, 19, 10, 30), IsBlock: true }
    ];
    const eventSettings: EventSettings = {
        dataSource: schedulerData,
        fields: {
            id: 'Id',
            subject: 'Subject',
            startTime: 'StartTime',
            endTime: 'EndTime',
            isBlock: 'IsBlock'
        }
    };

    return (
        <Scheduler height='34.375rem' defaultView='WorkWeek' defaultSelectedDate={new Date(2026, 5, 15)} eventSettings={eventSettings}>
            <DayView />
            <WeekView startHour="09:00" endHour="20:00" />
            <WorkWeekView startHour="09:00" endHour="18:00" />
            <MonthView />
        </Scheduler>
    );
}
```

## Read-only events

### Making the entire scheduler read-only

The `readOnly` property (default `false`) restricts user interaction: users can still view appointment details and navigate between dates and views, but cannot create, edit, drag, resize, or delete events:

```tsx
<Scheduler eventSettings={eventSettings} readOnly={true}>
    <DayView /><WeekView /><WorkWeekView /><MonthView />
</Scheduler>
```

### Making specific events read-only

Restrict editing on individual events via the `isReadonly` field — e.g. mark past events (those ended before the current date) read-only while keeping CRUD on others. By default, the event editor does not open for events with `isReadonly` set to `true`:

```tsx
const schedulerData = [
    { Id: 1, Subject: 'Past event (read-only)', StartTime: new Date(2026, 5, 15, 8, 0), EndTime: new Date(2026, 5, 15, 9, 0), isReadonly: true },
    { Id: 2, Subject: 'Ongoing event', StartTime: new Date(2026, 5, 15, 11, 30), EndTime: new Date(2026, 5, 15, 13, 0) },
    { Id: 3, Subject: 'Future event', StartTime: new Date(2026, 5, 15, 15, 0), EndTime: new Date(2026, 5, 15, 16, 0) }
];
const eventSettings: EventSettings = {
    dataSource: schedulerData,
    fields: { id: 'Id', subject: 'Subject', startTime: 'StartTime', endTime: 'EndTime', isReadonly: 'IsReadonly' }
};
```
