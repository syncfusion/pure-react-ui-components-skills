# Timeline Views

Timeline views display events in a horizontal, continuously scrollable timeline format. Unlike Day/Week/Month views which stack events vertically, timeline views render time slots horizontally with events flowing left-to-right, making them ideal for resource planning, project timelines, and multi-day event visualization.

## Timeline view types

The Scheduler provides four timeline view components:

| View | Purpose | Use Case |
|------|---------|----------|
| **TimelineDayView** | Horizontal timeline for a single day | Hourly schedule with detailed event placement |
| **TimelineWeekView** | Horizontal timeline for 7 consecutive days | Weekly planning across multiple days |
| **TimelineWorkWeekView** | Horizontal timeline for working days only | Business week viewing (Mon-Fri) |
| **TimelineMonthView** | Horizontal timeline for a month | Long-term planning and overview |

## Setting up timeline views

Render timeline views as children of the Scheduler component:

```tsx
import { Scheduler, TimelineDayView, TimelineWeekView, TimelineWorkWeekView, TimelineMonthView } from '@syncfusion/react-scheduler';
import { timelineEventData } from './dataSource';

export default function App() {
  const eventSettings = { dataSource: timelineEventData };

  return (
    <Scheduler
      height="34.375rem"
      defaultSelectedDate={new Date(2026, 0, 15)}
      eventSettings={eventSettings}
      defaultView="TimelineWeek"
      startHour="08:00"
      endHour="18:00"
    >
      <TimelineDayView />
      <TimelineWeekView />
      <TimelineWorkWeekView />
      <TimelineMonthView />
    </Scheduler>
  );
}
```

## Controlling visible periods with interval

The `interval` property controls how many periods (days/weeks/months) are displayed simultaneously. Use larger intervals for multi-period planning:

```tsx
import { Scheduler, TimelineDayView, TimelineWeekView, TimelineMonthView } from '@syncfusion/react-scheduler';
import { timelineEventData } from './dataSource';

export default function App() {
  return (
    <Scheduler
      height="34.375rem"
      defaultSelectedDate={new Date(2026, 0, 15)}
      eventSettings={{ dataSource: timelineEventData }}
      startHour="08:00"
    >
      {/* Show 3 consecutive days instead of 1 */}
      <TimelineDayView displayName="3 Days" interval={3} />
      
      {/* Show 2 consecutive weeks instead of 1 */}
      <TimelineWeekView displayName="2 Weeks" interval={2} />
      
      {/* Show 4 consecutive months instead of 1 */}
      <TimelineMonthView displayName="4 Months" interval={4} />
    </Scheduler>
  );
}
```

**Common interval values:**
- `TimelineDayView`: `1`, `3`, `5`, `7` (days)
- `TimelineWeekView`: `1`, `2`, `3`, `4` (weeks)
- `TimelineMonthView`: `1`, `2`, `3`, `6`, `12` (months)

## Customizing time scale granularity

Control the time-slot density using `timeScale` with `interval` (minutes per slot) and `slotCount` (sub-slots per main slot):

```tsx
import { useState } from 'react';
import { Scheduler, TimelineDayView, TimelineWeekView } from '@syncfusion/react-scheduler';
import { DropDownList } from '@syncfusion/react-dropdowns';
import { timelineEventData } from './dataSource';

export default function App() {
  const [interval, setInterval] = useState(60); // Minutes per slot
  const [slotCount, setSlotCount] = useState(2);  // Sub-slots per main slot

  const timeScale = {
    enable: true,
    interval: interval,    // 30, 60, 90, 120 minutes
    slotCount: slotCount   // 1, 2, 3, 4, 5, 6
  };

  return (
    <>
      <div style={{ marginBottom: '15px', display: 'flex', gap: '10px' }}>
        <DropDownList
          dataSource={[30, 60, 90, 120, 150, 180]}
          value={interval}
          placeholder="Slot interval (minutes)"
          onChange={(e) => setInterval(e.value as number)}
        />
        <DropDownList
          dataSource={[1, 2, 3, 4, 5, 6]}
          value={slotCount}
          placeholder="Sub-slots"
          onChange={(e) => setSlotCount(e.value as number)}
        />
      </div>

      <Scheduler
        height="34.375rem"
        defaultSelectedDate={new Date(2026, 0, 15)}
        eventSettings={{ dataSource: timelineEventData }}
        startHour="08:00"
        endHour="18:00"
        timeScale={timeScale}
      >
        <TimelineDayView />
        <TimelineWeekView />
      </Scheduler>
    </>
  );
}
```

