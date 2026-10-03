# LucidChart Standard Import API Reference

How to call LucidChart's `lucid_create_diagram_from_specification` tool correctly.

---

## Tool Call

```
lucid_create_diagram_from_specification(
  product = "lucidchart",
  title = "Process Name - BPMN",
  standard_import_json = <JSON string>
)
```

## JSON Structure

```json
{
  "version": 1,
  "pages": [
    {
      "id": "page1",
      "title": "Process Name",
      "shapes": [...],
      "lines": [...]
    }
  ]
}
```

The `standard_import_json` parameter takes a JSON STRING, not an object. Use `JSON.stringify()` or equivalent.

---

## Shape Properties

Every shape needs at minimum:

```json
{
  "id": "unique_string",
  "type": "bpmnActivity",
  "boundingBox": {"x": 100, "y": 100, "w": 160, "h": 80}
}
```

### Optional Properties

| Property | Type | Notes |
|----------|------|-------|
| `text` | string | Display text on shape. NOT supported on containers, `or`, `summingJunction`, `hotspot` |
| `style` | object | `{"fill": {"type": "color", "color": "#hex"}, "stroke": {"color": "#hex", "width": 1, "style": "solid"}}` |
| `opacity` | number | 0 (transparent) to 100 (opaque) |
| `note` | string | Additional note (for containers, shown above) |

### BPMN-Specific Properties

**bpmnEvent:** `eventGroup` (required: "start"/"intermediate"/"end"), `eventType` (optional)
**bpmnActivity:** `activityType` (required: "task"/"transaction"/"eventSubProcess"/"callActivity")
**bpmnGateway:** `gatewayType` (optional: "exclusive"/"parallel"/"inclusive"/"eventBased"/"none")
**bpmnPool:** `title`, `lanes[]`, `vertical` (all required)
**bpmnDataObject:** `dataType` (optional: "none"/"collection"/"input"/"output")

---

## Line Properties

```json
{
  "id": "unique_string",
  "lineType": "elbow",
  "endpoint1": {
    "type": "shapeEndpoint",
    "style": "none",
    "shapeId": "source_shape_id"
  },
  "endpoint2": {
    "type": "shapeEndpoint",
    "style": "arrow",
    "shapeId": "target_shape_id"
  },
  "stroke": {
    "color": "#333333",
    "width": 2,
    "style": "solid"
  },
  "text": [
    {"text": "Label", "position": 0.3, "side": "top"}
  ]
}
```

### Line Types
- `"elbow"` — Standard BPMN connector (USE THIS for all BPMN diagrams)
- `"straight"` — Direct line
- `"curved"` — Bezier curve

### Endpoint Styles
- `"none"` — No decoration (use on source/start end)
- `"arrow"` — Standard arrowhead (use on target/destination end)

### IMPORTANT: Endpoint Position Rules
- Position `{x, y}` on shapeEndpoints is optional
- If you specify position on ONE endpoint, you MUST specify it on BOTH
- Positions are RELATIVE to the shape's boundingBox: `{x: 0, y: 0}` = top-left corner, `{x: 1, y: 1}` = bottom-right corner

**Prefer smart routing (omit positions) whenever possible.** LucidChart's Auto Line Routing algorithm chooses optimal exit/entry points based on shape positions and avoids crossings automatically. This produces cleaner, more consistent results than manual positioning.

**When to use explicit positions (rare):**
- Loop-back connections where auto-routing can't infer the intended path
- When auto-routing consistently produces a bad result despite good shape placement

**Side-to-position mapping (relative coordinates):**

| Side | Position | Use for |
|------|----------|---------|
| RIGHT center | `{"x": 1, "y": 0.5}` | Main forward flow |
| BOTTOM center | `{"x": 0.5, "y": 1}` | Branch down / cross-lane down |
| TOP center | `{"x": 0.5, "y": 0}` | Branch up / cross-lane up |
| LEFT center | `{"x": 0, "y": 0.5}` | Loop-back entry / incoming flow |

**Example — Gateway with 3 distinct exit points:**

```json
// Main path exits RIGHT
{"id": "l_main", "lineType": "elbow",
 "endpoint1": {"type": "shapeEndpoint", "style": "none", "shapeId": "gw_1",
               "position": {"x": 1, "y": 0.5}},
 "endpoint2": {"type": "shapeEndpoint", "style": "arrow", "shapeId": "task_main",
               "position": {"x": 0, "y": 0.5}},
 "text": [{"text": "Qualified", "position": 0.3, "side": "top"}]},

// Alternate path exits TOP
{"id": "l_alt_up", "lineType": "elbow",
 "endpoint1": {"type": "shapeEndpoint", "style": "none", "shapeId": "gw_1",
               "position": {"x": 0.5, "y": 0}},
 "endpoint2": {"type": "shapeEndpoint", "style": "arrow", "shapeId": "end_rejected",
               "position": {"x": 0.5, "y": 1}},
 "text": [{"text": "Not a fit", "position": 0.3, "side": "top"}]},

// Alternate path exits BOTTOM
{"id": "l_alt_down", "lineType": "elbow",
 "endpoint1": {"type": "shapeEndpoint", "style": "none", "shapeId": "gw_1",
               "position": {"x": 0.5, "y": 1}},
 "endpoint2": {"type": "shapeEndpoint", "style": "arrow", "shapeId": "task_alt",
               "position": {"x": 0, "y": 0.5}},
 "text": [{"text": "Needs review", "position": 0.3, "side": "top"}]}
```

