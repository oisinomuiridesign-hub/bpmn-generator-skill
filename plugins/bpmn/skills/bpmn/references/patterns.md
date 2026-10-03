# BPMN Patterns Library

Reusable JSON snippets for common business process patterns. Copy and adapt these when building diagrams.

---

## Pattern 1: Simple Sequential Flow

Start → Task → Task → End

**When to use:** Linear processes with no decisions or branching.

```json
{
  "shapes": [
    {"id": "start", "type": "bpmnEvent", "eventGroup": "start", "eventType": "none",
     "boundingBox": {"x": 100, "y": 200, "w": 40, "h": 40}, "text": "Start",
     "style": {"fill": {"type": "color", "color": "#4CAF50"}}},
    {"id": "task_1", "type": "bpmnActivity", "activityType": "task",
     "boundingBox": {"x": 220, "y": 180, "w": 160, "h": 80}, "text": "First Task",
     "style": {"fill": {"type": "color", "color": "#BBDEFB"}}},
    {"id": "task_2", "type": "bpmnActivity", "activityType": "task",
     "boundingBox": {"x": 440, "y": 180, "w": 160, "h": 80}, "text": "Second Task",
     "style": {"fill": {"type": "color", "color": "#BBDEFB"}}},
    {"id": "end", "type": "bpmnEvent", "eventGroup": "end", "eventType": "none",
     "boundingBox": {"x": 680, "y": 200, "w": 40, "h": 40}, "text": "End",
     "style": {"fill": {"type": "color", "color": "#F44336"}}}
  ],
  "lines": [
    {"id": "l1", "lineType": "elbow",
     "endpoint1": {"type": "shapeEndpoint", "style": "none", "shapeId": "start"},
     "endpoint2": {"type": "shapeEndpoint", "style": "arrow", "shapeId": "task_1"},
     "stroke": {"color": "#333333", "width": 2, "style": "solid"}},
    {"id": "l2", "lineType": "elbow",
     "endpoint1": {"type": "shapeEndpoint", "style": "none", "shapeId": "task_1"},
     "endpoint2": {"type": "shapeEndpoint", "style": "arrow", "shapeId": "task_2"},
     "stroke": {"color": "#333333", "width": 2, "style": "solid"}},
    {"id": "l3", "lineType": "elbow",
     "endpoint1": {"type": "shapeEndpoint", "style": "none", "shapeId": "task_2"},
     "endpoint2": {"type": "shapeEndpoint", "style": "arrow", "shapeId": "end"},
     "stroke": {"color": "#333333", "width": 2, "style": "solid"}}
  ]
}
```

---

## Pattern 2: Exclusive Gateway (Decision)

Task → Decision → Yes Path / No Path → Reconverge

**When to use:** Any yes/no decision, approval/rejection, qualification check.

**Routing rule:** Each output exits from a DIFFERENT side of the gateway diamond. Yes (main) exits RIGHT, No (alternate) exits BOTTOM.

