# External Form Editing

Manage Scheduler events using an external form instead of the built-in editor. This approach provides complete control over the form layout, validation, and user experience for creating and editing events.

## Three core methods for programmatic event management

### 1. Create events with addEvent()

Add one or more appointments to the Scheduler using the `addEvent()` method:

```tsx
import { useRef } from "react";
import { Scheduler, IScheduler } from "@syncfusion/react-scheduler";

export default function App() {
  const schedulerRef = useRef<IScheduler>(null);
  
  const handleAddEvent = () => {
    const newEvent = {
      Id: Math.max(...existingEvents.map(e => e.Id), 0) + 1,
      Subject: "New Event",
      StartTime: new Date(2026, 5, 15, 10, 0),
      EndTime: new Date(2026, 5, 15, 11, 0)
    };
    
    schedulerRef.current?.addEvent(newEvent);
  };
  
  return (
    <>
      <button onClick={handleAddEvent}>Add Event</button>
      <Scheduler ref={schedulerRef}>
        {/* Views */}
      </Scheduler>
    </>
  );
}
```

Add multiple events at once:

```tsx
const handleAddMultipleEvents = () => {
  const events = [
    {
      Id: 1,
      Subject: "Meeting 1",
      StartTime: new Date(2026, 5, 15, 9, 0),
      EndTime: new Date(2026, 5, 15, 10, 0)
    },
    {
      Id: 2,
      Subject: "Meeting 2",
      StartTime: new Date(2026, 5, 15, 11, 0),
      EndTime: new Date(2026, 5, 15, 12, 0)
    }
  ];
  
  schedulerRef.current?.addEvent(events);
};
```

### 2. Update events with saveEvent()

Modify an existing appointment by passing an updated event object with a valid `Id`:

```tsx
const handleUpdateEvent = (eventId: number, updates: Partial<EventModel>) => {
  const existingEvent = existingEvents.find(e => e.Id === eventId);
  
  if (existingEvent) {
    const updatedEvent = {
      ...existingEvent,
      ...updates
    };
    
    schedulerRef.current?.saveEvent(updatedEvent);
  }
};

// Usage
handleUpdateEvent(1, {
  Subject: "Updated Title",
  StartTime: new Date(2026, 5, 15, 14, 0),
  EndTime: new Date(2026, 5, 15, 15, 0)
});
```

### 3. Delete events with deleteEvent()

Remove an appointment by passing either its `Id` or the entire event object:

```tsx
// Delete by ID
const handleDeleteById = (eventId: number) => {
  schedulerRef.current?.deleteEvent(eventId);
};

// Delete by event object
const handleDeleteByObject = (event: EventModel) => {
  schedulerRef.current?.deleteEvent(event);
};

// Delete with recurrence handling
const handleDeleteRecurringEvent = (event: EventModel, action: 'DeleteOccurrence' | 'DeleteSeries') => {
  schedulerRef.current?.deleteEvent(event, action);
};
```

## Building an external form

Combine these methods with a form component to create a complete event management interface:

```tsx
import { useRef, useState, useMemo } from "react";
import { Scheduler, IScheduler, EventModel } from "@syncfusion/react-scheduler";
import { Form, FormField, TextBox } from "@syncfusion/react-inputs";
import { Button, Variant } from "@syncfusion/react-buttons";
import { DatePicker } from "@syncfusion/react-calendars";
import { TimePicker } from "@syncfusion/react-calendars";

interface EventFormData {
  Id?: number;
  Subject: string;
  Location?: string;
  StartTime?: Date;
  EndTime?: Date;
  IsAllDay?: boolean;
  isEditing: boolean;
}

export default function App() {
  const schedulerRef = useRef<IScheduler>(null);
  const eventsRef = useRef<EventModel[]>([
    {
      Id: 1,
      Subject: "Design Review",
      Location: "Room 1",
      StartTime: new Date(2026, 5, 15, 9, 0),
      EndTime: new Date(2026, 5, 15, 10, 0)
    }
  ]);

  const [formData, setFormData] = useState<EventFormData>({
    Subject: "",
    Location: "",
    StartTime: new Date(),
    EndTime: new Date(),
    IsAllDay: false,
    isEditing: false
  });

  // Generate next ID
  const nextId = useMemo(() => {
    const maxId = Math.max(...eventsRef.current.map(e => e.Id || 0), 0);
    let id = maxId;
    return () => ++id;
  }, []);

  // Clear form
  const clearForm = () => {
    setFormData({
      Subject: "",
      Location: "",
      StartTime: new Date(),
      EndTime: new Date(),
      IsAllDay: false,
      isEditing: false
    });
  };

  // Click on event to edit
  const handleEventClick = (event: EventModel) => {
    setFormData({
      Id: event.Id as number,
      Subject: (event.Subject as string) || "",
      Location: (event.Location as string) || "",
      StartTime: new Date(event.StartTime as Date),
      EndTime: new Date(event.EndTime as Date),
      IsAllDay: (event.IsAllDay as boolean) || false,
      isEditing: true
    });
  };

  // Click on cell to create new event
  const handleCellClick = (startTime: Date, endTime: Date) => {
    setFormData({
      Subject: "",
      Location: "",
      StartTime,
      EndTime,
      IsAllDay: false,
      isEditing: false
    });
  };

  // Save event (create or update)
  const handleSave = () => {
    const eventData: EventModel = {
      Id: formData.Id ?? nextId(),
      Subject: formData.Subject || "Untitled",
      Location: formData.Location,
      StartTime: formData.StartTime,
      EndTime: formData.EndTime,
      IsAllDay: formData.IsAllDay
    };

    if (formData.isEditing && formData.Id) {
      schedulerRef.current?.saveEvent(eventData);
    } else {
      schedulerRef.current?.addEvent(eventData);
    }

    clearForm();
  };

  // Delete event
  const handleDelete = () => {
    if (formData.isEditing && formData.Id) {
      schedulerRef.current?.deleteEvent(formData.Id);
      clearForm();
    }
  };

  return (
    <div style={{ display: "grid", gridTemplateColumns: "350px 1fr", gap: "20px", padding: "20px" }}>
      {/* External Form */}
      <div style={{ border: "1px solid #e0e0e0", padding: "20px", borderRadius: "4px" }}>
        <h3>Event Form</h3>
        <Form>
          <FormField name="Subject">
            <div style={{ marginBottom: "15px" }}>
              <label htmlFor="subject" style={{ display: "block", marginBottom: "5px" }}>
                Title *
              </label>
              <TextBox
                id="subject"
                placeholder="Event title"
                value={formData.Subject}
                onChange={(e) => setFormData(prev => ({ ...prev, Subject: e.value || "" }))}
              />
            </div>
          </FormField>

          <FormField name="Location">
            <div style={{ marginBottom: "15px" }}>
              <label htmlFor="location" style={{ display: "block", marginBottom: "5px" }}>
                Location
              </label>
              <TextBox
                id="location"
                placeholder="Event location"
                value={formData.Location}
                onChange={(e) => setFormData(prev => ({ ...prev, Location: e.value || "" }))}
              />
            </div>
          </FormField>

          <FormField name="StartTime">
            <div style={{ marginBottom: "15px" }}>
              <label style={{ display: "block", marginBottom: "5px" }}>Start Date & Time</label>
              <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: "10px" }}>
                <DatePicker
                  value={formData.StartTime}
                  onChange={(e) => {
                    const date = e.value as Date;
                    const startTime = formData.StartTime as Date;
                    startTime.setFullYear(date.getFullYear(), date.getMonth(), date.getDate());
                    setFormData(prev => ({ ...prev, StartTime: new Date(startTime) }));
                  }}
                />
                <TimePicker
                  value={formData.StartTime}
                  onChange={(e) => {
                    const time = e.value as Date;
                    const startTime = formData.StartTime as Date;
                    startTime.setHours(time.getHours(), time.getMinutes(), 0, 0);
                    setFormData(prev => ({ ...prev, StartTime: new Date(startTime) }));
                  }}
                />
              </div>
            </div>
          </FormField>

          <FormField name="EndTime">
            <div style={{ marginBottom: "15px" }}>
              <label style={{ display: "block", marginBottom: "5px" }}>End Date & Time</label>
              <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: "10px" }}>
                <DatePicker
                  value={formData.EndTime}
                  onChange={(e) => {
                    const date = e.value as Date;
                    const endTime = formData.EndTime as Date;
                    endTime.setFullYear(date.getFullYear(), date.getMonth(), date.getDate());
                    setFormData(prev => ({ ...prev, EndTime: new Date(endTime) }));
                  }}
                />
                <TimePicker
                  value={formData.EndTime}
                  onChange={(e) => {
                    const time = e.value as Date;
                    const endTime = formData.EndTime as Date;
                    endTime.setHours(time.getHours(), time.getMinutes(), 0, 0);
                    setFormData(prev => ({ ...prev, EndTime: new Date(endTime) }));
                  }}
                />
              </div>
            </div>
          </FormField>

          <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: "10px" }}>
            <Button
              variant={Variant.Filled}
              onClick={handleSave}
            >
              {formData.isEditing ? "Update" : "Create"}
            </Button>
            <Button
              variant={Variant.Outlined}
              onClick={clearForm}
            >
              Cancel
            </Button>
          </div>

          {formData.isEditing && (
            <Button
              variant={Variant.Text}
              onClick={handleDelete}
              style={{ marginTop: "10px", color: "#d32f2f", width: "100%" }}
            >
              Delete Event
            </Button>
          )}
        </Form>
      </div>

      {/* Scheduler */}
      <Scheduler
        ref={schedulerRef}
        height="550px"
        eventSettings={{ dataSource: eventsRef.current }}
        onEventClick={(args) => handleEventClick(args.data as EventModel)}
        onCellClick={(args) => handleCellClick(args.startTime as Date, args.endTime as Date)}
      >
        {/* Views */}
      </Scheduler>
    </div>
  );
}
```