**Example — Task with main flow + side-effect (cross-lane webhook):**

```json
// Main flow exits RIGHT
{"id": "l_main", "lineType": "elbow",
 "endpoint1": {"type": "shapeEndpoint", "style": "none", "shapeId": "task_log",
               "position": {"x": 1, "y": 0.5}},
 "endpoint2": {"type": "shapeEndpoint", "style": "arrow", "shapeId": "gw_next",
               "position": {"x": 0, "y": 0.5}}},

// Webhook side-effect exits TOP (to n8n lane above)
{"id": "l_webhook", "lineType": "elbow",
 "endpoint1": {"type": "shapeEndpoint", "style": "none", "shapeId": "task_log",
               "position": {"x": 0.5, "y": 0}},
 "endpoint2": {"type": "shapeEndpoint", "style": "arrow", "shapeId": "task_increment",
               "position": {"x": 0.5, "y": 1}}}
```

**Default: OMIT positions on ALL endpoints.** This is the standard approach for BPMN diagrams:
- Auto-routing handles single and multi-output shapes cleanly when targets are well-positioned
- Sequential same-lane flow, cross-lane connections, gateway branches — all work automatically
- Annotation dashed connectors always use `"lineType": "straight"` with no positions
- The only time to add explicit positions is for loop-back lines where auto-routing produces a bad path

### Line Text
- `position`: 0 to 1 (where along the line)
- `side`: "top", "middle", or "bottom"
- Use `position: 0.3` and `side: "top"` for gateway branch labels

---

## Pool and Lane Rules

### bpmnPool

```json
{
  "id": "pool_1",
  "type": "bpmnPool",
  "title": "Organization",
  "vertical": false,
  "boundingBox": {"x": 50, "y": 50, "w": 1400, "h": 600},
  "lanes": [
    {"title": "Role A", "width": 200, "laneFill": "#E3F2FD"},
    {"title": "Role B", "width": 200, "laneFill": "#F3E5F5"},
    {"title": "Role C", "width": 200, "laneFill": "#E8F5E9"}
  ]
}
```

**CRITICAL GOTCHAS:**

1. `vertical: false` = horizontal lanes (rows, top to bottom). This is standard BPMN.
2. `vertical: true` = vertical lanes (columns, left to right). Rarely used in BPMN.
3. Each lane's `"width"` means the HEIGHT of that lane row when `vertical: false`
4. `title`, `width`, and `laneFill` are ALL REQUIRED for each lane
5. **Sum of all lane widths MUST equal `boundingBox.h`**
6. Shapes placed within the pool's boundingBox coordinates will visually appear inside the lanes

### Lane Color Palette

Use these for consistent, professional look:

| Lane | Fill Color | Notes |
|------|-----------|-------|
| Lane 1 | `#E3F2FD` | Light blue |
| Lane 2 | `#F3E5F5` | Light purple |
| Lane 3 | `#E8F5E9` | Light green |
| Lane 4 | `#FFF8E1` | Light amber |
| Lane 5 | `#FCE4EC` | Light pink |
| Lane 6 | `#E0F2F1` | Light teal |

---

## Common Mistakes

### 1. Lane width sum mismatch
**Wrong:** Pool height 600, lanes: [200, 200, 150] → sum = 550
**Right:** Pool height 600, lanes: [200, 200, 200] → sum = 600

### 2. Shapes outside pool bounds
All shapes meant to be in a pool must have coordinates WITHIN the pool's boundingBox.

### 3. Missing required BPMN properties
- `bpmnEvent` without `eventGroup` → fails
- `bpmnActivity` without `activityType` → fails
- `bpmnPool` without `lanes` → fails

### 4. Text on containers
Containers (including pools) use `title` or `note`, NOT `text`.

### 5. Emoji in text
Text values must NOT contain emoji characters — they render as black boxes in LucidChart.

### 6. Duplicate IDs
Every shape and line must have a globally unique `id` string.

---

## Fallback: Description-Based Generation

If `lucid_create_diagram_from_specification` is unavailable, fall back to:

```
lucid_create_diagram_from_description(
  title = "Process Name",
  user_prompt = "Create a BPMN 2.0 diagram showing...",
  diagram_type = "Flowchart"
)
```

This uses LucidChart's AI to generate the diagram. Less precise but always available.

---

## Size Limits

- Maximum document.json size: 2MB
- For large diagrams (>50 shapes), minimize whitespace in JSON
- Consider splitting into sub-process diagrams if exceeding limits
