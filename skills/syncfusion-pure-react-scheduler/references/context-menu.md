# Context Menu Integration

Context menus enable users to perform quick actions on cells (creating events) and events (editing, deleting) through right-click (desktop) or long-press (touch) interactions. The Scheduler integrates seamlessly with Syncfusion's `ContextMenu` component to provide a natural, desktop-like workflow.

## Overview

The context menu is externally rendered and attached to the Scheduler container. When the user right-clicks or long-presses:

1. **On a cell** - Show cell actions (create normal/recurring events, navigate to today)
2. **On an event** - Show event actions (edit, delete)
3. **On a recurring event** - Show recurrence-specific options (edit occurrence/series, delete occurrence/series)

## Setting up context menu

Import the `ContextMenu` component and link it to the Scheduler:

```tsx
import { ContextMenu, MenuItem, MenuItemIcon, MenuItemLabel } from "@syncfusion/react-navigations";
import { Scheduler, IScheduler, EventModel, SchedulerCellDetails } from "@syncfusion/react-scheduler";
import { useRef, useState } from "react";

export default function App() {
  const schedulerRef = useRef<IScheduler>(null);
  const targetRef = useRef<HTMLElement>(null);
  const [open, setOpen] = useState(false);
  const [menu, setMenu] = useState<MenuItemInterface[]>([]);

  const setContainerRef = (el: HTMLDivElement | null) => {
    targetRef.current = el as HTMLElement | null;
  };

  return (
    <div ref={setContainerRef}>
      <Scheduler ref={schedulerRef}>
        {/* Views */}
      </Scheduler>
      <ContextMenu
        open={open}
        targetRef={targetRef}
        onOpen={handleContextMenuOpen}
        onClose={() => setOpen(false)}
        onSelect={handleContextMenuSelect}
      >
        {/* Menu items rendered here */}
      </ContextMenu>
    </div>
  );
}
```

## Detecting click context (cell vs event)

In the `onOpen` handler, determine whether the user clicked on a cell or event:

```tsx
interface MenuItemInterface {
  text: string;
  id: string;
  icon?: ReactNode;
  items?: MenuItemInterface[];
}

const cellMenuItems: MenuItemInterface[] = [
  { text: 'New Event', id: 'Add' },
  { text: 'New Recurring Event', id: 'AddRecurrence' },
  { text: 'Today', id: 'Today' }
];

const eventMenuItems: MenuItemInterface[] = [
  { text: 'Edit Event', id: 'Edit' },
  { text: 'Delete Event', id: 'Delete' }
];

const recurrenceEventMenuItems: MenuItemInterface[] = [
  {
    text: 'Edit Event',
    id: 'EditRecurrenceEvent',
    items: [
      { text: 'Edit Occurrence', id: 'EditOccurrence' },
      { text: 'Edit Series', id: 'EditSeries' }
    ]
  },
  {
    text: 'Delete Event',
    id: 'DeleteRecurrenceEvent',
    items: [
      { text: 'Delete Occurrence', id: 'DeleteOccurrence' },
      { text: 'Delete Series', id: 'DeleteSeries' }
    ]
  }
];

const onContextMenuBeforeOpen = (args: Event) => {
  const target = (args?.target as HTMLElement) ?? null;
  if (!schedulerRef.current || !target) return;

  // Check if clicked on an event
  const selectedEvent = target.closest('.sf-appointment');
  if (selectedEvent) {
    const eventDetails = schedulerRef.current.getEventDetails(selectedEvent);
    if (eventDetails?.recurrenceRule || eventDetails?.recurrenceID) {
      setMenu(recurrenceEventMenuItems);
    } else {
      setMenu(eventMenuItems);
    }
    setOpen(true);
    return;
  }

  // Check if clicked on a cell
  const isMonthView = !!target.closest('.sf-month-view');
  const cellSelector = isMonthView 
    ? '.sf-work-cells, .sf-all-day-cell' 
    : '.sf-work-cells, .sf-all-day-cell, .sf-header-cells';
  const selectedCell = target.closest(cellSelector);
  if (selectedCell) {
    setMenu(cellMenuItems);
    setOpen(true);
  }
};
```

