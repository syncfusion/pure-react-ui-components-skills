# Editor Window, Quick Popup, Context Menu, and Tooltip

The built-in editor window (modal dialog) handles creating and editing events. Alongside it, a quick popup appears on cell/event clicks for lightweight interactions, a context menu can be attached for right-click actions, and event tooltips display details on hover.

## Contents

- [Editor window](#editor-window) — SchedulerEditorProps, customizing built-in fields, field validation, full layout customization, view-level override
- [Quick popup](#quick-popup) — six renderers, validation, preventing the popup, built-in methods
- [Context menu integration](#context-menu-integration) — menu structure, target detection, dispatching actions
- [Event tooltip](#event-tooltip) — default tooltip and full customization APIs

## Editor window

Editor customization is managed through the Scheduler `editor` property, which accepts a function returning a `SchedulerEditorPopup` component (or any React node). This enables customizing the dialog chrome (header and footer), modifying fields and validation, replacing fields with custom inputs, or constructing a fully custom form layout.

### SchedulerEditorProps

- `data` — the current event data being created or edited.
- `originalData` — the original event data before any changes.
- `action` — the current editor action, such as `Add` or `Edit`.
- `dialogProps` — props for configuring the underlying dialog component.
- `fields` — an array of field configurations for the built-in form inputs.
- `onClose` — a callback function invoked when the editor is closed.
- `children` — custom React nodes to replace the default field layout.

### Customizing built-in fields

Modify existing fields without rebuilding the form by passing `SchedulerEditorField[]` configurations to `fields` — changing labels, placeholders, variants, or hiding inputs:

```tsx
import { DayView, MonthView, Scheduler, EventSettings, SchedulerEditorField, SchedulerEditorPopup, SchedulerEditorProps, WeekView, WorkWeekView } from "@syncfusion/react-scheduler";
import { Variant } from "@syncfusion/react-base";
import { ReactNode } from "react";

export default function App() {
    const eventSettings: EventSettings = { dataSource: defaultData };

    const customEditor = (props: SchedulerEditorProps): ReactNode => {
        const fields: SchedulerEditorField[] = [
            { name: 'subject', props: { labelMode: 'Auto', variant: Variant.Outlined, placeholder: 'Add title' } },
            { name: 'location', props: { labelMode: 'Auto', variant: Variant.Outlined, placeholder: 'Add location' } },
            { name: 'startDate', props: { variant: Variant.Outlined } },
            { name: 'startTime', props: { variant: Variant.Outlined } },
            { name: 'endDate', props: { variant: Variant.Outlined } },
            { name: 'endTime', props: { variant: Variant.Outlined } },
            { name: 'isAllDay', props: { label: 'Full day' } },
            { name: 'description', props: { variant: Variant.Outlined, placeholder: 'Notes' } }
        ];
        return (
            <SchedulerEditorPopup {...props} fields={fields} />
        );
    };

    return (
        <Scheduler
            height="550px"
            width="100%"
            eventSettings={eventSettings}
            editor={customEditor}
            defaultSelectedDate={new Date(2025, 0, 12)}
            startHour="09:00"
        >
            <DayView /><WeekView /><WorkWeekView /><MonthView />
        </Scheduler>
    );
}
```

### Field validation

Add a `validationRule` to any field configuration to enforce constraints (required, minLength, maxLength, ...) validated on submission. Validation failures prevent the event from being saved, giving immediate feedback:

```tsx
const customEditor = (props: SchedulerEditorProps): ReactNode => {
    const fields: SchedulerEditorField[] = [
        {
            name: 'subject', validationRule: {
                required: [true, 'Subject is required'], minLength: [3, 'Subject must be at least 3 characters']
            }
        },
        { name: 'location', validationRule: { required: [true, 'Location is required'] } },
        { name: 'startDate', validationRule: { required: [true, 'Start Date is required'] } },
        { name: 'startTime', validationRule: { required: [true, 'Start Time is required'] } },
        { name: 'endDate', validationRule: { required: [true, 'End Date is required'] } },
        { name: 'endTime', validationRule: { required: [true, 'End Time is required'] } },
        { name: 'isAllDay' },
        { name: 'description', validationRule: { required: [true, 'Description is required'] } }
    ];

    return (
        <SchedulerEditorPopup className="scheduler-field-validation" {...props} fields={fields} />
    );
};
```

### Full layout customization

For advanced scenarios, use the `children` property of `SchedulerEditorPopup` — when children are provided, default field generation is bypassed and you control the entire DOM structure. Validate individual inputs with the `rule` property on form components, and run custom validation at save time in `onEditorSubmit` by setting `args.cancel = true` when validation fails (then merge form values into `args.data` on success).

Complete example — a medical appointment form with custom fields (patient name, doctor dropdown, notes) built from Syncfusion Form components, with view-level field mapping:

```tsx
import { DayView, MonthView, Scheduler, SchedulerEditorPopup, SchedulerEditorProps, SchedulerEditorSubmitEvent, WeekView, WorkWeekView } from "@syncfusion/react-scheduler";
import { RefObject, useEffect, useMemo, useRef, useState } from "react";
import { DropDownList, DropDownListProps } from "@syncfusion/react-dropdowns";
import { DatePicker, DatePickerChangeEvent, TimePicker, TimePickerChangeEvent } from "@syncfusion/react-calendars";
import { Form, FormField, FormState, IFormValidator, TextArea, TextBox, ValidationRules, Variant } from "@syncfusion/react-inputs";

interface AppointmentData {
    PatientName: string;
    DoctorId: number;
    Notes: string;
    From: Date;
    To: Date;
}

const defaultData: AppointmentData = {
    PatientName: '', DoctorId: 4, Notes: '', From: new Date(), To: new Date()
};

export default function App() {
    const formDataRef = useRef<Partial<AppointmentData>>({});
    const formRef: RefObject<IFormValidator | null> = useRef<IFormValidator>(null);
    const isSubmittedRef = useRef(false);

    // Field mapping lets custom data keys drive the editor
    const eventSettings = {
        dataSource: patientData,
        fields: {
            id: 'AppointmentId',
            subject: 'PatientName',
            startTime: 'From',
            endTime: 'To',
            description: 'Notes'
        }
    };

    const doctorsData = [
        { DoctorId: 1, Name: 'Dr. Alice Johnson - Cardiologist' },
        { DoctorId: 2, Name: 'Dr. Bob Smith - Surgeon' },
        { DoctorId: 3, Name: 'Dr. Carol White - Neuro Specialist' },
        { DoctorId: 4, Name: 'Dr. David Brown - General Physician' }
    ];
    const doctorFields = { text: 'Name', value: 'DoctorId' };

    const formRules: ValidationRules = {
        patientName: {
            required: [true, 'Patient name is required'],
            minLength: [3, 'Name must be at least 3 characters']
        },
        notes: { required: [true, 'Notes is required'] }
    };

    // Save-time validation: cancel when invalid, merge values when valid
    const handleEditorSave = (args: SchedulerEditorSubmitEvent) => {
        isSubmittedRef.current = true;
        const isValid = formRef.current?.validate();
        if (!isValid) {
            args.cancel = true;
            return;
        }
        isSubmittedRef.current = false;
        args.data = { ...args.data, ...formDataRef.current };
    };

    const customEditor = (props: SchedulerEditorProps) => {
        const { action, data, originalData, open } = props;
        const [formData, setFormData] = useState<AppointmentData>(defaultData);
        const [formState, setFormState] = useState<FormState>();

        useEffect(() => { formDataRef.current = formData; }, [formData]);

        const combineDateAndTime = (datePart: Date | null, timePart: Date | null): Date => {
            const base = new Date((datePart || timePart || new Date()).getTime());
            const t = timePart || datePart;
            return new Date(base.setHours(t ? t.getHours() : 0, t ? t.getMinutes() : 0, 0, 0));
        };

        useEffect(() => {
            isSubmittedRef.current = false;
            if (!open) {
                setFormData(defaultData);
                return;
            }
            if (action === 'Edit' && originalData) {
                setFormData(originalData as unknown as AppointmentData);
            } else {
                setFormData(prev => ({
                    ...prev,
                    From: data?.startTime ? new Date(data.startTime) : prev.From,
                    To: data?.endTime ? new Date(data.endTime) : prev.To,
                }));
            }
        }, [open, action, originalData, data?.startTime, data?.endTime]);

        return (
            <SchedulerEditorPopup
                {...props}
                style={{ width: '600px' }}
                dialogProps={{
                    open: open as boolean,
                    header: action === 'Add' ? 'Book Appointment' : 'Edit Appointment',
                    animation: { effect: 'FadeZoom', duration: 400, delay: 0 }
                }}
            >
                {/* Full custom form:
                    <Form ref={formRef} rules={formRules} validateOnChange={true}
                          onFormStateChange={setFormState} initialValues={...}>
                      FormField "patientName" -> TextBox
                      FormField "doctorName"  -> DropDownList (dataSource={doctorsData}, fields={doctorFields})
                      FormField "from"/"to"   -> DatePicker + TimePicker pairs (combineDateAndTime on change)
                      FormField "notes"       -> TextArea
                    </Form> */}
            </SchedulerEditorPopup>
        );
    };

    return (
        <Scheduler
            height="550px"
            width="100%"
            defaultSelectedDate={new Date(2026, 1, 2)}
            defaultView="Week"
            eventSettings={eventSettings}
            editor={customEditor}
            onEditorSubmit={handleEditorSave}
            startHour='09:00'
        >
            <DayView /><WeekView /><WorkWeekView /><MonthView />
        </Scheduler>
    );
}
```

Key implementation points from the full pattern:

- Keep a `formDataRef` mirror of form state so `onEditorSubmit` (outside the editor function) can read current values.
- Show inline errors via `formState.errors['<field>']` when `formState.modified['<field>']` or `isSubmittedRef.current` is truthy.
- Use `combineDateAndTime` to merge DatePicker and TimePicker values into one `Date` per boundary.

### View-level editor override

The editor can also be customized per view — e.g. a simplified form for the `Month` view vs. a detailed form for `Day`. Define the `editor` property on the specific view directive to override the global editor configuration:

```tsx
<MonthView editor={simplifiedEditor} />
```

## Quick popup

The built-in quick popup (quickInfo) appears when clicking a cell or an existing event, providing a lightweight contextual interface without opening the editor window.

### Customizing the quick popup layout

The `quickInfo` property provides six renderer props covering header, content, and footer for both the cell (Add) and event (Edit) popup types. Each renderer receives contextual data (`cellData` or `eventData`) and returns a React node:

**Cell popup**

- `addHeader` — custom header for the cell popup (context or branding).
- `addContent` — custom content such as input fields, previews, or default values.
- `addFooter` — custom action buttons (e.g. Save, Cancel) for streamlined event creation.

**Event popup**

- `editHeader` — custom header showing event titles, status badges, or icons.
- `editContent` — replaces the default event form with a custom layout (attendee lists, file attachments, ...).
- `editFooter` — custom footer actions (e.g. Update, Delete, Duplicate) for existing appointments.

When the `adaptive` property is `true`, both popups automatically transform into a full-screen modal on mobile devices.

### Built-in methods for popup actions

`openEditor`, `closeQuickInfoPopup`, `addEvent`, and `saveEvent` (all accessed via the scheduler ref — see [crud-external-forms.md](crud-external-forms.md)) perform actions from inside quick popup renderers, like opening the full editor or programmatically adding events.

### Preventing the quick popup

Set `showQuickInfo={false}` to stop the quick popup from appearing on cell or event clicks.

### Validation in the quick popup

Validate cell popup fields before saving: run the validation logic in the footer renderer, check form state before invoking `addEvent`, display inline error messages on failure, and prevent the save.

Full example — customized quick popup with field-level validation:

```tsx
import { DayView, EventModel, IScheduler, MonthView, Scheduler, SchedulerCellDetails, WeekView, WorkWeekView } from "@syncfusion/react-scheduler";
import { useRef, useState, RefObject, useEffect } from "react";
import { Button, Variant } from "@syncfusion/react-buttons";
import { Color, formatDate, Size } from "@syncfusion/react-base";
import { TextArea, TextBox, ITextBox } from "@syncfusion/react-inputs";

export default function App() {
    const schedulerRef = useRef<IScheduler | null>(null);
    const [subject, setSubject] = useState<string>('');
    const [description, setDescription] = useState<string>('');
    const [errors, setErrors] = useState<{ subject?: string; description?: string }>({});
    const textBoxRef: RefObject<ITextBox | null> = useRef<ITextBox>(null);

    const resetForm = () => { setSubject(''); setDescription(''); setErrors({}); };

    const formatTimeRange = (start?: Date | null, end?: Date | null): string => {
        if (!start || !end || !(start instanceof Date) || !(end instanceof Date)) { return '—'; }
        return `${formatDate(start, { format: 'hh:mm a' })} - ${formatDate(end, { format: 'hh:mm a' })}`;
    };

    const validateSubject = (value: string): string | undefined =>
        (!value || value.trim() === '') ? 'Title is required' : undefined;

    const validateDescription = (value: string): string | undefined => {
        if (!value || value.trim() === '') return 'Description is required';
        if (value.length > 20) return 'Description must not exceed 20 characters';
        return undefined;
    };

    // Validation runs in the footer renderer before addEvent is invoked
    const validateForm = (): boolean => {
        const newErrors: { subject?: string; description?: string } = {};
        const subjectError = validateSubject(subject);
        const descriptionError = validateDescription(description);
        if (subjectError) newErrors.subject = subjectError;
        if (descriptionError) newErrors.description = descriptionError;
        setErrors(newErrors);
        textBoxRef?.current?.element?.focus();
        return Object.keys(newErrors).length === 0;
    };

    const handleMoreDetails = (cellData: SchedulerCellDetails) => {
        const updatedCellData = { ...cellData, Subject: subject, Description: description };
        schedulerRef?.current?.openEditor('Add', updatedCellData);
        schedulerRef?.current?.closeQuickInfoPopup();
        resetForm();
    };

    const handleEdit = (eventData: EventModel) => {
        schedulerRef?.current?.openEditor('Edit', eventData);
        schedulerRef?.current?.closeQuickInfoPopup();
    };

    const handleDelete = (eventData: EventModel) => {
        schedulerRef?.current?.deleteEvent(eventData);
        schedulerRef?.current?.closeQuickInfoPopup();
        resetForm();
    };

    const handleCancel = () => {
        schedulerRef?.current?.closeQuickInfoPopup();
        resetForm();
    };

    const handleSave = (cellData: SchedulerCellDetails) => {
        if (!validateForm()) return;
        schedulerRef?.current?.addEvent({
            Id: 'sf-' + Math.floor(Math.random() * 100),
            Subject: subject || 'New Event',
            Description: description,
            IsAllDay: cellData?.isAllDay,
            StartTime: cellData?.startTime,
            EndTime: cellData?.endTime
        });
        schedulerRef?.current?.closeQuickInfoPopup();
        resetForm();
    };

    const handleKeyDown = (e: React.KeyboardEvent<HTMLInputElement> | React.KeyboardEvent<HTMLButtonElement>, callback: () => void) => {
        if (e.key === 'Enter' || e.key === ' ') {
            e.preventDefault();
            callback();
        }
    };

    // --- Cell popup ---
    const customAddHeader = () => (
        <div className="quick-info-header display-flex">
            <Button className="custom-close-icon" variant={Variant.Standard} onClick={() => { handleCancel(); }}>×</Button>
        </div>
    );

    const customAddContent = ({ cellData }: { cellData: SchedulerCellDetails }) => {
        useEffect(() => {
            if (textBoxRef?.current) {
                textBoxRef.current.element?.focus();
                resetForm();
            }
        }, [textBoxRef?.current?.element]);

        return (
            <div className="quick-info-content padding-inline-20">
                <div className="form-field">
                    <TextBox ref={textBoxRef} placeholder="Add title" size={Size.Large} variant={Variant.Standard} value={subject}
                        onKeyDown={(e) => handleKeyDown(e, () => handleSave(cellData))}
                        onChange={(e) => {
                            setSubject(e.value as string);
                            if (errors.subject) { setErrors(prev => ({ ...prev, subject: undefined })); }
                        }}
                    />
                    {errors.subject && <div className="sf-form-error">{errors.subject}</div>}
                </div>
                <div className="quick-info-time display-flex gap-10">
                    <span className="sf-form-label time-display">{formatTimeRange(cellData?.startTime, cellData?.endTime)}</span>
                </div>
                <div className="form-field">
                    <TextArea placeholder="Add description" variant={Variant.Filled} value={description}
                        onChange={(e) => {
                            setDescription(e.value as string);
                            if (errors.description) { setErrors(prev => ({ ...prev, description: undefined })); }
                        }}
                    />
                    {errors.description && <div className="sf-form-error">{errors.description}</div>}
                </div>
            </div>
        );
    };

    const customAddFooter = ({ cellData }: { cellData: SchedulerCellDetails }) => (
        <div className="quick-info-footer display-flex">
            <Button variant={Variant.Standard} size={Size.Medium} onClick={() => handleMoreDetails(cellData)}>More options</Button>
            <Button variant={Variant.Filled} size={Size.Medium} color={Color.Primary} onClick={() => handleSave(cellData)}>Save</Button>
        </div>
    );

    // --- Event popup ---
    const customEditHeader = ({ eventData }: { eventData: EventModel }) => (
        <div className="quick-info-header-edit display-flex padding-10">
            <div className="action-icons display-flex gap-6">
                <Button variant={Variant.Standard} color={Color.Secondary} onClick={() => handleEdit(eventData)}>Edit</Button>
                <Button variant={Variant.Standard} color={Color.Secondary} onClick={() => handleDelete(eventData)}>Delete</Button>
                <Button variant={Variant.Standard} color={Color.Secondary} onClick={() => handleCancel()}>Close</Button>
            </div>
        </div>
    );

    const customEditContent = ({ eventData }: { eventData: EventModel }) => {
        const calculateDuration = (eventData: EventModel) => {
            if (eventData.isAllDay) {
                const hours: number = Math.floor(((eventData?.endTime as Date)?.getTime() - (eventData?.startTime as Date)?.getTime()) / 3600000) + 24;
                return hours > 0 ? `${hours} hours` : "24 hours";
            }
            const minutes: number = Math.floor(((eventData?.endTime as Date)?.getTime() - (eventData?.startTime as Date)?.getTime()) / 60000);
            return `${minutes} minutes`;
        };

        return (
            <div className="quick-info-content-edit display-flex gap-10">
                <div className="sf-font-size-xl sb-content-color">
                    <div className="sf-ellipsis" title={eventData?.subject}>{eventData.subject}</div>
                </div>
                <div className="sf-font-size-sm display-flex gap-10">
                    <span>{calculateDuration(eventData)}</span>
                </div>
                {eventData?.location && (
                    <div className="sf-font-size-sm display-flex gap-10">
                        <span>{eventData.location}</span>
                    </div>
                )}
                {eventData?.description && (
                    <div className="sf-font-size-sm display-flex gap-10">
                        <span>{eventData.description}</span>
                    </div>
                )}
            </div>
        );
    };

    return (
        <Scheduler
            ref={schedulerRef}
            height="650px"
            startHour="09:00"
            defaultSelectedDate={new Date(2025, 0, 10)}
            eventSettings={{ dataSource: defaultData }}
            quickInfo={{
                addHeader: customAddHeader,
                addContent: customAddContent,
                addFooter: customAddFooter,
                editHeader: customEditHeader,
                editContent: customEditContent,
                editFooter: () => <div />
            }}
        >
            <DayView /><WeekView /><WorkWeekView /><MonthView />
        </Scheduler>
    );
}
```

The example also shows keyboard accessibility inside popups: wire Enter/Space on inputs and buttons via a `handleKeyDown` helper.

## Context menu integration

Attach an interactive context menu to both **cells** and **events** for right-click (desktop) or long-press (touch) access to frequent actions — without cluttering the UI or interrupting the scheduling flow.

The `ContextMenu` (from `@syncfusion/react-navigations`) is externally rendered and linked to the scheduler by assigning the scheduler container element as the menu's target. Based on where the user right-clicks, the menu dynamically switches between cell actions (create events / jump to today) and event actions (edit or delete).

### Key APIs for contextual operations

- `openEditor()` — open the event editor for creating or modifying events.
- `deleteEvent()` — remove the selected event (option occurrence/series variants: see [crud-external-forms.md](crud-external-forms.md)).
- `getEventDetails(target)` — retrieve event data when a user right-clicks an event.
- `getCellDetails(target)` — retrieve cell data for the right-clicked cell.
- Quick navigation — selecting "Today" updates `selectedDate` to jump to the current date.

### Structure

```tsx
// Cell actions
const cellMenuItems = [
    { text: 'New Event', id: 'Add', icon: <AddNotesIcon/> },
    { text: 'New Recurring Event', id: 'AddRecurrence', icon: <RepeatIcon/> },
    { text: 'Today', id: 'Today', icon: <DayIcon/> }
];
// Simple (non-recurring) event actions
const eventMenuItems = [
    { text: 'Edit Event', id: 'Edit', icon: <EditIcon/> },
    { text: 'Delete Event', id: 'Delete', icon: <DeleteNotesIcon/> }
];
// Recurring events get nested occurrence/series options
const recurrenceEventMenuItems = [
    { text: 'Edit Event', id: 'EditRecurrenceEvent', icon: <EditIcon/>, items: [
        { text: 'Edit Occurrence', id: 'EditOccurrence' },
        { text: 'Edit Series', id: 'EditSeries' }
    ]},
    { text: 'Delete Event', id: 'DeleteRecurrenceEvent', icon: <DeleteNotesIcon/>, items: [
        { text: 'Delete Occurrence', id: 'DeleteOccurrence' },
        { text: 'Delete Series', id: 'DeleteSeries' }
    ]}
];
```

Before the menu opens, inspect the click target to decide which menu to show: if it closes over `.sf-appointment`, fetch details via `getEventDetails` and show event items (recurrence variant when the event has a `recurrenceRule` or `recurrenceID`); otherwise, if it matches a work cell selector (`.sf-work-cells, .sf-all-day-cell`, plus `.sf-header-cells` outside month view), show cell items:

```tsx
const onContextMenuBeforeOpen = (args: Event) => {
    const target = (args?.target as HTMLElement) ?? null;
    if (!schedulerRef.current || !target) { return; }

    const selectedEvent: Element | null = target.closest?.('.sf-appointment');
    if (selectedEvent) {
        selectedTarget.current = selectedEvent;
        const eventDetails: EventModel | null = schedulerRef.current.getEventDetails(selectedEvent);
        if (eventDetails && (eventDetails.recurrenceRule || eventDetails.recurrenceID)) {
            setMenu(recurrenceEventMenuItems);
        } else {
            setMenu(eventMenuItems);
        }
        setOpen(true);
        return;
    }

    const isMonthView: boolean = !!target.closest?.('.sf-month-view');
    const cellSelector: string = isMonthView ? '.sf-work-cells, .sf-all-day-cell' : '.sf-work-cells, .sf-all-day-cell, .sf-header-cells';
    const selectedCell: HTMLElement | null = target.closest?.(cellSelector) ?? null;
    if (selectedCell) {
        selectedTarget.current = selectedCell;
        setMenu(cellMenuItems);
        setOpen(true);
    }
};
```

On select, dispatch to the matching scheduler API. For `Add`/`AddRecurrence`, fetch cell details and open the editor (pre-filling a `recurrenceRule` for the recurring case); for `Today`, set the selected date to `new Date()`:

```tsx
const onContextMenuSelect = (args: MenuSelectEvent) => {
    const type: string | undefined = args?.item?.id;
    let selectedEvent: Record<string, any> | null = null;
    if (!schedulerRef.current || !type) { return; }

    if (selectedTarget.current) {
        const details = schedulerRef.current.getEventDetails(selectedTarget.current);
        if (details !== null) { selectedEvent = details as Record<string, any>; }
    }

    switch (type) {
        case 'Today':
            setSelectedDate(new Date());
            break;
        case 'Add':
        case 'AddRecurrence': {
            const activeCellDetails = schedulerRef.current.getCellDetails(selectedTarget.current);
            if (type === 'Add') {
                schedulerRef.current.openEditor(type, activeCellDetails as SchedulerCellDetails);
            }
            if (type === 'AddRecurrence') {
                const cellInfo: EventModel = {
                    startTime: (activeCellDetails as SchedulerCellDetails).startTime,
                    endTime: (activeCellDetails as SchedulerCellDetails).endTime,
                    isAllDay: (activeCellDetails as SchedulerCellDetails).isAllDay,
                    recurrenceRule: 'FREQ=DAILY;INTERVAL=1;'
                };
                schedulerRef.current.openEditor('Add', cellInfo);
            }
            break;
        }
        case 'Edit':
            if (selectedEvent) { schedulerRef.current.openEditor(type, selectedEvent); }
            break;
        case 'EditOccurrence':
        case 'EditSeries':
            if (selectedEvent) { schedulerRef.current.openEditor(type, selectedEvent); }
            break;
        case 'Delete':
            if (selectedEvent) { schedulerRef.current.deleteEvent(selectedEvent); }
            break;
        case 'DeleteOccurrence':
            if (selectedEvent) { schedulerRef.current.deleteEvent(selectedEvent, 'DeleteOccurrence'); }
            break;
        case 'DeleteSeries':
            if (selectedEvent) { schedulerRef.current.deleteEvent(selectedEvent, 'DeleteSeries'); }
            break;
    }
};
```

Render the menu inside a wrapper `div` that also holds the `<Scheduler>`, capture the wrapper with a callback ref, and pass it as the ContextMenu's `targetRef`:

```tsx
const setContainerRef = (el: HTMLDivElement | null) => {
    targetRef.current = el as HTMLElement | null;
};

<div className="scheduler-control" ref={setContainerRef}>
    <Scheduler ref={schedulerRef} selectedDate={selectedDate} onSelectedDateChange={onSelectedDateChange} ...>
        <DayView /><WeekView /><MonthView />
    </Scheduler>
    <ContextMenu
        open={open}
        targetRef={targetRef as React.RefObject<HTMLElement>}
        onOpen={onContextMenuBeforeOpen}
        onClose={onContextMenuClose}
        onSelect={onContextMenuSelect}
    >
        {/* renderMenuItems(menu) — MenuItem/MenuItemIcon/MenuItemLabel per item, recursive for nested items */}
    </ContextMenu>
</div>
```

## Event tooltip

Built-in tooltips show event details without opening the editor. Setting `eventTooltip={true}` displays a default tooltip on hover (desktop) or long press (touch) with the event's subject and time:

```tsx
<Scheduler eventTooltip={true} ...>
    <DayView /><WeekView /><MonthView />
</Scheduler>
```

### Tooltip customization

`eventTooltip` also accepts an object with a full React template replacing the default content. Available APIs:

| **API** | **Type** | **Purpose** |
| --- | --- | --- |
| `content` | `React.ReactNode` | Specifies the content of the tooltip template. |
| `arrow` | `boolean` | Shows or hides the pointer of the tooltip. |
| `position` | `Position` | Position of the tooltip element relative to the target element. |
| `animation` | `TooltipAnimationOptions` | Animation options for the tooltip's open and close states. |
| `followCursor` | `boolean` | Allows the tooltip to follow mouse-pointer movement over the target element. |
| `opensOn` | `string` | Specifies the device mode used to display the tooltip content. |
| `sticky` | `boolean` | Keeps the tooltip open until it is manually closed. |

Custom template example (title, location, and a formatted time range with icons and structured layout):

```tsx
import { Position } from '@syncfusion/react-popups';

const tooltipContent = ({ data }: { data: { subject?: string; location?: string; startTime?: string | Date; endTime?: string | Date; } }) => {
    const formatDateRange = (startTime?: string | Date, endTime?: string | Date) => {
        if (!startTime || !endTime) { return `${String(startTime ?? '')} - ${String(endTime ?? '')}`; }
        const start = new Date(startTime as string | number | Date);
        const end = new Date(endTime as string | number | Date);
        if (isNaN(start.getTime()) || isNaN(end.getTime())) { return `${String(startTime)} - ${String(endTime)}`; }
        const startMonth = start.toLocaleString('default', { month: 'short' });
        const endMonth = end.toLocaleString('default', { month: 'short' });
        const startDay = start.getDate();
        const endDay = end.getDate();
        const startYear = start.getFullYear();
        const endYear = end.getFullYear();
        const startTimeStr = start.toLocaleString('default', { hour: 'numeric', minute: '2-digit', hour12: true });
        const endTimeStr = end.toLocaleString('default', { hour: 'numeric', minute: '2-digit', hour12: true });
        if (startYear !== endYear) {
            return `${startMonth} ${startDay}, ${startYear} - ${endMonth} ${endDay}, ${endYear} (${startTimeStr} - ${endTimeStr})`;
        } else if (startMonth !== endMonth) {
            return `${startMonth} ${startDay} - ${endMonth} ${endDay}, ${startYear} (${startTimeStr} - ${endTimeStr})`;
        } else if (startDay !== endDay) {
            return `${startMonth} ${startDay} - ${endDay}, ${startYear} (${startTimeStr} - ${endTimeStr})`;
        } else {
            return `${startMonth} ${startDay}, ${startYear} (${startTimeStr} - ${endTimeStr})`;
        }
    };

    return (
        <div className='tooltip-template flex'>
            <div className='tooltip-content gap-8'>
                <div className='tooltip-subject align-item-center display-flex gap-6'>
                    <strong>{data?.subject ?? 'Untitled'}</strong>
                </div>
                <div className='display-grid gap-8 tooltip-text'>
                    <div className='align-item-center display-flex gap-6'>{String(data?.location ?? '')}</div>
                    <div className='timezone-icon align-item-center display-flex gap-6'>
                        {formatDateRange(data?.startTime, data?.endTime)}
                    </div>
                </div>
            </div>
        </div>
    );
};

<Scheduler
    eventTooltip={{
        content: tooltipContent,
        arrow: true,
        position: 'TopCenter' as Position
    }}
    ...
>
```

Common positions: `BottomCenter`, `RightTop`, `LeftTop`, `TopCenter`.
