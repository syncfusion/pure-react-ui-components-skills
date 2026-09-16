# Blockout Dates/Hours and Read-Only Events

Control scheduling constraints by marking time ranges as unavailable or making events non-editable. Both features enhance calendar control and prevent unintended modifications.

## Blockout dates and hours

Block specific dates or time ranges to prevent users from creating or modifying events during unavailable periods (e.g., lunch breaks, facility maintenance, holidays).

### Setting up blockout times

Add blockout events to your data with the `isBlock` field set to `true`:

```tsx
import { Scheduler, DayView, WeekView, WorkWeekView, MonthView, EventSettings } from '@syncfusion/react-scheduler';

const schedulerData = [
  // Normal events
  {
    Id: 1,
    Subject: 'Team Meeting',
    StartTime: new Date(2026, 5, 15, 10, 0),
    EndTime: new Date(2026, 5, 15, 11, 0),
    IsBlock: false
  },
  // Blockout: entire day unavailable
  {
    Id: 2,
    Subject: 'Holiday',
    StartTime: new Date(2026, 5, 14, 9, 0),
    EndTime: new Date(2026, 5, 14, 17, 0),
    IsBlock: true
  },
  // Blockout: lunch hour
  {
    Id: 3,
    Subject: 'Lunch',
    StartTime: new Date(2026, 5, 15, 12, 0),
    EndTime: new Date(2026, 5, 15, 13, 0),
    IsBlock: true
  }
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

export default function App() {
  return (
    <Scheduler height="550px" eventSettings={eventSettings} defaultSelectedDate={new Date(2026, 5, 15)}>
      <DayView startHour="09:00" endHour="18:00" />
      <WeekView startHour="09:00" endHour="18:00" />
      <WorkWeekView startHour="09:00" endHour="18:00" />
      <MonthView />
    </Scheduler>
  );
}
```

### Blockout visual styling

Blocked time slots are displayed with a distinct visual appearance (grayed out or with a different background color) to make them visually distinct from available time slots. You can customize the appearance using templates or CSS classes.

### Interaction behavior

When a time range is blocked:
- **Cell selection** - Clicking a blocked cell does not trigger the event creation flow
- **Drag and drop** - Events cannot be dragged into or out of blocked time ranges
- **Resizing** - Events cannot be resized into blocked time ranges
- **User feedback** - The disabled appearance indicates unavailability to the user

### Common blockout scenarios

**Daily lunch hour blockout:**
```tsx
const getLunchBlockouts = () => {
  const blockouts = [];
  const startDate = new Date(2026, 5, 15);
  for (let i = 0; i < 5; i++) {
    const date = new Date(startDate);
    date.setDate(date.getDate() + i);
    blockouts.push({
      Id: 1000 + i,
      Subject: 'Lunch',
      StartTime: new Date(date.getFullYear(), date.getMonth(), date.getDate(), 12, 0),
      EndTime: new Date(date.getFullYear(), date.getMonth(), date.getDate(), 13, 0),
      IsBlock: true
    });
  }
  return blockouts;
};

const schedulerData = [...normalEvents, ...getLunchBlockouts()];
```

**Facility maintenance blockout:**
```tsx
const maintenanceBlockout = {
  Id: 2000,
  Subject: 'Facility Maintenance',
  StartTime: new Date(2026, 5, 20, 9, 0),
  EndTime: new Date(2026, 5, 20, 17, 0),
  IsBlock: true
};

const schedulerData = [...events, maintenanceBlockout];
```

**Weekend blockout:**
```tsx
const getWeekendBlockouts = () => {
  const blockouts = [];
  let currentDate = new Date(2026, 5, 13); // Start on a Saturday
  for (let week = 0; week < 12; week++) {
    // Saturday
    blockouts.push({
      Id: 3000 + week * 2,
      Subject: 'Weekend',
      StartTime: new Date(currentDate),
      EndTime: new Date(new Date(currentDate).setDate(currentDate.getDate() + 1)),
      IsBlock: true
    });
    // Sunday
    blockouts.push({
      Id: 3000 + week * 2 + 1,
      Subject: 'Weekend',
      StartTime: new Date(new Date(currentDate).setDate(currentDate.getDate() + 1)),
      EndTime: new Date(new Date(currentDate).setDate(currentDate.getDate() + 2)),
      IsBlock: true
    });
    currentDate = new Date(new Date(currentDate).setDate(currentDate.getDate() + 7));
  }
  return blockouts;
};
```

## Read-only events

Restrict user interaction on individual events or the entire scheduler using the `readOnly` property. Read-only events can be viewed but not modified.

### Disable all event editing

Set the Scheduler's `readOnly` property to `true` to prevent all modifications:

```tsx
import { Scheduler, DayView, WeekView, MonthView, EventSettings } from '@syncfusion/react-scheduler';

const eventSettings: EventSettings = {
  dataSource: [
    {
      Id: 1,
      Subject: 'Read-only Team Meeting',
      StartTime: new Date(2026, 5, 15, 10, 0),
      EndTime: new Date(2026, 5, 15, 11, 0)
    }
  ]
};

export default function App() {
  return (
    <Scheduler 
      height="550px" 
      eventSettings={eventSettings} 
      defaultSelectedDate={new Date(2026, 5, 15)}
      readOnly={true}
    >
      <DayView />
      <WeekView />
      <MonthView />
    </Scheduler>
  );
}
```