**Time scale combinations:**
- Fine granularity: `interval: 15, slotCount: 4` (15-minute slots with 4 subdivisions)
- Standard: `interval: 30, slotCount: 2` (30-minute slots with 2 subdivisions)
- Coarse: `interval: 60, slotCount: 1` (1-hour slots, no subdivisions)

## Flexible working days

Define which days are working days using `workDays` array (0=Sunday through 6=Saturday):

```tsx
import { useState } from 'react';
import { Scheduler, TimelineWeekView, TimelineWorkWeekView } from '@syncfusion/react-scheduler';
import { DropDownList } from '@syncfusion/react-dropdowns';
import { Checkbox } from '@syncfusion/react-buttons';

export default function App() {
  const workDayOptions = [
    { text: 'Mon-Fri', workDays: [1, 2, 3, 4, 5] },
    { text: 'Mon, Wed, Fri', workDays: [1, 3, 5] },
    { text: 'Tue-Sat', workDays: [2, 3, 4, 5, 6] }
  ];

  const [selectedWorkDays, setSelectedWorkDays] = useState([1, 2, 3, 4, 5]);
  const [showWeekend, setShowWeekend] = useState(false);
  const [firstDayOfWeek, setFirstDayOfWeek] = useState(0); // 0=Sunday, 1=Monday

  return (
    <>
      <div style={{ marginBottom: '15px', display: 'flex', gap: '10px', alignItems: 'center' }}>
        <DropDownList
          dataSource={workDayOptions}
          fields={{ text: 'text', value: 'workDays' }}
          value={selectedWorkDays}
          placeholder="Work days"
          onChange={(e) => setSelectedWorkDays(e.value as number[])}
        />
        <Checkbox
          label="Show weekends"
          checked={showWeekend}
          onChange={(e) => setShowWeekend(e.value as boolean)}
        />
        <DropDownList
          dataSource={['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday']}
          value={firstDayOfWeek}
          placeholder="First day of week"
          onChange={(e) => setFirstDayOfWeek(e.value as number)}
        />
      </div>

      <Scheduler
        height="34.375rem"
        defaultSelectedDate={new Date(2026, 0, 15)}
        eventSettings={{ dataSource: timelineEventData }}
        defaultView="TimelineWeek"
        workDays={selectedWorkDays}
        showWeekend={showWeekend}
        firstDayOfWeek={firstDayOfWeek}
      >
        <TimelineWeekView />
        <TimelineWorkWeekView />
      </Scheduler>
    </>
  );
}
```

## Flexible working hours

Highlight working hours and set the business hours range using `workHours`:

```tsx
import { Scheduler, TimelineDayView, TimelineWeekView } from '@syncfusion/react-scheduler';
import { TimePicker } from '@syncfusion/react-calendars';
import { useState } from 'react';

export default function App() {
  const [startHour, setStartHour] = useState(new Date(2026, 0, 15, 7, 0));
  const [endHour, setEndHour] = useState(new Date(2026, 0, 15, 20, 0));

  const workHours = {
    highlight: true,
    start: '07:00', // Start of working hours
    end: '20:00'    // End of working hours
  };

  return (
    <>
      <div style={{ marginBottom: '15px', display: 'flex', gap: '10px' }}>
        <TimePicker
          placeholder="Start hour"
          value={startHour}
          onChange={(e) => setStartHour(e.value as Date)}
        />
        <TimePicker
          placeholder="End hour"
          value={endHour}
          onChange={(e) => setEndHour(e.value as Date)}
        />
      </div>

      <Scheduler
        height="34.375rem"
        defaultSelectedDate={new Date(2026, 0, 15)}
        eventSettings={{ dataSource: timelineEventData }}
        workHours={workHours}
        defaultView="TimelineDay"
      >
        <TimelineDayView />
        <TimelineWeekView />
      </Scheduler>
    </>
  );
}
```

