# Load-on-Demand Data Binding

Load events dynamically based on the active view's date range rather than loading the entire dataset at once. This approach significantly optimizes performance and reduces memory consumption, especially with large event datasets.

## Overview

Load-on-Demand fetches only the events that fall within the currently visible date range. When users navigate to a different date or switch views, the Scheduler automatically requests the appropriate subset of data.

## Two implementation approaches

### 1. onDataRequest event

The `onDataRequest` event fires before data is loaded and provides the start and end dates of the active view. You control the fetching logic manually:

```tsx
import { Scheduler, DayView, WeekView, MonthView, SchedulerDataRequestEvent } from "@syncfusion/react-scheduler";
import { useCallback } from "react";

async function fetchEventsForDateRange(startDate: Date, endDate: Date) {
  // Add one day to endDate for inclusive end
  const MS_PER_DAY = 86400000;
  const adjustedEndDate = new Date(endDate.getTime() + MS_PER_DAY);
  
  // Fetch from your API
  const response = await fetch(
    `/api/events?start=${startDate.toISOString()}&end=${adjustedEndDate.toISOString()}`
  );
  return response.json();
}

export default function App() {
  const onDataRequest = useCallback(async (event: SchedulerDataRequestEvent) => {
    if (event.startDate && event.endDate) {
      try {
        event.result = await fetchEventsForDateRange(event.startDate, event.endDate);
      } catch (error) {
        console.error("Failed to fetch events:", error);
        event.cancel = true;
      }
    }
  }, []);

  return (
    <Scheduler
      height="550px"
      onDataRequest={onDataRequest}
      defaultSelectedDate={new Date(2026, 5, 15)}
    >
      <DayView />
      <WeekView />
      <MonthView />
    </Scheduler>
  );
}
```

### 2. Syncfusion DataManager

Use `DataManager` with a `WebApiAdaptor` to automatically handle data fetching. The DataManager sends start/end date query parameters to your server:

```tsx
import { Scheduler, WeekView, EventSettings } from "@syncfusion/react-scheduler";
import { DataManager, WebApiAdaptor } from "@syncfusion/react-data";

const dataManager = new DataManager({
  url: "https://your-api.com/api/events",
  adaptor: new WebApiAdaptor(),
  crossDomain: true
});

const eventSettings: EventSettings = {
  dataSource: dataManager
};

export default function App() {
  return (
    <Scheduler eventSettings={eventSettings} height="550px">
      <WeekView />
    </Scheduler>
  );
}
```

**Query parameters sent by DataManager:**
```
GET /api/events?$skip=0&$take=1000&start=2026-05-15&end=2026-05-22
```

## Server-side filtering

Your backend should accept `start` and `end` query parameters and return only events that fall within that range:

**ASP.NET Core example:**
```csharp
[HttpGet("api/events")]
public IEnumerable<EventModel> GetEvents([FromQuery] DateTime start, [FromQuery] DateTime end)
{
    return events
        .Where(e => e.StartTime >= start && e.EndTime <= end)
        .ToList();
}
```

**Node.js/Express example:**
```javascript
app.get("/api/events", (req, res) => {
  const { start, end } = req.query;
  const startDate = new Date(start);
  const endDate = new Date(end);
  
  const filtered = events.filter(e => 
    e.startTime >= startDate && e.endTime <= endDate
  );
  
  res.json(filtered);
});
```

## Handling data changes

Update the Scheduler when events are created, edited, or deleted. Use the `onDataChangeStart` event to sync changes back to your server:

```tsx
import { SchedulerDataChangeEvent } from "@syncfusion/react-scheduler";

const onDataChangeStart = useCallback(async (event: SchedulerDataChangeEvent) => {
  const { addedRecords, changedRecords, deletedRecords } = event;
  
  try {
    // Send changes to your API
    if (addedRecords?.length) {
      await fetch("/api/events", {
        method: "POST",
        body: JSON.stringify(addedRecords)
      });
    }
    
    if (changedRecords?.length) {
      await fetch("/api/events", {
        method: "PATCH",
        body: JSON.stringify(changedRecords)
      });
    }
    
    if (deletedRecords?.length) {
      await fetch("/api/events", {
        method: "DELETE",
        body: JSON.stringify(deletedRecords.map(e => e.Id))
      });
    }
  } catch (error) {
    event.cancel = true; // Revert changes on error
    console.error("Failed to sync changes:", error);
  }
}, []);

return (
  <Scheduler
    onDataRequest={onDataRequest}
    onDataChangeStart={onDataChangeStart}
  >
    {/* Views */}
  </Scheduler>
);
```

## Performance considerations

### Pagination strategy

For very large datasets, combine load-on-demand with pagination:

```tsx
const onDataRequest = useCallback(async (event: SchedulerDataRequestEvent) => {
  if (event.startDate && event.endDate) {
    const MS_PER_DAY = 86400000;
    const adjustedEndDate = new Date(event.endDate.getTime() + MS_PER_DAY);
    
    // Paginate: fetch 1000 events at a time
    const response = await fetch(
      `/api/events?start=${event.startDate.toISOString()}&end=${adjustedEndDate.toISOString()}&limit=1000&offset=0`
    );
    event.result = await response.json();
  }
}, []);
```

