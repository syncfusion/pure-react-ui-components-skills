# Recurrence Exceptions

Recurrence exceptions allow you to exclude specific occurrences from a recurring event series or create unique modifications for individual occurrences while keeping the series intact.

## Understanding recurrence exceptions

A recurring event repeats according to its `recurrenceRule`. Exceptions let you:
- **Skip specific dates** — exclude certain occurrences
- **Modify single occurrences** — change a specific date without affecting the series
- **Create occurrence-specific events** — add a replacement event for a skipped date

**Example:** A daily standup meeting (Mon-Fri) that doesn't occur on holidays.

## Excluding dates from recurrence

Use the `recurrenceException` field to specify dates that should NOT occur:

```tsx
import { Scheduler, WeekView, EventSettings } from '@syncfusion/react-scheduler';

export default function App() {
  const events = [
    {
      id: 1,
      subject: 'Daily Standup',
      startTime: new Date(2026, 0, 26, 10, 0),
      endTime: new Date(2026, 0, 26, 10, 30),
      // Repeat daily for 10 days
      recurrenceRule: 'FREQ=DAILY;COUNT=10',
      // Skip: Jan 31 at 10:00, Feb 2 at 10:00 (UTC format required)
      recurrenceException: '20260131T100000Z,20260202T100000Z'
    }
  ];

  const eventSettings: EventSettings = {
    dataSource: events,
    fields: {
      id: 'id',
      subject: 'subject',
      startTime: 'startTime',
      endTime: 'endTime',
      recurrenceRule: 'recurrenceRule',
      recurrenceException: 'recurrenceException'
    }
  };

  return (
    <Scheduler eventSettings={eventSettings} defaultSelectedDate={new Date(2026, 0, 26)}>
      <WeekView />
    </Scheduler>
  );
}
```

## Exception date format

Exception dates use **ISO 8601 UTC format** without separators: `YYYYMMDDTHHMMSSZ`

| Date/Time | Exception String | Notes |
|-----------|------------------|-------|
| Jan 31, 2026 @ 10:00 AM UTC | `20260131T100000Z` | Z = UTC timezone |
| Feb 2, 2026 @ 2:30 PM UTC | `20260202T143000Z` | 14:30 = 2:30 PM (24-hour) |
| Mar 15, 2026 @ 12:00 AM UTC | `20260315T000000Z` | Midnight |

**Conversion helper:**

```tsx
const formatExceptionDate = (date: Date): string => {
  const year = date.getUTCFullYear();
  const month = String(date.getUTCMonth() + 1).padStart(2, '0');
  const day = String(date.getUTCDate()).padStart(2, '0');
  const hours = String(date.getUTCHours()).padStart(2, '0');
  const minutes = String(date.getUTCMinutes()).padStart(2, '0');
  const seconds = String(date.getUTCSeconds()).padStart(2, '0');
  return `${year}${month}${day}T${hours}${minutes}${seconds}Z`;
};

// Usage
const exceptionDate = new Date(2026, 0, 31, 10, 0); // Local: Jan 31, 10:00 AM
const exceptionString = formatExceptionDate(exceptionDate);
console.log(exceptionString); // "20260131T100000Z" (if local timezone offset is 0)
```

**⚠️ Timezone considerations:**
Exception dates must match the UTC time of the recurring event's start time. If your event is at 10:00 AM in your local timezone, convert to UTC first:

```tsx
// Event in EST (UTC-5)
const eventStart = new Date(2026, 0, 26, 10, 0); // Jan 26, 10:00 AM EST

// To create an exception for Jan 31 at the same local time:
// 10:00 AM EST = 15:00 UTC
const exceptionDate = new Date(2026, 0, 31, 15, 0); // Convert to UTC hours
const exceptionString = formatExceptionDate(exceptionDate);
console.log(exceptionString); // "20260131T150000Z"
```

## Multiple exceptions (comma-separated)

List multiple exceptions as comma-separated strings:

```tsx
const events = [
  {
    id: 1,
    subject: 'Team Meeting',
    startTime: new Date(2026, 0, 5, 14, 0),
    endTime: new Date(2026, 0, 5, 15, 0),
    recurrenceRule: 'FREQ=WEEKLY;BYDAY=MO;COUNT=12',
    // Skip: Feb 2, Feb 23, Mar 9, Mar 23
    recurrenceException: '20260202T140000Z,20260223T140000Z,20260309T140000Z,20260323T140000Z'
  }
];
```

## Modifying a single occurrence

To change a specific occurrence without affecting the series, create a new event with:
- `recurrenceID` — points to the parent recurring event's ID
- Same subject/time zone as parent
- The modified start/end times

```tsx
const events = [
  {
    // Parent recurring event
    id: 1,
    subject: 'Daily Standup',
    startTime: new Date(2026, 0, 26, 10, 0),
    endTime: new Date(2026, 0, 26, 10, 30),
    recurrenceRule: 'FREQ=DAILY;COUNT=10'
  },
  {
    // Exception event: Feb 2 moved to 11:00 AM
    id: 2,
    subject: 'Daily Standup',
    startTime: new Date(2026, 2, 2, 11, 0),  // Same date, different time
    endTime: new Date(2026, 2, 2, 11, 30),
    recurrenceID: 1  // Links to parent recurring event
  }
];

const eventSettings: EventSettings = {
  dataSource: events,
  fields: {
    id: 'id',
    subject: 'subject',
    startTime: 'startTime',
    endTime: 'endTime',
    recurrenceRule: 'recurrenceRule',
    recurrenceID: 'recurrenceID'
  }
};
```

## Replacing an occurrence

Create an exception event to replace a skipped occurrence:

```tsx
const events = [
  {
    // Parent recurring event
    id: 1,
    subject: 'Daily Standup',
    startTime: new Date(2026, 0, 26, 10, 0),
    endTime: new Date(2026, 0, 26, 10, 30),
    recurrenceRule: 'FREQ=DAILY;COUNT=10',
    // Skip Jan 31
    recurrenceException: '20260131T100000Z'
  },
  {
    // Replacement event for Jan 31: moved to 2:00 PM
    id: 2,
    subject: 'Daily Standup (Rescheduled)',
    startTime: new Date(2026, 0, 31, 14, 0),
    endTime: new Date(2026, 0, 31, 14, 30),
    recurrenceID: 1
  }
];
```

## Programmatically managing exceptions

Add or remove exceptions dynamically:

```tsx
import { Scheduler, WeekView, EventSettings } from '@syncfusion/react-scheduler';
import { useState } from 'react';

export default function App() {
  const [events, setEvents] = useState([
    {
      id: 1,
      subject: 'Weekly Team Meeting',
      startTime: new Date(2026, 0, 26, 10, 0),
      endTime: new Date(2026, 0, 26, 11, 0),
      recurrenceRule: 'FREQ=WEEKLY;BYDAY=MO;COUNT=12',
      recurrenceException: ''
    }
  ]);

  const addException = (date: Date) => {
    const exceptionString = formatExceptionDate(date);
    setEvents(prev =>
      prev.map(event => ({
        ...event,
        recurrenceException: event.recurrenceException
          ? `${event.recurrenceException},${exceptionString}`
          : exceptionString
      }))
    );
  };

  const removeException = (date: Date) => {
    const exceptionString = formatExceptionDate(date);
    setEvents(prev =>
      prev.map(event => ({
        ...event,
        recurrenceException: event.recurrenceException
          .split(',')
          .filter(e => e !== exceptionString)
          .join(',')
      }))
    );
  };

  return (
    <>
      <button onClick={() => addException(new Date(2026, 1, 2))}>
        Skip Feb 2
      </button>
      <button onClick={() => removeException(new Date(2026, 1, 2))}>
        Restore Feb 2
      </button>
      <Scheduler eventSettings={{ dataSource: events }} defaultSelectedDate={new Date(2026, 0, 26)}>
        <WeekView />
      </Scheduler>
    </>
  );
}

const formatExceptionDate = (date: Date): string => {
  const year = date.getUTCFullYear();
  const month = String(date.getUTCMonth() + 1).padStart(2, '0');
  const day = String(date.getUTCDate()).padStart(2, '0');
  const hours = String(date.getUTCHours()).padStart(2, '0');
  const minutes = String(date.getUTCMinutes()).padStart(2, '0');
  const seconds = String(date.getUTCSeconds()).padStart(2, '0');
  return `${year}${month}${day}T${hours}${minutes}${seconds}Z`;
};
```

## Real-world use cases

### Holiday schedule (skip recurring workday event)
```tsx
{
  id: 1,
  subject: 'Daily Standup',
  recurrenceRule: 'FREQ=DAILY;BYDAY=MO,TU,WE,TH,FR',
  // Skip: New Year, Thanksgiving, Christmas
  recurrenceException: '20260101T100000Z,20261126T100000Z,20261225T100000Z'
}
```

### Vacation coverage (replace single occurrence)
```tsx
// Original recurring event
{ id: 1, subject: 'John\'s Office Hours', recurrenceRule: 'FREQ=DAILY' }

// During John's vacation week, create coverage events
{
  id: 2,
  subject: 'Jane\'s Office Hours (Covering)',
  startTime: new Date(2026, 1, 15, 10, 0),
  recurrenceID: 1  // Replace Feb 15 occurrence
}
```

### One-time event cancellation
```tsx
{
  id: 1,
  subject: 'Weekly Meeting',
  recurrenceRule: 'FREQ=WEEKLY;BYDAY=WE',
  // Canceled only for Feb 10 (meeting rescheduled to Thursday)
  recurrenceException: '20260210T140000Z'
}
```

## Exception handling best practices

### ✅ DO:
- Use UTC format consistently for all exceptions
- Add exceptions before the event occurs
- Validate exception dates against the recurrence rule
- Document why specific dates are excluded
- Test exception behavior in different timezones

### ❌ DON'T:
- Mix timezone formats (some UTC, some local)
- Create exceptions for dates outside the recurrence range
- Forget the Z suffix (denotes UTC)
- Use local time instead of UTC for comparisons

## Troubleshooting exceptions

**Exception not appearing:**
- ✓ Verify exception date format: `YYYYMMDDTHHMMSSZ`
- ✓ Ensure time matches parent event time (in UTC)
- ✓ Check that exception date is within recurrence rule range

**Exception date calculation:**
```tsx
// Parent event: 3:00 PM EST (UTC-5) on Jan 26
const parentStart = new Date(2026, 0, 26, 15, 0); // 3:00 PM EST

// To skip the same time on Feb 2:
// 3:00 PM EST = 8:00 PM UTC
const exceptionDate = new Date(2026, 1, 2, 20, 0); // UTC
const exception = formatExceptionDate(exceptionDate); // 20260202T200000Z
```

## Related APIs

- [EventModel.recurrenceException](https://react.syncfusion.com/react-ui/scheduler/#eventmodel)
- [EventModel.recurrenceID](https://react.syncfusion.com/react-ui/scheduler/#eventmodel)
- [EventModel.recurrenceRule](https://react.syncfusion.com/react-ui/scheduler/#eventmodel)
- [See recurrence.md](./recurrence.md) for recurrence rule syntax and patterns
- [See event-types.md](./event-types.md) for recurring event overview