```json
{
  "shapes": [
    {"id": "gw_decide", "type": "bpmnGateway", "gatewayType": "exclusive",
     "boundingBox": {"x": 400, "y": 190, "w": 60, "h": 60},
     "style": {"fill": {"type": "color", "color": "#FFF9C4"}}},
    {"id": "task_yes", "type": "bpmnActivity", "activityType": "task",
     "boundingBox": {"x": 540, "y": 110, "w": 160, "h": 80}, "text": "Approved Path",
     "style": {"fill": {"type": "color", "color": "#C8E6C9"}}},
    {"id": "task_no", "type": "bpmnActivity", "activityType": "task",
     "boundingBox": {"x": 540, "y": 310, "w": 160, "h": 80}, "text": "Rejected Path",
     "style": {"fill": {"type": "color", "color": "#FFCDD2"}}},
    {"id": "gw_join", "type": "bpmnGateway", "gatewayType": "exclusive",
     "boundingBox": {"x": 780, "y": 190, "w": 60, "h": 60},
     "style": {"fill": {"type": "color", "color": "#FFF9C4"}}}
  ],
  "lines": [
    {"id": "l_yes", "lineType": "elbow",
     "endpoint1": {"type": "shapeEndpoint", "style": "none", "shapeId": "gw_decide",
                   "position": {"x": 1, "y": 0.5}},
     "endpoint2": {"type": "shapeEndpoint", "style": "arrow", "shapeId": "task_yes",
                   "position": {"x": 0, "y": 0.5}},
     "stroke": {"color": "#333333", "width": 2, "style": "solid"},
     "text": [{"text": "Yes", "position": 0.3, "side": "top"}]},
    {"id": "l_no", "lineType": "elbow",
     "endpoint1": {"type": "shapeEndpoint", "style": "none", "shapeId": "gw_decide",
                   "position": {"x": 0.5, "y": 1}},
     "endpoint2": {"type": "shapeEndpoint", "style": "arrow", "shapeId": "task_no",
                   "position": {"x": 0, "y": 0.5}},
     "stroke": {"color": "#333333", "width": 2, "style": "solid"},
     "text": [{"text": "No", "position": 0.2, "side": "top"}]},
    {"id": "l_join_yes", "lineType": "elbow",
     "endpoint1": {"type": "shapeEndpoint", "style": "none", "shapeId": "task_yes",
                   "position": {"x": 1, "y": 0.5}},
     "endpoint2": {"type": "shapeEndpoint", "style": "arrow", "shapeId": "gw_join",
                   "position": {"x": 0.5, "y": 0}},
     "stroke": {"color": "#333333", "width": 2, "style": "solid"}},
    {"id": "l_join_no", "lineType": "elbow",
     "endpoint1": {"type": "shapeEndpoint", "style": "none", "shapeId": "task_no",
                   "position": {"x": 1, "y": 0.5}},
     "endpoint2": {"type": "shapeEndpoint", "style": "arrow", "shapeId": "gw_join",
                   "position": {"x": 0.5, "y": 1}},
     "stroke": {"color": "#333333", "width": 2, "style": "solid"}}
  ]
}
```

---

## Pattern 3: Parallel Gateway (Fork & Join)

Fork → Simultaneous Tasks → Join

**When to use:** Multiple things must happen at the same time (send email AND update CRM AND notify manager).

**Routing rule:** Fork exits from TOP (upper task) and BOTTOM (lower task). Join receives from TOP and BOTTOM. 60px minimum vertical gap between parallel tasks.

```json
{
  "shapes": [
    {"id": "gw_fork", "type": "bpmnGateway", "gatewayType": "parallel",
     "boundingBox": {"x": 400, "y": 190, "w": 60, "h": 60},
     "style": {"fill": {"type": "color", "color": "#FFF9C4"}}},
    {"id": "task_a", "type": "bpmnActivity", "activityType": "task",
     "boundingBox": {"x": 540, "y": 100, "w": 160, "h": 80}, "text": "Task A",
     "style": {"fill": {"type": "color", "color": "#BBDEFB"}}},
    {"id": "task_b", "type": "bpmnActivity", "activityType": "task",
     "boundingBox": {"x": 540, "y": 300, "w": 160, "h": 80}, "text": "Task B",
     "style": {"fill": {"type": "color", "color": "#BBDEFB"}}},
    {"id": "gw_join", "type": "bpmnGateway", "gatewayType": "parallel",
     "boundingBox": {"x": 780, "y": 190, "w": 60, "h": 60},
     "style": {"fill": {"type": "color", "color": "#FFF9C4"}}}
  ],
  "lines": [
    {"id": "l_fork_a", "lineType": "elbow",
     "endpoint1": {"type": "shapeEndpoint", "style": "none", "shapeId": "gw_fork",
                   "position": {"x": 0.5, "y": 0}},
     "endpoint2": {"type": "shapeEndpoint", "style": "arrow", "shapeId": "task_a",
                   "position": {"x": 0, "y": 0.5}},
     "stroke": {"color": "#333333", "width": 2, "style": "solid"}},
    {"id": "l_fork_b", "lineType": "elbow",
     "endpoint1": {"type": "shapeEndpoint", "style": "none", "shapeId": "gw_fork",
                   "position": {"x": 0.5, "y": 1}},
     "endpoint2": {"type": "shapeEndpoint", "style": "arrow", "shapeId": "task_b",
                   "position": {"x": 0, "y": 0.5}},
     "stroke": {"color": "#333333", "width": 2, "style": "solid"}},
    {"id": "l_join_a", "lineType": "elbow",
     "endpoint1": {"type": "shapeEndpoint", "style": "none", "shapeId": "task_a",
                   "position": {"x": 1, "y": 0.5}},
     "endpoint2": {"type": "shapeEndpoint", "style": "arrow", "shapeId": "gw_join",
                   "position": {"x": 0.5, "y": 0}},
     "stroke": {"color": "#333333", "width": 2, "style": "solid"}},
    {"id": "l_join_b", "lineType": "elbow",
     "endpoint1": {"type": "shapeEndpoint", "style": "none", "shapeId": "task_b",
                   "position": {"x": 1, "y": 0.5}},
     "endpoint2": {"type": "shapeEndpoint", "style": "arrow", "shapeId": "gw_join",
                   "position": {"x": 0.5, "y": 1}},
     "stroke": {"color": "#333333", "width": 2, "style": "solid"}}
  ]
}
```

