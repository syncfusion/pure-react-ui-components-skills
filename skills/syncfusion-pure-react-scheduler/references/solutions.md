# Solved Use Case: Tech Event Management

A real-world scheduling problem combining many scheduler features at once, demonstrating how they compose: resource-based scheduling, drag-and-drop, custom validation, custom templates, the quick popup, and a custom header — all in one application.

## Contents

- [The problem](#the-problem)
- [Feature map](#feature-map)
- [Complete implementation](#complete-implementation)
- [Why this composes well](#why-this-composes-well)

## The problem

Planning technical events involves coordinating multiple sessions across different rooms while utilizing resources efficiently. Organizers need to:

- Maintain a centralized view of sessions across multiple rooms (resource-based scheduling).
- Keep unscheduled sessions in a separate list and assign them to available time slots as plans evolve.
- Filter by room and by event track to focus on what matters.
- Enforce custom business rules: **validate room capacity** and **prevent conflicting room allocations**, blocking invalid changes during scheduling.

## Feature map

| Requirement in this use case | Scheduler feature used |
|---|---|
| Sessions organized per room | `resources` + `group.resources` (resource grouping, see [resources-grouping.md](resources-grouping.md)) |
| Unscheduled sessions dragged onto the grid | Syncfusion `DragDrop`/`Draggable`/`Droppable` (from `@syncfusion/react-base`) + scheduler `getCellDetails` |
| Business-rule validation (capacity, conflicts) | `onDataChangeStart` with `args.cancel = true` (see [events-api.md](events-api.md), [events-data.md](events-data.md)) |
| Custom session cards | `eventTemplate` on the view (see [templates-ui.md](templates-ui.md)) |
| Room headers showing capacity | `resourceHeader` template (see [templates-ui.md](templates-ui.md)) |
| Room filter in the toolbar | `header`/`SchedulerHeader` with `ToolbarItem` (see [templates-ui.md](templates-ui.md)) |
| Read-only viewing of break events | `args.cancel = true` in `onEventClick` / suppressing popups (see [crud-external-forms.md](crud-external-forms.md)) |
| Custom event detail popup | `quickInfo` renderers (see [editor-and-popups.md](editor-and-popups.md)) |
| Business hours highlight | `workHours` (see [views.md](views.md)) |
| Fine-grained slots | `timeScale` (see [views.md](views.md)) |

## Complete implementation

```tsx
import { useRef, useState } from 'react';
import {
    DayView, EventModel, EventSettings, IScheduler, Scheduler,
    SchedulerHeader, SchedulerHeaderProps, SchedulerResource, SchedulerResourceHeaderProps,
    WeekView, SchedulerEventClickEvent, SchedulerCellClickEvent, SchedulerDataChangeEvent, WorkHoursProps
} from '@syncfusion/react-scheduler';
import { DropDownList, ChangeEvent } from '@syncfusion/react-dropdowns';
import { ToolbarItem, ToolbarSpacer, OverflowMode } from '@syncfusion/react-navigations';
import { Dialog } from '@syncfusion/react-popups';
import { Button, Variant, Size, Color } from '@syncfusion/react-buttons';
import { formatDate, DragDrop, Draggable, Droppable, DropEvent } from '@syncfusion/react-base';

// Rooms double as the scheduler resource collection; each has a Color and a Capacity
const techEventRooms: Record<string, any>[] = [
    { Id: 1, RoomName: 'Room A', Capacity: 100, Color: '#0F6CBD' },
    { Id: 2, RoomName: 'Room B', Capacity: 200, Color: '#B71C1C' },
    { Id: 3, RoomName: 'Room C', Capacity: 300, Color: '#E65100' },
    { Id: 4, RoomName: 'Room D', Capacity: 400, Color: '#558B2F' }
];
const ROOM_FILTER_DATA = [
    { Id: 0, RoomName: 'All', Capacity: 0, Color: '#666' },
    ...techEventRooms
];

const workHours: WorkHoursProps = { highlight: true, start: '08:00', end: '18:00' };

const formatTime = (date?: Date | null) => date ? formatDate(date, { format: 'h:mm a' }) : '';
const ROOM_CONFLICT_MESSAGE = 'This room is already booked for this time slot. Please select a different room or time.';

export default function App() {
    const schedulerRef = useRef<IScheduler | null>(null);
    const draggingSession = useRef<Record<string, any> | null>(null);
    const dragHelperRef = useRef<HTMLElement | null>(null);
    const [selectedRoom, setSelectedRoom] = useState<number>(0);
    const [scheduledEvents, setScheduledEvents] = useState(() => [...TechnicalEventData]);
    const [unscheduledEvents, setUnscheduledEvents] = useState(() => [...CloudSecurityEventData, ...AIAutomationEventData]);
    const [notice, setNotice] = useState<string>('');
    const [showNotice, setShowNotice] = useState<boolean>(false);

    // Filtering state drives a derived view of rooms and events
    const visibleRooms = techEventRooms.filter(room => selectedRoom === 0 || room.Id === selectedRoom);
    const visibleScheduledEvents = scheduledEvents.filter(event => selectedRoom === 0 || event.RoomId === selectedRoom);

    const resources: SchedulerResource[] = [{
        name: 'Rooms', field: 'RoomId', title: 'Room',
        dataSource: visibleRooms, textField: 'RoomName', idField: 'Id', colorField: 'Color'
    }];
    const eventSettings: EventSettings = { dataSource: visibleScheduledEvents };

    const closeNotice = () => setShowNotice(false);

    // ---- Business rules ----
    const hasRoomConflict = (roomId: number, startTime: Date, endTime: Date, excludedEventId?: number) =>
        scheduledEvents.some((event) => {
            if (event.Id === excludedEventId || event.RoomId !== roomId || !event.StartTime || !event.EndTime) {
                return false;
            }
            const existingStart = new Date(event.StartTime);
            const existingEnd = new Date(event.EndTime);
            return startTime < existingEnd && endTime > existingStart;
        });

    const validateSchedule = (room: Record<string, any>, startTime: Date, endTime: Date, capacity: number, excludedEventId?: number): boolean => {
        if (hasRoomConflict(room.Id, startTime, endTime, excludedEventId)) {
            showValidationNotice(ROOM_CONFLICT_MESSAGE);
            return false;
        }
        if (capacity > room.Capacity) {
            showValidationNotice('This room cannot accommodate the stated number of attendees. Please select a room with a suitable capacity.');
            return false;
        }
        return true;
    };

    const showValidationNotice = (message: string) => {
        setNotice(message);
        setShowNotice(true);
        clearDrag();
    };

    // Validate scheduler-initiated changes (drag/resize/editor): cancel to revert invalid ones
    const onDataChangeStart = (args: SchedulerDataChangeEvent) => {
        const changedEvent = args.changedRecords?.[0];
        if (!changedEvent) { return; }
        const room = techEventRooms.find((item) => item.Id === changedEvent.RoomId);
        if (!room || !validateSchedule(room, changedEvent.StartTime, changedEvent.EndTime, changedEvent.Capacity, changedEvent.Id)) {
            args.cancel = true;
        }
    };

    const onDataChangeComplete = (args: SchedulerDataChangeEvent) => {
        const changedEvent = args.changedRecords?.[0];
        if (!changedEvent) { return; }
        setScheduledEvents((previous) =>
            previous.map((event) =>
                event.Id === changedEvent.Id
                    ? { ...event, StartTime: changedEvent.StartTime, EndTime: changedEvent.EndTime, RoomId: changedEvent.RoomId }
                    : event
            )
        );
    };

    // ---- Drag-and-drop of unscheduled sessions onto the grid ----
    const createDragHelper = (item: Record<string, any>): HTMLElement | null => {
        dragHelperRef.current?.remove();
        const source = document.querySelector<HTMLElement>(`[data-event-id="${item.Id}"]`);
        if (!source) { return null; }
        const helper = source.cloneNode(true) as HTMLElement;
        helper.removeAttribute('data-event-id');
        helper.style.width = `${source.offsetWidth}px`;
        document.body.appendChild(helper);
        dragHelperRef.current = helper;
        return helper;
    };

    const clearDrag = () => {
        dragHelperRef.current?.remove();
        dragHelperRef.current = null;
        draggingSession.current = null;
    };

    const onSchedulerDrop = (args: DropEvent) => {
        const session = draggingSession.current;
        const dropTarget = (args.event?.target as HTMLElement | null)?.closest<HTMLElement>('.sf-work-cells, .sf-all-day-cell, .sf-appointment');
        if (!session || !dropTarget) { clearDrag(); return; }
        if (dropTarget.classList.contains('sf-appointment')) {
            return showValidationNotice(ROOM_CONFLICT_MESSAGE);
        }
        // Read the drop cell's time and the resource row it belongs to
        const cellDetails = schedulerRef.current?.getCellDetails(dropTarget);
        const groupIndex = Number(dropTarget.dataset.groupIndex);
        const room = Number.isInteger(groupIndex) ? visibleRooms[groupIndex] : undefined;
        if (!cellDetails || !room) { clearDrag(); return; }

        const startTime = new Date(cellDetails.startTime);
        const endTime = new Date(startTime.getTime() + session.DurationInMinutes * 60000);
        if (!validateSchedule(room, startTime, endTime, session.Capacity)) { return; }

        setScheduledEvents((previous) => {
            const id = Math.max(...previous.map((item) => item.Id), 0) + 1;
            return [...previous, { ...session, Id: id, StartTime: startTime, EndTime: endTime, RoomId: room.Id }];
        });
        setUnscheduledEvents((previous) => previous.filter((item) => item.Id !== session.Id));
        clearDrag();
    };

    // ---- Templates ----
    const isBreakEvent = (event: EventModel) => (event as Record<string, unknown>).EventType === 'Break';

    const eventTemplate = (props: EventModel) => (
        <div className={`tech-event-template ${isBreakEvent(props) ? 'tech-break-event' : ''}`}
            title={`${props.subject}\n${formatTime(props.startTime)} - ${formatTime(props.endTime)}\n${props.description}`}>
            <div className="tech-event-title sf-ellipsis bold">{props.subject}</div>
            <div className="tech-event-time sf-ellipsis">{formatTime(props.startTime)} - {formatTime(props.endTime)}</div>
        </div>
    );

    const roomHeaderTemplate = ({ resourceData }: SchedulerResourceHeaderProps) => (
        <div className="tech-room-header text-align-center width-100p" title={`${resourceData.RoomName} - Capacity: ${resourceData.Capacity}`}>
            <div className="tech-room-title bold">{resourceData.RoomName}</div>
            <div>Capacity - {resourceData.Capacity}</div>
        </div>
    );

    // ---- Custom header with a room filter ----
    const customHeader = (props: SchedulerHeaderProps) => (
        <SchedulerHeader overflowMode={OverflowMode.Scrollable} {...props}>
            {props.previous}
            {props.next}
            {props.dateRange}
            <ToolbarSpacer />
            {props.viewSwitcher}
            <ToolbarItem>
                <DropDownList
                    dataSource={ROOM_FILTER_DATA}
                    fields={{ text: 'RoomName', value: 'Id' }}
                    value={selectedRoom}
                    placeholder="Room"
                    onChange={(args?: ChangeEvent) => setSelectedRoom(args?.value as number)}
                />
            </ToolbarItem>
        </SchedulerHeader>
    );

    // ---- Custom quick popup: read-only event details ----
    const getEventRoom = (event: EventModel) => {
        const { RoomId } = event as Record<string, unknown>;
        return techEventRooms.find((room) => room.Id === Number(RoomId));
    };

    const getQuickInfoDurationText = (eventData: EventModel) => {
        if (!eventData.startTime || !eventData.endTime) { return ''; }
        const date = formatDate(eventData.startTime, { format: 'EEEE, MMMM d, y' });
        return `${date} (${formatTime(eventData.startTime)} - ${formatTime(eventData.endTime)})`;
    };

    const eventQuickInfoHeader = ({ eventData }: { eventData: EventModel }) => {
        const room = getEventRoom(eventData);
        return (
            <div className="tech-quick-info-wrapper">
                <Button
                    className="tech-quick-info-close"
                    variant={Variant.Outlined}
                    size={Size.Small}
                    color={Color.Error}
                    aria-label="Close event details"
                    onClick={() => schedulerRef.current?.closeQuickInfoPopup()}
                >
                    Close
                </Button>
                <div className="tech-quick-info-header" style={{ borderBottom: '4px solid ' + room?.Color }}>
                    <div className="tech-quick-info-title sf-ellipsis bold">{eventData.subject}</div>
                    <div className="tech-quick-info-date">{getQuickInfoDurationText(eventData)}</div>
                </div>
            </div>
        );
    };

    const eventQuickInfoContent = ({ eventData }: { eventData: EventModel }) => {
        const room = getEventRoom(eventData);
        return (
            <div className="tech-quick-info-content display-grid gap-12">
                <div className="display-grid gap-10">
                    <span>Room</span>
                    <span>{room?.RoomName || ''}</span>
                </div>
                <div className="display-grid gap-10">
                    <span>Audience</span>
                    <span>{String((eventData as Record<string, unknown>).Capacity || '')}</span>
                </div>
            </div>
        );
    };

    // Block interactions: break events are read-only; no built-in editor/popups
    const onEventClick = (args: SchedulerEventClickEvent) => { args.cancel = isBreakEvent(args.data); };
    const onEventDoubleClick = (args: SchedulerEventClickEvent) => { args.cancel = true; };
    const preventCellAction = (args: SchedulerCellClickEvent) => { args.cancel = true; };

    return (
        <div className="tech-event-organizer">
            <DragDrop>
                <div className="tech-event-layout display-grid gap-12">
                    {/* The scheduler is a drop target for unscheduled session cards */}
                    <Droppable scope="tech-events" accept=".tech-session-card" onDrop={onSchedulerDrop}>
                        <Scheduler
                            ref={schedulerRef}
                            height="620px"
                            defaultSelectedDate={new Date(2026, 6, 13)}
                            defaultView="Day"
                            startHour="08:00"
                            endHour="18:00"
                            workHours={workHours}
                            timeScale={{ enable: true, interval: 60, slotCount: 3 }}
                            eventSettings={eventSettings}
                            resources={resources}
                            group={{ resources: ['Rooms'] }}
                            header={customHeader}
                            onCellClick={preventCellAction}
                            onCellDoubleClick={preventCellAction}
                            onEventClick={onEventClick}
                            onEventDoubleClick={onEventDoubleClick}
                            onDataChangeStart={onDataChangeStart}
                            onDataChangeComplete={onDataChangeComplete}
                            quickInfo={{
                                editHeader: eventQuickInfoHeader,
                                editContent: eventQuickInfoContent,
                                editFooter: () => null
                            }}
                        >
                            <DayView eventTemplate={eventTemplate} resourceHeader={roomHeaderTemplate} />
                            <WeekView eventTemplate={eventTemplate} resourceHeader={roomHeaderTemplate} />
                        </Scheduler>
                    </Droppable>

                    {/* Unscheduled sessions panel — each card is a Draggable */}
                    <div className="tech-unscheduled-panel overflow-hidden">
                        <div className="tech-panel-header bold padding-12 text-align-center">Unscheduled Events</div>
                        <div className="tech-session-list overflow-auto">
                            {unscheduledEvents.map((item) => (
                                <Draggable
                                    key={item.Id}
                                    scope="tech-events"
                                    clone={true}
                                    helper={() => createDragHelper(item)}
                                    onDragStart={() => { draggingSession.current = item; }}
                                    onDragStop={clearDrag}
                                >
                                    <div className="tech-session-card" data-event-id={item.Id}>
                                        <div className="bold">{item.Subject}</div>
                                        <div>Duration: {item.DurationInMinutes} minutes</div>
                                        <div>Audience Size: {item.Capacity}</div>
                                    </div>
                                </Draggable>
                            ))}
                        </div>
                    </div>
                </div>
            </DragDrop>

            {/* Validation notice dialog */}
            <Dialog
                open={showNotice}
                header="Notice"
                style={{ width: '335px' }}
                modal={true}
                closeIcon={true}
                onClose={closeNotice}
                footer={<Button variant={Variant.Filled} onClick={closeNotice}>OK</Button>}
            >
                {notice}
            </Dialog>
        </div>
    );
}
```

## Why this composes well

- **The `groupIndex` trick**: with resource grouping enabled, each work cell carries a `dataset.groupIndex` that maps directly to the visible resource — used here to resolve which room an external drop landed on. `getCellDetails` converts the same cell element to time data.
- **One validation chokepoint**: both the scheduler's own edit flow (`onDataChangeStart` → `args.cancel`) and the external drop flow (`validateSchedule` before committing state) share `hasRoomConflict` and the capacity check; invalid changes are reverted or never committed.
- **Read-only by suppression**: since all scheduling happens through drag and the quick popup is display-only, the built-in editor is suppressed by canceling click/double-click events rather than by `eventSettings` flags — a per-event form of the read-only pattern from [event-types.md](event-types.md).
- **Derived filtering**: room and track filtering happen before data reaches the scheduler (`visibleRooms`, `visibleScheduledEvents`), so the scheduler always receives already-filtered resources/events — no post-render manipulation.
