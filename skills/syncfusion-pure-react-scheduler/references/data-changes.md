# Handling Data Changes and Synchronization

When users add, edit, or delete events, the Scheduler triggers `onDataChangeStart` and `onDataChangeComplete` callbacks. These events enable persistence to a backend, state management, UI feedback, and error handling.

## Data change event lifecycle

```
User Action (Add/Edit/Delete)
         ↓
   onDataChangeStart (event triggered)
         ↓
   [Can cancel here: event.cancel = true]
         ↓
   Data modified in Scheduler
         ↓
   onDataChangeComplete (success callback)
         ↓
   Update UI state / Show confirmation
```

## onDataChangeStart callback

Triggered immediately when a data modification begins. Use this to:
- Show a loading indicator
- Validate data before committing
- Cancel the operation if needed
- Track change type (add/edit/delete)

```tsx
import { Scheduler, WeekView, SchedulerDataChangeEvent, EventSettings } from '@syncfusion/react-scheduler';
import { useState } from 'react';

export default function App() {
  const [isLoading, setIsLoading] = useState(false);
  const [events, setEvents] = useState(defaultData);

  const handleDataChangeStart = (args: SchedulerDataChangeEvent) => {
    // Show loading state
    setIsLoading(true);
    console.log('Data change started');

    // Identify change type
    if (args.addedRecords && args.addedRecords.length > 0) {
      console.log('Adding event:', args.addedRecords[0]);
    }
    if (args.changedRecords && args.changedRecords.length > 0) {
      console.log('Editing event:', args.changedRecords[0]);
    }
    if (args.deletedRecords && args.deletedRecords.length > 0) {
      console.log('Deleting event:', args.deletedRecords[0]);
    }
  };

  const eventSettings: EventSettings = { dataSource: events };

  return (
    <Scheduler
      eventSettings={eventSettings}
      onDataChangeStart={handleDataChangeStart}
      defaultSelectedDate={new Date(2026, 0, 15)}
    >
      <WeekView />
    </Scheduler>
  );
}
```

## onDataChangeComplete callback

Triggered after data modification succeeds. Use this to:
- Hide loading indicators
- Display success/error notifications
- Update parent state
- Sync with backend

```tsx
import { Scheduler, WeekView, SchedulerDataChangeEvent, EventSettings } from '@syncfusion/react-scheduler';
import { Message, Severity } from '@syncfusion/react-notifications';
import { useState } from 'react';

export default function App() {
  const [events, setEvents] = useState(defaultData);
  const [message, setMessage] = useState('');
  const [severity, setSeverity] = useState<Severity>(Severity.Normal);

  const handleDataChangeComplete = (args: SchedulerDataChangeEvent) => {
    // Determine action type and show message
    if (args.addedRecords && args.addedRecords.length > 0) {
      setMessage(`✓ Created ${args.addedRecords.length} event(s)`);
      setSeverity(Severity.Success);
    } else if (args.changedRecords && args.changedRecords.length > 0) {
      setMessage(`✓ Updated ${args.changedRecords.length} event(s)`);
      setSeverity(Severity.Success);
    } else if (args.deletedRecords && args.deletedRecords.length > 0) {
      setMessage(`✓ Deleted ${args.deletedRecords.length} event(s)`);
      setSeverity(Severity.Success);
    }

    // Clear message after 3 seconds
    setTimeout(() => setMessage(''), 3000);
  };

  const eventSettings: EventSettings = { dataSource: events };

  return (
    <>
      {message && <Message severity={severity}>{message}</Message>}
      <Scheduler
        eventSettings={eventSettings}
        onDataChangeComplete={handleDataChangeComplete}
        defaultSelectedDate={new Date(2026, 0, 15)}
      >
        <WeekView />
      </Scheduler>
    </>
  );
}
```

## Updating local state on changes

Sync Scheduler changes to React state for persistence:

```tsx
import { Scheduler, WeekView, SchedulerDataChangeEvent, EventSettings } from '@syncfusion/react-scheduler';
import { useState } from 'react';

export default function App() {
  const [events, setEvents] = useState(defaultData);

  const handleDataChangeStart = (args: SchedulerDataChangeEvent) => {
    // Update state to reflect changes immediately
    if (args.addedRecords) {
      setEvents(prev => [...prev, ...args.addedRecords]);
    }
    if (args.changedRecords) {
      setEvents(prev =>
        prev.map(event =>
          args.changedRecords?.some(changed => changed.id === event.id)
            ? args.changedRecords.find(changed => changed.id === event.id)!
            : event
        )
      );
    }
    if (args.deletedRecords) {
      setEvents(prev =>
        prev.filter(event =>
          !args.deletedRecords?.some(deleted => deleted.id === event.id)
        )
      );
    }
  };

  const eventSettings: EventSettings = { dataSource: events };

  return (
    <Scheduler
      eventSettings={eventSettings}
      onDataChangeStart={handleDataChangeStart}
      defaultSelectedDate={new Date(2026, 0, 15)}
    >
      <WeekView />
    </Scheduler>
  );
}
```

## Persisting changes to a backend

Send add/edit/delete operations to a backend API:

```tsx
import { Scheduler, WeekView, SchedulerDataChangeEvent, EventSettings } from '@syncfusion/react-scheduler';
import { useState } from 'react';

export default function App() {
  const [events, setEvents] = useState(defaultData);
  const [isSaving, setIsSaving] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const persistToBackend = async (args: SchedulerDataChangeEvent) => {
    try {
      setIsSaving(true);

      // Create (POST)
      if (args.addedRecords && args.addedRecords.length > 0) {
        const response = await fetch('/api/events', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(args.addedRecords)
        });
        if (!response.ok) throw new Error('Failed to create events');
      }

      // Update (PUT)
      if (args.changedRecords && args.changedRecords.length > 0) {
        await Promise.all(
          args.changedRecords.map(record =>
            fetch(`/api/events/${record.id}`, {
              method: 'PUT',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify(record)
            })
          )
        );
      }

      // Delete (DELETE)
      if (args.deletedRecords && args.deletedRecords.length > 0) {
        await Promise.all(
          args.deletedRecords.map(record =>
            fetch(`/api/events/${record.id}`, { method: 'DELETE' })
          )
        );
      }

      setError(null);
    } catch (err) {
      setError((err as Error).message);
      // Cancel the change to prevent inconsistency
      args.cancel = true;
    } finally {
      setIsSaving(false);
    }
  };

  const eventSettings: EventSettings = { dataSource: events };

  return (
    <Scheduler
      eventSettings={eventSettings}
      onDataChangeStart={persistToBackend}
      defaultSelectedDate={new Date(2026, 0, 15)}
    >
      <WeekView />
    </Scheduler>
  );
}
```

## Canceling data changes

Prevent a change from being applied by setting `event.cancel = true`:

```tsx
import { Scheduler, WeekView, SchedulerDataChangeEvent, EventSettings } from '@syncfusion/react-scheduler';

export default function App() {
  const handleDataChangeStart = (args: SchedulerDataChangeEvent) => {
    // Cancel deletion if event is marked as protected
    if (args.deletedRecords) {
      const protectedEvent = args.deletedRecords.find((e: any) => e.protected === true);
      if (protectedEvent) {
        args.cancel = true;
        alert('This event cannot be deleted');
      }
    }

    // Cancel if scheduling conflict detected
    if (args.changedRecords) {
      const hasConflict = args.changedRecords.some(newEvent => {
        return args.changedRecords!.some(other =>
          other.id !== newEvent.id &&
          newEvent.startTime < other.endTime &&
          newEvent.endTime > other.startTime
        );
      });
      if (hasConflict) {
        args.cancel = true;
        alert('This time slot is already booked');
      }
    }
  };

  const eventSettings: EventSettings = { dataSource: defaultData };

  return (
    <Scheduler
      eventSettings={eventSettings}
      onDataChangeStart={handleDataChangeStart}
      defaultSelectedDate={new Date(2026, 0, 15)}
    >
      <WeekView />
    </Scheduler>
  );
}
```

## Handling errors and rollback

Implement error handling with user-friendly feedback:

```tsx
import { Scheduler, WeekView, SchedulerDataChangeEvent, EventSettings } from '@syncfusion/react-scheduler';
import { Toast, ToastPosition, ToastOpen } from '@syncfusion/react-popups';
import { useRef, useState } from 'react';

export default function App() {
  const [events, setEvents] = useState(defaultData);
  const toastRef = useRef<Toast>(null);

  const handleDataChangeStart = async (args: SchedulerDataChangeEvent) => {
    try {
      // Attempt persistence
      if (args.addedRecords) {
        const res = await fetch('/api/events', {
          method: 'POST',
          body: JSON.stringify(args.addedRecords)
        });

        if (res.status === 409) {
          // Conflict: event already exists
          args.cancel = true;
          showToast('Event already exists', 'warning');
          return;
        }

        if (!res.ok) {
          throw new Error(`Server error: ${res.statusText}`);
        }
      }

      showToast('Changes saved successfully', 'success');
    } catch (err) {
      // Roll back: cancel the operation
      args.cancel = true;
      showToast(`Error: ${(err as Error).message}`, 'error');
    }
  };

  const showToast = (message: string, type: 'success' | 'error' | 'warning') => {
    const toast = toastRef.current as any;
    toast.show({
      title: type.toUpperCase(),
      content: message,
      position: ToastPosition.TopRight
    });
  };

  const eventSettings: EventSettings = { dataSource: events };

  return (
    <>
      <Toast ref={toastRef} />
      <Scheduler
        eventSettings={eventSettings}
        onDataChangeStart={handleDataChangeStart}
        defaultSelectedDate={new Date(2026, 0, 15)}
      >
        <WeekView />
      </Scheduler>
    </>
  );
}
```

## Change notification and audit logging

Track all changes for compliance and debugging:

```tsx
import { SchedulerDataChangeEvent } from '@syncfusion/react-scheduler';

interface AuditLog {
  timestamp: Date;
  action: 'CREATE' | 'UPDATE' | 'DELETE';
  eventId: string | number;
  changes: Record<string, any>;
  userId: string;
}

const auditLogs: AuditLog[] = [];

const handleDataChangeStart = (args: SchedulerDataChangeEvent) => {
  const userId = getCurrentUser().id; // Your auth context

  if (args.addedRecords) {
    args.addedRecords.forEach(record => {
      auditLogs.push({
        timestamp: new Date(),
        action: 'CREATE',
        eventId: record.id,
        changes: record,
        userId
      });
    });
  }

  if (args.changedRecords) {
    args.changedRecords.forEach(record => {
      auditLogs.push({
        timestamp: new Date(),
        action: 'UPDATE',
        eventId: record.id,
        changes: record,
        userId
      });
    });
  }

  if (args.deletedRecords) {
    args.deletedRecords.forEach(record => {
      auditLogs.push({
        timestamp: new Date(),
        action: 'DELETE',
        eventId: record.id,
        changes: record,
        userId
      });
    });
  }
};
```

## SchedulerDataChangeEvent properties

| Property | Type | Purpose |
|----------|------|---------|
| `addedRecords` | EventModel[] | Newly created events |
| `changedRecords` | EventModel[] | Modified events |
| `deletedRecords` | EventModel[] | Deleted events |
| `cancel` | boolean | Set `true` to prevent the change |

## Common data change patterns

### Save after every change
```tsx
onDataChangeStart={(args) => persistToBackend(args)}
```

### Batch save after delay
```tsx
const batchSave = debounce((args) => persistToBackend(args), 2000);
onDataChangeStart={batchSave}
```

### Notify team members
```tsx
if (args.changedRecords) {
  notifyTeam(`Event updated by ${currentUser.name}`);
}
```

### Validate before save
```tsx
if (args.addedRecords) {
  const isValid = validateEvents(args.addedRecords);
  if (!isValid) args.cancel = true;
}
```

## Related APIs

- [Scheduler.onDataChangeStart](https://react.syncfusion.com/react-ui/scheduler/#ondatachangestart)
- [Scheduler.onDataChangeComplete](https://react.syncfusion.com/react-ui/scheduler/#ondatachangecomplete)
- [SchedulerDataChangeEvent](https://react.syncfusion.com/react-ui/scheduler/#schedulerdatachangeevent)
- [See events-data.md](./events-data.md) for initial data binding
- [See external-forms.md](./external-forms.md) for programmatic event creation
- [See load-on-demand.md](./load-on-demand.md) for remote data fetching patterns