---

## Pattern 4: Timer Wait

Task → Wait N days → Next Task

**When to use:** SLA waits, follow-up delays, cooling-off periods.

```json
{
  "shapes": [
    {"id": "timer_wait", "type": "bpmnEvent", "eventGroup": "intermediate", "eventType": "timer",
     "boundingBox": {"x": 400, "y": 200, "w": 40, "h": 40}, "text": "Wait 48h",
     "style": {"fill": {"type": "color", "color": "#FFE0B2"}}}
  ]
}
```

---

## Pattern 5: Error Handling

Task with error boundary → Exception path

**When to use:** API call failures, validation errors, timeout exceptions.

```json
{
  "shapes": [
    {"id": "task_risky", "type": "bpmnActivity", "activityType": "task",
     "boundingBox": {"x": 300, "y": 180, "w": 160, "h": 80}, "text": "API Call",
     "style": {"fill": {"type": "color", "color": "#BBDEFB"}}},
    {"id": "error_catch", "type": "bpmnEvent", "eventGroup": "intermediate", "eventType": "error",
     "boundingBox": {"x": 420, "y": 240, "w": 40, "h": 40}, "text": "Error",
     "style": {"fill": {"type": "color", "color": "#FFCDD2"}}},
    {"id": "task_handle", "type": "bpmnActivity", "activityType": "task",
     "boundingBox": {"x": 520, "y": 300, "w": 160, "h": 80}, "text": "Log Error & Notify",
     "style": {"fill": {"type": "color", "color": "#FFCDD2"}}}
  ],
  "lines": [
    {"id": "l_error", "lineType": "elbow",
     "endpoint1": {"type": "shapeEndpoint", "style": "none", "shapeId": "error_catch"},
     "endpoint2": {"type": "shapeEndpoint", "style": "arrow", "shapeId": "task_handle"},
     "stroke": {"color": "#F44336", "width": 2, "style": "dashed"}}
  ]
}
```

**Note:** Position the error event on the bottom edge of the task it's attached to.

---

## Pattern 6: Approval Loop

Submit → Review → Approved? → Yes: Continue / No: Revise → Re-submit

**When to use:** Document approvals, budget sign-offs, any iterative review.

**Routing rule:** Gateway exits RIGHT (approved) and BOTTOM (rejected). Loop-back uses dashed line, exits LEFT from revise task, enters BOTTOM of submit task (different from forward-flow entry at LEFT).