## Auto-adjusting row height

Timeline views automatically adjust row height based on content. Control this with the `rowAutoHeight` property:

```tsx
<Scheduler
  height="34.375rem"
  rowAutoHeight={true} // Expands rows to fit event content
  eventSettings={{ dataSource: timelineEventData }}
  defaultView="TimelineWeek"
>
  <TimelineWeekView />
</Scheduler>
```

## Header rows for timeline context

Add header rows to show date/time hierarchy (useful for month views showing weeks and days):

```tsx
<Scheduler
  height="34.375rem"
  eventSettings={{ dataSource: timelineEventData }}
  defaultView="TimelineMonth"
>
  {/* TimelineMonthView automatically shows month/week/day headers */}
  <TimelineMonthView />
</Scheduler>
```

## Timeline with resource grouping

Combine timeline views with resources for resource-based timeline scheduling:

```tsx
import { Scheduler, TimelineWeekView } from '@syncfusion/react-scheduler';

export default function App() {
  const resources = [
    {
      field: 'ResourceId',
      title: 'Rooms',
      name: 'Rooms',
      dataSource: [
        { Text: 'Room 1', Id: 1, Color: '#0066cc' },
        { Text: 'Room 2', Id: 2, Color: '#ff6600' },
        { Text: 'Room 3', Id: 3, Color: '#00cc66' }
      ],
      textField: 'Text',
      idField: 'Id',
      colorField: 'Color'
    }
  ];

  const events = [
    {
      Id: 1,
      Subject: 'Conference',
      StartTime: new Date(2026, 0, 15, 9, 0),
      EndTime: new Date(2026, 0, 15, 11, 0),
      ResourceId: 1
    },
    {
      Id: 2,
      Subject: 'Training',
      StartTime: new Date(2026, 0, 15, 10, 0),
      EndTime: new Date(2026, 0, 15, 12, 0),
      ResourceId: 2
    }
  ];

  return (
    <Scheduler
      height="34.375rem"
      eventSettings={{ dataSource: events }}
      resources={resources}
      defaultView="TimelineWeek"
    >
      <TimelineWeekView />
    </Scheduler>
  );
}
```

## Common timeline use cases

### Project timeline with milestones
```tsx
// Display project phases (intervals=4 weeks each) with milestone events
<Scheduler defaultView="TimelineMonth">
  <TimelineMonthView interval={4} displayName="Project Timeline" />
</Scheduler>
```

### Resource availability planner
```tsx
// Show multiple resources across a timeline with event conflicts visible
<Scheduler defaultView="TimelineWeek" resources={roomResources}>
  <TimelineWeekView interval={2} />
</Scheduler>
```

### Hour-by-hour booking system
```tsx
// Fine-grained hourly view with 30-minute slots for exact booking times
<Scheduler
  timeScale={{ enable: true, interval: 30, slotCount: 2 }}
  defaultView="TimelineDay"
>
  <TimelineDayView />
</Scheduler>
```

## Related APIs

- [Scheduler.timeScale](https://react.syncfusion.com/react-ui/scheduler/#timescale)
- [Scheduler.workHours](https://react.syncfusion.com/react-ui/scheduler/#workhours)
- [Scheduler.workDays](https://react.syncfusion.com/react-ui/scheduler/#workdays)
- [TimelineWeekView](https://react.syncfusion.com/react-ui/scheduler/#timelineweekview)
- [TimelineMonthView](https://react.syncfusion.com/react-ui/scheduler/#timelinemonthview)
- [Scheduler.rowAutoHeight](https://react.syncfusion.com/react-ui/scheduler/#rowautoheight)
- [See resources-grouping.md](./resources-grouping.md) for resource-based timeline grouping
- [See views.md](./views.md) for overview of all view types
