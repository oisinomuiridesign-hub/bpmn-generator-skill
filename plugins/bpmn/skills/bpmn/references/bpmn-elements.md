# BPMN Elements Reference

Complete mapping from business process concepts to LucidChart Standard Import BPMN shape types.

---

## Events

Events represent something that happens during a process.

### Start Events
The trigger that kicks off the process.

| Variant | eventType | When to Use |
|---------|-----------|-------------|
| None (generic) | `"none"` | Process starts manually or trigger is unspecified |
| Message | `"message"` | Triggered by receiving a message/email/webhook |
| Timer | `"timer"` | Triggered on a schedule or at a specific time |
| Signal | `"signal"` | Triggered by a broadcast signal from another process |
| Conditional | `"conditional"` | Triggered when a business condition becomes true |

```json
{
  "id": "start_1",
  "type": "bpmnEvent",
  "eventGroup": "start",
  "eventType": "message",
  "boundingBox": {"x": 100, "y": 200, "w": 40, "h": 40},
  "text": "Form Submitted",
  "style": {"fill": {"type": "color", "color": "#4CAF50"}}
}
```

### End Events
The outcome when the process completes.

| Variant | eventType | When to Use |
|---------|-----------|-------------|
| None (generic) | `"none"` | Process simply ends |
| Message | `"message"` | Process ends by sending a notification |
| Error | `"error"` | Process ends due to an error |
| Terminate | `"terminate"` | Immediately stops all process activity |
| Signal | `"signal"` | Process ends and broadcasts a signal |

```json
{
  "id": "end_1",
  "type": "bpmnEvent",
  "eventGroup": "end",
  "eventType": "none",
  "boundingBox": {"x": 1200, "y": 200, "w": 40, "h": 40},
  "text": "Complete",
  "style": {"fill": {"type": "color", "color": "#F44336"}}
}
```

### Intermediate Events
Something that happens during the process (between start and end).

| Variant | eventType | When to Use |
|---------|-----------|-------------|
| Timer | `"timer"` | Wait for a duration (e.g., "wait 3 days") |
| Message (catch) | `"message"` | Wait to receive a response |
| Error | `"error"` | Catch an error from a task |
| Escalation | `"escalation"` | Escalate to a higher authority |
| Signal | `"signal"` | Catch or throw a signal |
| Compensation | `"compensation"` | Undo a completed activity |

```json
{
  "id": "timer_1",
  "type": "bpmnEvent",
  "eventGroup": "intermediate",
  "eventType": "timer",
  "boundingBox": {"x": 500, "y": 200, "w": 40, "h": 40},
  "text": "Wait 48h"
}
```

---

## Activities

Activities represent work being performed.

### Tasks
A single unit of work.

```json
{
  "id": "task_1",
  "type": "bpmnActivity",
  "activityType": "task",
  "boundingBox": {"x": 300, "y": 180, "w": 160, "h": 80},
  "text": "Review Application",
  "style": {"fill": {"type": "color", "color": "#BBDEFB"}}
}
```

### Sub-processes
A compound activity containing nested steps (shown as collapsed).

```json
{
  "id": "sub_1",
  "type": "bpmnActivity",
  "activityType": "eventSubProcess",
  "boundingBox": {"x": 300, "y": 180, "w": 200, "h": 100},
  "text": "Payment Processing"
}
```

### Call Activities
Reference to a reusable process defined elsewhere.

```json
{
  "id": "call_1",
  "type": "bpmnActivity",
  "activityType": "callActivity",
  "boundingBox": {"x": 300, "y": 180, "w": 160, "h": 80},
  "text": "Run KYC Check"
}
```

---

## Gateways

Gateways control flow branching and merging.

| Type | gatewayType | Symbol | When to Use |
|------|-------------|--------|-------------|
| Exclusive (XOR) | `"exclusive"` | X | One path only (if/else, yes/no) |
| Parallel (AND) | `"parallel"` | + | All paths execute simultaneously |
| Inclusive (OR) | `"inclusive"` | O | One or more paths may execute |
| Event-based | `"eventBased"` | Pentagon | Path depends on which event happens first |

```json
{
  "id": "gw_1",
  "type": "bpmnGateway",
  "gatewayType": "exclusive",
  "boundingBox": {"x": 500, "y": 190, "w": 60, "h": 60},
  "style": {"fill": {"type": "color", "color": "#FFF9C4"}}
}
```