### Caching

Implement client-side caching to avoid repeated API calls for the same date range:

```tsx
import { useRef, useCallback } from "react";

export default function App() {
  const cacheRef = useRef<Map<string, any[]>>(new Map());
  
  const getCacheKey = (start: Date, end: Date) => {
    return `${start.toISOString()}_${end.toISOString()}`;
  };
  
  const onDataRequest = useCallback(async (event: SchedulerDataRequestEvent) => {
    if (!event.startDate || !event.endDate) return;
    
    const key = getCacheKey(event.startDate, event.endDate);
    
    // Check cache first
    if (cacheRef.current.has(key)) {
      event.result = cacheRef.current.get(key);
      return;
    }
    
    // Fetch if not cached
    try {
      const MS_PER_DAY = 86400000;
      const adjustedEndDate = new Date(event.endDate.getTime() + MS_PER_DAY);
      const response = await fetch(
        `/api/events?start=${event.startDate.toISOString()}&end=${adjustedEndDate.toISOString()}`
      );
      const data = await response.json();
      
      // Cache the result
      cacheRef.current.set(key, data);
      event.result = data;
    } catch (error) {
      event.cancel = true;
    }
  }, []);
  
  return (
    <Scheduler onDataRequest={onDataRequest} height="550px">
      {/* Views */}
    </Scheduler>
  );
}
```

## Complete example with onDataRequest

```tsx
import { useCallback } from "react";
import { Scheduler, DayView, WeekView, MonthView, SchedulerDataRequestEvent, SchedulerDataChangeEvent } from "@syncfusion/react-scheduler";

const API_BASE = "https://your-api.com";

async function fetchEventsForDateRange(startDate: Date, endDate: Date) {
  const MS_PER_DAY = 86400000;
  const adjustedEndDate = new Date(endDate.getTime() + MS_PER_DAY);
  
  const response = await fetch(
    `${API_BASE}/events?start=${startDate.toISOString()}&end=${adjustedEndDate.toISOString()}`
  );
  
  if (!response.ok) {
    throw new Error(`Failed to fetch events: ${response.statusText}`);
  }
  
  return response.json();
}

async function syncChangesToServer(changes: {
  addedRecords?: any[];
  changedRecords?: any[];
  deletedRecords?: any[];
}) {
  const { addedRecords, changedRecords, deletedRecords } = changes;
  
  if (addedRecords?.length) {
    await fetch(`${API_BASE}/events`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(addedRecords)
    });
  }
  
  if (changedRecords?.length) {
    await fetch(`${API_BASE}/events`, {
      method: "PATCH",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(changedRecords)
    });
  }
  
  if (deletedRecords?.length) {
    await fetch(`${API_BASE}/events`, {
      method: "DELETE",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(deletedRecords.map(e => e.Id))
    });
  }
}

export default function App() {
  const onDataRequest = useCallback(async (event: SchedulerDataRequestEvent) => {
    if (event.startDate && event.endDate) {
      try {
        event.result = await fetchEventsForDateRange(event.startDate, event.endDate);
      } catch (error) {
        console.error("Data request failed:", error);
        event.cancel = true;
      }
    }
  }, []);
  
  const onDataChangeStart = useCallback(async (event: SchedulerDataChangeEvent) => {
    try {
      await syncChangesToServer({
        addedRecords: event.addedRecords,
        changedRecords: event.changedRecords,
        deletedRecords: event.deletedRecords
      });
    } catch (error) {
      console.error("Failed to sync changes:", error);
      event.cancel = true;
    }
  }, []);
  
  return (
    <Scheduler
      height="550px"
      width="100%"
      onDataRequest={onDataRequest}
      onDataChangeStart={onDataChangeStart}
      defaultSelectedDate={new Date(2026, 5, 15)}
      startHour="09:00"
    >
      <DayView />
      <WeekView />
      <MonthView />
    </Scheduler>
  );
}
```

## Comparing approaches

| Feature | `onDataRequest` | `DataManager` |
|---|---|---|
| **Control** | Full manual control | Automatic handling |
| **Learning curve** | Moderate | Steeper |
| **API flexibility** | Any endpoint format | Expects specific format |
| **Caching** | Manual implementation | Built-in |
| **Error handling** | Manual | Built-in retry logic |
| **Best for** | Custom backends, REST APIs | Syncfusion backends, OData |

## Related APIs

- [Scheduler.onDataRequest](https://react.syncfusion.com/react-ui/scheduler/#events-onDataRequest)
- [Scheduler.onDataChangeStart](https://react.syncfusion.com/react-ui/scheduler/#events-onDataChangeStart)
- [EventSettings.dataSource](https://react.syncfusion.com/react-ui/scheduler/#prop-eventsettings)
- [DataManager documentation](https://react.syncfusion.com/react-ui/datagrid/#data-binding)