## Handling menu actions

Process menu selections using the `onSelect` handler:

```tsx
let selectedTarget = useRef<Element>(null);

const onContextMenuBeforeOpen = (args: Event) => {
  const target = (args?.target as HTMLElement) ?? null;
  // ... detection logic ...
  if (selectedEvent) {
    selectedTarget.current = selectedEvent;
  }
  if (selectedCell) {
    selectedTarget.current = selectedCell;
  }
};

const onContextMenuSelect = (args: MenuSelectEvent) => {
  const type = args?.item?.id;
  if (!schedulerRef.current || !type) return;

  let selectedEvent: EventModel | null = null;
  if (selectedTarget.current) {
    const details = schedulerRef.current.getEventDetails(selectedTarget.current);
    selectedEvent = details;
  }

  switch (type) {
    case 'Today': {
      // Navigate to today
      schedulerRef.current.scrollToDate(new Date());
      break;
    }
    case 'Add': {
      // Create new normal event
      const cellDetails = schedulerRef.current.getCellDetails(selectedTarget.current);
      schedulerRef.current.openEditor('Add', cellDetails);
      break;
    }
    case 'AddRecurrence': {
      // Create new recurring event
      const cellDetails = schedulerRef.current.getCellDetails(selectedTarget.current);
      const eventData: EventModel = {
        startTime: cellDetails?.startTime,
        endTime: cellDetails?.endTime,
        isAllDay: cellDetails?.isAllDay,
        recurrenceRule: 'FREQ=DAILY;INTERVAL=1;'
      };
      schedulerRef.current.openEditor('Add', eventData);
      break;
    }
    case 'Edit': {
      // Edit normal event
      if (selectedEvent) {
        schedulerRef.current.openEditor('Edit', selectedEvent);
      }
      break;
    }
    case 'EditOccurrence': {
      // Edit single occurrence of recurring event
      if (selectedEvent) {
        schedulerRef.current.openEditor('EditOccurrence', selectedEvent);
      }
      break;
    }
    case 'EditSeries': {
      // Edit entire recurring series
      if (selectedEvent) {
        schedulerRef.current.openEditor('EditSeries', selectedEvent);
      }
      break;
    }
    case 'Delete': {
      // Delete normal event
      if (selectedEvent) {
        schedulerRef.current.deleteEvent(selectedEvent);
      }
      break;
    }
    case 'DeleteOccurrence': {
      // Delete single occurrence of recurring event
      if (selectedEvent) {
        schedulerRef.current.deleteEvent(selectedEvent, 'DeleteOccurrence');
      }
      break;
    }
    case 'DeleteSeries': {
      // Delete entire recurring series
      if (selectedEvent) {
        schedulerRef.current.deleteEvent(selectedEvent, 'DeleteSeries');
      }
      break;
    }
  }
};
```

## Rendering menu items

Use a helper function to recursively render menu items and submenus:

```tsx
import { AddNotesIcon, DeleteNotesIcon, EditIcon, RepeatIcon, DayIcon } from "@syncfusion/react-icons";

const cellMenuItems: MenuItemInterface[] = [
  { text: 'New Event', id: 'Add', icon: <AddNotesIcon /> },
  { text: 'New Recurring Event', id: 'AddRecurrence', icon: <RepeatIcon /> },
  { text: 'Today', id: 'Today', icon: <DayIcon /> }
];

const eventMenuItems: MenuItemInterface[] = [
  { text: 'Edit Event', id: 'Edit', icon: <EditIcon /> },
  { text: 'Delete Event', id: 'Delete', icon: <DeleteNotesIcon /> }
];

const renderMenuItems = (items: MenuItemInterface[]) => {
  return items.map((item, index) =>
    item.items ? (
      <MenuItem key={index} id={item.id}>
        {item.icon && <MenuItemIcon>{item.icon}</MenuItemIcon>}
        <MenuItemLabel>{item.text}</MenuItemLabel>
        {renderMenuItems(item.items)}
      </MenuItem>
    ) : (
      <MenuItem key={index} id={item.id}>
        {item.icon && <MenuItemIcon>{item.icon}</MenuItemIcon>}
        <MenuItemLabel>{item.text}</MenuItemLabel>
      </MenuItem>
    )
  );
};

return (
  <ContextMenu
    open={open}
    targetRef={targetRef}
    onOpen={onContextMenuBeforeOpen}
    onClose={() => setOpen(false)}
    onSelect={onContextMenuSelect}
  >
    {renderMenuItems(menu)}
  </ContextMenu>
);
```