**Decision labeling:** Always label outgoing lines from gateways:
```json
{
  "id": "line_yes",
  "lineType": "elbow",
  "endpoint1": {"type": "shapeEndpoint", "style": "none", "shapeId": "gw_1"},
  "endpoint2": {"type": "shapeEndpoint", "style": "arrow", "shapeId": "task_approved"},
  "text": [{"text": "Yes", "position": 0.3, "side": "top"}]
}
```

---

## Pools and Lanes

Pools represent organizations or major participants. Lanes represent roles within a pool.

```json
{
  "id": "pool_1",
  "type": "bpmnPool",
  "title": "Organization Name",
  "vertical": false,
  "boundingBox": {"x": 50, "y": 50, "w": 1400, "h": 600},
  "lanes": [
    {"title": "Customer", "width": 200, "laneFill": "#E3F2FD"},
    {"title": "Sales Team", "width": 200, "laneFill": "#F3E5F5"},
    {"title": "Operations", "width": 200, "laneFill": "#E8F5E9"}
  ]
}
```

**Key rules:**
- `vertical: false` = horizontal lanes (standard BPMN orientation)
- Each lane's `"width"` = vertical height of that lane row
- Sum of all lane widths MUST equal `boundingBox.h`
- `title`, `width`, and `laneFill` are REQUIRED for each lane

### Black Box Pool
When an external party is involved but you don't show their internal process:

```json
{
  "id": "external_1",
  "type": "bpmnBlackBoxPool",
  "boundingBox": {"x": 50, "y": 700, "w": 1400, "h": 60},
  "text": "External Payment Provider"
}
```

---

## Data Objects

### Data Object
Represents data produced or consumed by tasks.

```json
{
  "id": "data_1",
  "type": "bpmnDataObject",
  "dataType": "none",
  "boundingBox": {"x": 350, "y": 50, "w": 40, "h": 50},
  "text": "Application Form"
}
```

### Data Store
Represents a persistent storage system (database, CRM, etc.).

```json
{
  "id": "store_1",
  "type": "bpmnDataStore",
  "boundingBox": {"x": 600, "y": 50, "w": 60, "h": 50},
  "text": "CRM Database"
}
```

---

## Annotations

### Text Annotation
Add notes or clarifications to the diagram.

```json
{
  "id": "note_1",
  "type": "bpmnTextAnnotation",
  "boundingBox": {"x": 300, "y": 50, "w": 200, "h": 60},
  "text": "SLA: Must respond within 24 hours"
}
```

### Group
Visual grouping of related elements (no flow impact).

```json
{
  "id": "group_1",
  "type": "bpmnGroup",
  "boundingBox": {"x": 250, "y": 150, "w": 400, "h": 200},
  "text": "Validation Phase"
}
```

---

## Color Palette

Standard colors used across all BPMN diagrams for consistency:

### Events & Gateways (fixed colors, regardless of lane)

| Element | Color | Hex |
|---------|-------|-----|
| Start Event | Green | `#4CAF50` |
| End Event | Red | `#F44336` |
| Gateway | Light Yellow | `#FFF9C4` |
| Error Event | Light Red | `#FFCDD2` |
| Timer Event | Light Orange | `#FFE0B2` |
| Data Object | Light Grey | `#F5F5F5` |
| Annotation | White | `#FFFFFF` |

### Tasks — MUST match their swim lane color

**CRITICAL:** Tasks (bpmnActivity) must use the same fill color as the swim lane they sit in. This makes it immediately obvious which lane owns a task, especially when connections cross multiple lanes.

| Lane # | Lane Fill | Task Fill (same) |
|--------|-----------|-----------------|
| Lane 1 | `#E3F2FD` | `#E3F2FD` |
| Lane 2 | `#F3E5F5` | `#F3E5F5` |
| Lane 3 | `#E8F5E9` | `#E8F5E9` |
| Lane 4 | `#FFF8E1` | `#FFF8E1` |
| Lane 5 | `#F5F5F5` | `#F5F5F5` |
| Lane 6 | `#FCE4EC` | `#FCE4EC` |

Do NOT use a single color (e.g. `#BBDEFB`) for all tasks — this loses the lane association.

### Lane Colors (pool background)

| Lane | Color | Hex |
|------|-------|-----|
| Lane 1 | Light Blue | `#E3F2FD` |
| Lane 2 | Light Purple | `#F3E5F5` |
| Lane 3 | Light Green | `#E8F5E9` |
| Lane 4 | Light Amber | `#FFF8E1` |
| Lane 5 | Light Grey | `#F5F5F5` |
| Lane 6 | Light Pink | `#FCE4EC` |
| Pool Header | Dark Blue | `#1565C0` |