```json
{
  "shapes": [
    {"id": "task_submit", "type": "bpmnActivity", "activityType": "task",
     "boundingBox": {"x": 220, "y": 180, "w": 160, "h": 80}, "text": "Submit Request"},
    {"id": "task_review", "type": "bpmnActivity", "activityType": "task",
     "boundingBox": {"x": 440, "y": 180, "w": 160, "h": 80}, "text": "Review Request"},
    {"id": "gw_approved", "type": "bpmnGateway", "gatewayType": "exclusive",
     "boundingBox": {"x": 660, "y": 190, "w": 60, "h": 60}},
    {"id": "task_revise", "type": "bpmnActivity", "activityType": "task",
     "boundingBox": {"x": 440, "y": 340, "w": 160, "h": 80}, "text": "Revise Request"}
  ],
  "lines": [
    {"id": "l_reject", "lineType": "elbow",
     "endpoint1": {"type": "shapeEndpoint", "style": "none", "shapeId": "gw_approved",
                   "position": {"x": 0.5, "y": 1}},
     "endpoint2": {"type": "shapeEndpoint", "style": "arrow", "shapeId": "task_revise",
                   "position": {"x": 1, "y": 0.5}},
     "stroke": {"color": "#333333", "width": 2, "style": "solid"},
     "text": [{"text": "No", "position": 0.3, "side": "top"}]},
    {"id": "l_resubmit", "lineType": "elbow",
     "endpoint1": {"type": "shapeEndpoint", "style": "none", "shapeId": "task_revise",
                   "position": {"x": 0, "y": 0.5}},
     "endpoint2": {"type": "shapeEndpoint", "style": "arrow", "shapeId": "task_submit",
                   "position": {"x": 0.5, "y": 1}},
     "stroke": {"color": "#333333", "width": 2, "style": "dashed"}}
  ]
}
```

---

## Pattern 7: Escalation with Timer

Task → Timer expires → Escalate to Manager

**When to use:** SLA breaches, unresponsive assignments, overdue reviews.

```json
{
  "shapes": [
    {"id": "task_assign", "type": "bpmnActivity", "activityType": "task",
     "boundingBox": {"x": 300, "y": 180, "w": 160, "h": 80}, "text": "Process Request"},
    {"id": "timer_sla", "type": "bpmnEvent", "eventGroup": "intermediate", "eventType": "timer",
     "boundingBox": {"x": 420, "y": 240, "w": 40, "h": 40}, "text": "SLA 24h",
     "style": {"fill": {"type": "color", "color": "#FFE0B2"}}},
    {"id": "task_escalate", "type": "bpmnActivity", "activityType": "task",
     "boundingBox": {"x": 520, "y": 300, "w": 160, "h": 80}, "text": "Escalate to Manager",
     "style": {"fill": {"type": "color", "color": "#FFE0B2"}}}
  ],
  "lines": [
    {"id": "l_escalate", "lineType": "elbow",
     "endpoint1": {"type": "shapeEndpoint", "style": "none", "shapeId": "timer_sla"},
     "endpoint2": {"type": "shapeEndpoint", "style": "arrow", "shapeId": "task_escalate"},
     "stroke": {"color": "#FF9800", "width": 2, "style": "dashed"}}
  ]
}
```

---

## Pattern 8: Multi-Lane Pool (Role-Based Process)

Complete pool setup with 3 roles.

```json
{
  "id": "pool_main",
  "type": "bpmnPool",
  "title": "Business Process",
  "vertical": false,
  "boundingBox": {"x": 50, "y": 50, "w": 1400, "h": 600},
  "lanes": [
    {"title": "Customer",   "width": 200, "laneFill": "#E3F2FD"},
    {"title": "Staff",      "width": 200, "laneFill": "#F3E5F5"},
    {"title": "Management", "width": 200, "laneFill": "#E8F5E9"}
  ]
}
```

**Placing shapes in lanes:**
- Customer lane: y range 50 → 250 (center shapes at y ≈ 110)
- Staff lane: y range 250 → 450 (center shapes at y ≈ 310)
- Management lane: y range 450 → 650 (center shapes at y ≈ 510)

Adjust these based on actual lane heights and the 50px title bar.