## Complete example

```tsx
import { useRef, useState, ReactNode } from "react";
import { 
  ContextMenu, MenuSelectEvent, MenuItem, MenuItemIcon, MenuItemLabel 
} from "@syncfusion/react-navigations";
import { 
  DayView, WeekView, MonthView, Scheduler, EventSettings, 
  EventModel, IScheduler, SchedulerDateChangeEvent, SchedulerCellDetails
} from "@syncfusion/react-scheduler";
import { AddNotesIcon, DeleteNotesIcon, EditIcon, RepeatIcon, DayIcon } from "@syncfusion/react-icons";
import { defaultData } from './dataSource';

interface MenuItemInterface {
  text: string;
  id: string;
  icon?: ReactNode;
  items?: MenuItemInterface[];
}

export default function App() {
  const schedulerRef = useRef<IScheduler>(null);
  const targetRef = useRef<HTMLElement>(null);
  const selectedTargetRef = useRef<Element>(null);
  const [open, setOpen] = useState(false);
  const [menu, setMenu] = useState<MenuItemInterface[]>([]);
  const [selectedDate, setSelectedDate] = useState(new Date(2025, 0, 12));

  const eventSettings: EventSettings = { dataSource: defaultData };

  const cellMenuItems: MenuItemInterface[] = [
    { text: 'New Event', id: 'Add', icon: <AddNotesIcon /> },
    { text: 'New Recurring Event', id: 'AddRecurrence', icon: <RepeatIcon /> },
    { text: 'Today', id: 'Today', icon: <DayIcon /> }
  ];

  const eventMenuItems: MenuItemInterface[] = [
    { text: 'Edit Event', id: 'Edit', icon: <EditIcon /> },
    { text: 'Delete Event', id: 'Delete', icon: <DeleteNotesIcon /> }
  ];

  const recurrenceEventMenuItems: MenuItemInterface[] = [
    {
      text: 'Edit Event',
      id: 'EditRecurrenceEvent',
      icon: <EditIcon />,
      items: [
        { text: 'Edit Occurrence', id: 'EditOccurrence' },
        { text: 'Edit Series', id: 'EditSeries' }
      ]
    },
    {
      text: 'Delete Event',
      id: 'DeleteRecurrenceEvent',
      icon: <DeleteNotesIcon />,
      items: [
        { text: 'Delete Occurrence', id: 'DeleteOccurrence' },
        { text: 'Delete Series', id: 'DeleteSeries' }
      ]
    }
  ];

  const onContextMenuBeforeOpen = (args: Event) => {
    const target = (args?.target as HTMLElement) ?? null;
    if (!schedulerRef.current || !target) return;

    const selectedEvent = target.closest('.sf-appointment');
    if (selectedEvent) {
      selectedTargetRef.current = selectedEvent;
      const eventDetails = schedulerRef.current.getEventDetails(selectedEvent);
      setMenu(
        eventDetails?.recurrenceRule || eventDetails?.recurrenceID
          ? recurrenceEventMenuItems
          : eventMenuItems
      );
      setOpen(true);
      return;
    }

    const isMonthView = !!target.closest('.sf-month-view');
    const cellSelector = isMonthView 
      ? '.sf-work-cells, .sf-all-day-cell' 
      : '.sf-work-cells, .sf-all-day-cell, .sf-header-cells';
    const selectedCell = target.closest(cellSelector);
    if (selectedCell) {
      selectedTargetRef.current = selectedCell;
      setMenu(cellMenuItems);
      setOpen(true);
    }
  };

  const onContextMenuSelect = (args: MenuSelectEvent) => {
    const type = args?.item?.id;
    if (!schedulerRef.current || !type) return;

    let selectedEvent: EventModel | null = null;
    if (selectedTargetRef.current) {
      const details = schedulerRef.current.getEventDetails(selectedTargetRef.current);
      selectedEvent = details;
    }

    switch (type) {
      case 'Today':
        setSelectedDate(new Date());
        break;
      case 'Add':
      case 'AddRecurrence': {
        const cellDetails = schedulerRef.current.getCellDetails(selectedTargetRef.current);
        if (type === 'Add') {
          schedulerRef.current.openEditor('Add', cellDetails);
        } else {
          const eventData: EventModel = {
            startTime: cellDetails?.startTime,
            endTime: cellDetails?.endTime,
            isAllDay: cellDetails?.isAllDay,
            recurrenceRule: 'FREQ=DAILY;INTERVAL=1;'
          };
          schedulerRef.current.openEditor('Add', eventData);
        }
        break;
      }
      case 'Edit':
        if (selectedEvent) schedulerRef.current.openEditor('Edit', selectedEvent);
        break;
      case 'EditOccurrence':
        if (selectedEvent) schedulerRef.current.openEditor('EditOccurrence', selectedEvent);
        break;
      case 'EditSeries':
        if (selectedEvent) schedulerRef.current.openEditor('EditSeries', selectedEvent);
        break;
      case 'Delete':
        if (selectedEvent) schedulerRef.current.deleteEvent(selectedEvent);
        break;
      case 'DeleteOccurrence':
        if (selectedEvent) schedulerRef.current.deleteEvent(selectedEvent, 'DeleteOccurrence');
        break;
      case 'DeleteSeries':
        if (selectedEvent) schedulerRef.current.deleteEvent(selectedEvent, 'DeleteSeries');
        break;
    }
  };

  const renderMenuItems = (items: MenuItemInterface[]) => {
    return items.map((item, index) =>
      item.items ? (
        <MenuItem key={index} id={item.id}>
          {item.icon && <MenuItemIcon>{item.icon}</MenuItemIcon>}
          <MenuItemLabel>{item.text}</MenuItemLabel>
          {renderMenuItems(item.items)}
        </MenuItem>
      ) : (
        <MenuItem key={index} id={item.id}>
          {item.icon && <MenuItemIcon>{item.icon}</MenuItemIcon>}
          <MenuItemLabel>{item.text}</MenuItemLabel>
        </MenuItem>
      )
    );
  };

  return (
    <div ref={(el) => (targetRef.current = el as HTMLElement | null)}>
      <Scheduler
        ref={schedulerRef}
        height="550px"
        width="100%"
        eventSettings={eventSettings}
        selectedDate={selectedDate}
        onSelectedDateChange={(e: SchedulerDateChangeEvent) => setSelectedDate(e.value)}
      >
        <DayView />
        <WeekView />
        <MonthView />
      </Scheduler>
      <ContextMenu
        open={open}
        targetRef={targetRef}
        onOpen={onContextMenuBeforeOpen}
        onClose={() => setOpen(false)}
        onSelect={onContextMenuSelect}
      >
        {renderMenuItems(menu)}
      </ContextMenu>
    </div>
  );
}
```

## Styling context menu

Apply theme-aware styles to the context menu:

```tsx
<ContextMenu
  open={open}
  targetRef={targetRef}
  style={{
    border: '1px solid #e0e0e0',
    borderRadius: '4px',
    boxShadow: '0 2px 8px rgba(0, 0, 0, 0.15)'
  }}
  onOpen={onContextMenuBeforeOpen}
  onClose={() => setOpen(false)}
  onSelect={onContextMenuSelect}
>
  {renderMenuItems(menu)}
</ContextMenu>
```

## Related APIs

- [ContextMenu documentation](https://react.syncfusion.com/react-ui/context-menu/#api-contextmenu)
- [Scheduler.getEventDetails()](https://react.syncfusion.com/react-ui/scheduler/#methods-getEventDetails)
- [Scheduler.getCellDetails()](https://react.syncfusion.com/react-ui/scheduler/#methods-getCellDetails)
- [Scheduler.openEditor()](https://react.syncfusion.com/react-ui/scheduler/#methods-openEditor)
- [Scheduler.deleteEvent()](https://react.syncfusion.com/react-ui/scheduler/#methods-deleteEvent)