When `readOnly={true}`:
- Event creation - Disabled
- Event editing - Disabled (editor dialog does not open on click)
- Drag and drop - Disabled
- Event resizing - Disabled
- Event deletion - Disabled
- **Allowed** - Viewing event details, navigating dates/views

### Make specific events read-only

Mark individual events as read-only using the `isReadonly` field:

```tsx
import { Scheduler, DayView, WeekView, MonthView, EventSettings } from '@syncfusion/react-scheduler';

const schedulerData = [
  {
    Id: 1,
    Subject: 'Past Event (Read-only)',
    StartTime: new Date(2026, 5, 15, 8, 0),
    EndTime: new Date(2026, 5, 15, 9, 0),
    isReadonly: true
  },
  {
    Id: 2,
    Subject: 'Editable Event',
    StartTime: new Date(2026, 5, 15, 10, 0),
    EndTime: new Date(2026, 5, 15, 11, 0),
    isReadonly: false
  },
  {
    Id: 3,
    Subject: 'Future Event (Editable)',
    StartTime: new Date(2026, 5, 15, 15, 0),
    EndTime: new Date(2026, 5, 15, 16, 0)
    // isReadonly defaults to false
  }
];

const eventSettings: EventSettings = {
  dataSource: schedulerData,
  fields: {
    id: 'Id',
    subject: 'Subject',
    startTime: 'StartTime',
    endTime: 'EndTime',
    isReadonly: 'isReadonly'
  }
};

export default function App() {
  return (
    <Scheduler height="550px" eventSettings={eventSettings} defaultSelectedDate={new Date(2026, 5, 15)}>
      <DayView />
      <WeekView />
      <MonthView />
    </Scheduler>
  );
}
```

### Conditional read-only logic

Make events read-only based on application logic:

```tsx
import { useMemo } from 'react';

const now = new Date();

const schedulerData = [
  {
    Id: 1,
    Subject: 'Past Event',
    StartTime: new Date(2026, 5, 10, 10, 0),
    EndTime: new Date(2026, 5, 10, 11, 0)
  },
  {
    Id: 2,
    Subject: 'Upcoming Event',
    StartTime: new Date(2026, 5, 20, 10, 0),
    EndTime: new Date(2026, 5, 20, 11, 0)
  }
];

export default function App() {
  const eventsWithReadonly = useMemo(() => {
    return schedulerData.map(event => ({
      ...event,
      // Mark past events as read-only
      isReadonly: event.EndTime < now
    }));
  }, []);

  const eventSettings: EventSettings = {
    dataSource: eventsWithReadonly,
    fields: {
      id: 'Id',
      subject: 'Subject',
      startTime: 'StartTime',
      endTime: 'EndTime',
      isReadonly: 'isReadonly'
    }
  };

  return (
    <Scheduler height="550px" eventSettings={eventSettings} defaultSelectedDate={new Date(2026, 5, 15)}>
      <DayView />
      <WeekView />
      <MonthView />
    </Scheduler>
  );
}
```

### Permission-based read-only

Apply read-only status based on user permissions:

```tsx
interface UserPermissions {
  canEditOwn: boolean;
  canEditAll: boolean;
  userId: number;
}

const currentUser: UserPermissions = {
  userId: 1,
  canEditOwn: true,
  canEditAll: false
};

const eventsWithPermissions = schedulerData.map(event => ({
  ...event,
  isReadonly: !currentUser.canEditAll && event.OwnerId !== currentUser.userId
}));
```

## Combining blockout and read-only

Use both features together for comprehensive scheduling control:

```tsx
const schedulerData = [
  // Normal editable events
  {
    Id: 1,
    Subject: 'Team Meeting',
    StartTime: new Date(2026, 5, 15, 10, 0),
    EndTime: new Date(2026, 5, 15, 11, 0),
    IsBlock: false,
    isReadonly: false
  },
  // Past event (read-only)
  {
    Id: 2,
    Subject: 'Completed Task',
    StartTime: new Date(2026, 5, 14, 10, 0),
    EndTime: new Date(2026, 5, 14, 11, 0),
    IsBlock: false,
    isReadonly: true
  },
  // Blocked lunch hour
  {
    Id: 3,
    Subject: 'Lunch',
    StartTime: new Date(2026, 5, 15, 12, 0),
    EndTime: new Date(2026, 5, 15, 13, 0),
    IsBlock: true,
    isReadonly: false
  }
];

const eventSettings: EventSettings = {
  dataSource: schedulerData,
  fields: {
    id: 'Id',
    subject: 'Subject',
    startTime: 'StartTime',
    endTime: 'EndTime',
    isBlock: 'IsBlock',
    isReadonly: 'isReadonly'
  }
};
```

## Related APIs

- [Scheduler.readOnly](https://react.syncfusion.com/react-ui/scheduler/#prop-readOnly)
- [EventSettings.fields.isBlock](https://react.syncfusion.com/react-ui/scheduler/#prop-eventsettings)
- [EventSettings.fields.isReadonly](https://react.syncfusion.com/react-ui/scheduler/#prop-eventsettings)
- [Event templates and styling](./templates-ui.md)