## Handling all-day and spanned events

Support all-day events and multi-day spans in your external form:

```tsx
interface ExtendedEventData extends EventModel {
  IsAllDay?: boolean;
  IsSpanned?: boolean;
}

const handleSave = () => {
  const eventData: ExtendedEventData = {
    Id: formData.Id ?? nextId(),
    Subject: formData.Subject || "Untitled",
    Location: formData.Location,
    StartTime: formData.StartTime,
    EndTime: formData.EndTime,
    IsAllDay: formData.IsAllDay,
    IsSpanned: formData.IsSpanned || false
  };

  if (formData.isEditing && formData.Id) {
    schedulerRef.current?.saveEvent(eventData);
  } else {
    schedulerRef.current?.addEvent(eventData);
  }

  clearForm();
};
```

## Validation

Add form validation before saving:

```tsx
const validateForm = (): string[] => {
  const errors: string[] = [];

  if (!formData.Subject?.trim()) {
    errors.push("Title is required");
  }

  if (!formData.StartTime) {
    errors.push("Start date is required");
  }

  if (!formData.EndTime) {
    errors.push("End date is required");
  }

  if (formData.StartTime && formData.EndTime && formData.StartTime >= formData.EndTime) {
    errors.push("End time must be after start time");
  }

  return errors;
};

const handleSave = () => {
  const errors = validateForm();

  if (errors.length > 0) {
    alert(errors.join("\n"));
    return;
  }

  // Proceed with save...
};
```

## Related APIs

- [Scheduler.addEvent()](https://react.syncfusion.com/react-ui/scheduler/#methods-addEvent)
- [Scheduler.saveEvent()](https://react.syncfusion.com/react-ui/scheduler/#methods-saveEvent)
- [Scheduler.deleteEvent()](https://react.syncfusion.com/react-ui/scheduler/#methods-deleteEvent)
- [Scheduler.onEventClick()](https://react.syncfusion.com/react-ui/scheduler/#events-onEventClick)
- [Scheduler.onCellClick()](https://react.syncfusion.com/react-ui/scheduler/#events-onCellClick)
- [Event model interface](./events-data.md#event-fields)
