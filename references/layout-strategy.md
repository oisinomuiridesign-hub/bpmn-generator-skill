# Layout Strategy

How to position BPMN elements in LucidChart Standard Import JSON for clean, readable diagrams.

---

## Grid System

Use a column-and-row grid. All shapes snap to grid intersections.

### Spacing Constants

| Constant | Value | Purpose |
|----------|-------|---------|
| `COL_SPACING` | 220px | Horizontal gap between shape centers |
| `ROW_SPACING` | 140px | Vertical gap between parallel branches |
| `LANE_HEIGHT` | 200px | Height per swim lane |
| `LANE_HEADER` | 50px | Title bar height for pools |
| `POOL_LEFT_MARGIN` | 260px | Left padding inside pool — must clear the vertical pool title bar (~30px) AND the horizontal lane label text (~200px rendered by LucidChart). 80px is NOT enough; shapes will overlap lane titles. |
| `SHAPE_GAP` | 40px | Minimum gap between any two shapes |

### Shape Sizes

| Element | Width | Height |
|---------|-------|--------|
| Events (start, end, intermediate) | 40 | 40 |
| Tasks (bpmnActivity) | 160 | 80 |
| Gateways | 60 | 60 |
| Data Objects | 40 | 50 |
| Data Stores | 60 | 50 |
| Text Annotations | 200 | 60 |
| Black Box Pool | full width | 60 |

---

## Flow Direction

Always LEFT → RIGHT. This is standard BPMN convention.

### Column Assignment

1. Start event → Column 0
2. Each subsequent step → next column
3. Gateway → its own column
4. Gateway branches → same column for the gateway, then branches spread to parallel rows
5. Join/merge gateway → column after longest branch
6. End event → last column

### Example Column Layout

```
Col 0     Col 1        Col 2        Col 3        Col 4     Col 5
[Start] → [Task A] → [Gateway] → [Task B] → [Join] → [End]
                         ↓                      ↑
                      [Task C] ────────────────→┘
```

### Calculating X Positions

```
x = POOL_LEFT_MARGIN + (column * COL_SPACING)
```

For shapes within a pool, add the pool's x offset:
```
shape_x = pool_x + POOL_LEFT_MARGIN + (column * COL_SPACING)
```

### Calculating Y Positions (within lanes)

Center shapes vertically within their lane:
```
lane_top = pool_y + LANE_HEADER + (lane_index * LANE_HEIGHT)
shape_y = lane_top + (LANE_HEIGHT - shape_height) / 2
```

For branching paths, offset from center:
```
main_path_y = lane_center
upper_branch_y = lane_center - ROW_SPACING / 2
lower_branch_y = lane_center + ROW_SPACING / 2
```

---

## Pool and Lane Setup

### Single Pool with Horizontal Lanes (most common)

```json
{
  "id": "pool_main",
  "type": "bpmnPool",
  "title": "Process Name",
  "vertical": false,
  "boundingBox": {
    "x": 50,
    "y": 50,
    "w": POOL_LEFT_MARGIN + (num_columns * COL_SPACING) + 60,
    "h": LANE_HEADER + (num_lanes * LANE_HEIGHT)
  },
  "lanes": [
    {"title": "Role A", "width": 200, "laneFill": "#E3F2FD"},
    {"title": "Role B", "width": 200, "laneFill": "#F3E5F5"},
    {"title": "Role C", "width": 200, "laneFill": "#E8F5E9"}
  ]
}
```

**CRITICAL RULES:**
- `vertical: false` means lanes are horizontal rows (standard BPMN)
- Each lane's `"width"` = the lane's VERTICAL height (confusing but correct)
- Sum of all lane widths MUST equal `boundingBox.h - LANE_HEADER` (if titleBar height is in the pool's header, check the exact spec)
- Actually: sum of lane widths = `boundingBox.h` (the pool handles the title bar separately)

### Sizing the Pool

Calculate pool dimensions based on content:

```
pool_width = POOL_LEFT_MARGIN + (num_columns * COL_SPACING) + 60
pool_height = LANE_HEADER + (num_lanes * LANE_HEIGHT)
```

Or if no header bar is separate:
```
pool_height = num_lanes * LANE_HEIGHT
```

Ensure: `sum(lane.width for lane in lanes) == pool_height`

---

## Placing Shapes in Lanes

Each shape belongs in the lane of its responsible role. To place a shape correctly:

1. Determine which lane (role) owns the task
2. Calculate the lane's vertical band:
   - `lane_top = pool_y + LANE_HEADER + sum(previous_lane_heights)`
   - `lane_bottom = lane_top + this_lane_height`
3. Center the shape vertically: `shape_y = lane_top + (lane_height - shape_height) / 2`
4. Set `shape_x` based on column position
5. **Set the task fill color to match the lane fill color** (see `bpmn-elements.md` Color Palette). This is mandatory for tasks — it reinforces lane ownership visually.

### Cross-Lane Connections

When a connection crosses lanes (e.g., Customer submits → Sales reviews):
- Use elbow lines (`"lineType": "elbow"`)
- Let LucidChart route the line automatically (don't specify positions on endpoints)
- Smart lines (no position specified) handle cross-lane routing best

---

## Connection Routing Rules

### CRITICAL: No overlapping routes

Every connection (route) must have a clear, unobstructed path. Follow these rules strictly:

#### Rule 1: Distinct exit points via shape placement

LucidChart's Auto Line Routing automatically chooses the best exit/entry side based on the **relative position** of connected shapes. Instead of forcing exit points with endpoint positions, **position your shapes so auto-routing naturally picks distinct sides.**

**Placement strategy for multi-output shapes:**

| Route type | Place target shape... | Auto-routing will use... |
|-----------|----------------------|--------------------------|
| Main flow (→ next step) | To the RIGHT, same row | RIGHT exit → LEFT entry |
| Branch down (→ lower path) | BELOW and to the right | BOTTOM exit → TOP or LEFT entry |
| Branch up (→ upper path) | ABOVE and to the right | TOP exit → BOTTOM or LEFT entry |
| Cross-lane down | In a LOWER lane, same or next column | BOTTOM exit → TOP entry |
| Cross-lane up | In an UPPER lane, same or next column | TOP exit → BOTTOM entry |
| Side-effect (→ triggered task) | Directly ABOVE or BELOW in another lane | TOP/BOTTOM exit → BOTTOM/TOP entry |

**Gateway target placement for automatic distinct exits:**

```
2 outputs:  Main target to the RIGHT → alternate target BELOW (or ABOVE)
3 outputs:  Main target RIGHT → one target ABOVE → one target BELOW
```

**Task with main flow + cross-lane side-effect:**

```
              [Triggered task in upper lane]   ← place directly above
                                                  (auto-routes via TOP)
[Previous] → [Task] → [Next main step]        ← place to the right
                                                  (auto-routes via RIGHT)
```

**Do NOT use explicit endpoint positions** unless auto-routing fails — see `lucidchart-api.md`.

#### Rule 2: Minimum spacing between elements

| Between | Minimum gap |
|---------|-------------|
| Any two shapes (edge to edge) | 40px |
| Parallel branch shapes (vertical gap) | 60px |
| Annotation and nearest connection path | 30px |
| Annotation and nearest shape | 20px |
| Shape text and connection line | 20px |

To calculate: if a task is at y=200 with h=80 (bottom at y=280), the next shape below must start at y ≥ 320 (280 + 40px gap).

#### Rule 3: No routes crossing through shapes or text

- Position shapes so that elbow-routed lines have clear corridors
- Leave vertical corridors between columns for cross-lane routing
- Annotations must sit ABOVE or BELOW connection paths, never straddling them
- Connection labels (`text` on lines) use `position: 0.2–0.4` and `side: "top"` to avoid overlapping with shapes

#### Rule 4: Annotation placement

Annotations connect to their target shape with dashed straight lines. To avoid clutter:

```
GOOD: Annotation sits in clear space above/below, dashed line goes straight down/up

     [Annotation text here]
           |  (dashed)
     [Target task]

BAD: Annotation overlaps with connection between shapes

     [Task A] ──────→ [Task B]
        [Annotation sitting on top of the arrow]
```

**Placement strategy:**
- Place annotations in the SAME lane as their target, offset vertically toward the lane edge
- If the lane center is occupied by the main flow, place the annotation near the lane top or bottom
- Ensure the dashed connector doesn't cross any flow lines

---

## Gateway Branching Layout

### Exclusive Gateway (Yes/No) — 2 outputs

```
                 (TOP exit)
                 [End: Rejected]

[Task] → (LEFT) <Gateway> (RIGHT) → [Task: Main Path] → [Next]

                 (BOTTOM exit)
                 [Task: Alt Path] → [Rejoin]
```

- Gateway at column N
- Main path (RIGHT exit): same row, column N+1
- Alternate path: BOTTOM exit to row below, or TOP exit to row above
- Each exit uses a different side of the diamond
- Join gateway: column after longest branch

### Exclusive Gateway (3-way) — 3 outputs

```
                 (TOP exit)
                 [End: Path A]

[Task] → (LEFT) <Gateway> (RIGHT) → [Task: Main Path]

                 (BOTTOM exit)
                 [Task: Path B]
```

- RIGHT: main/happy path
- TOP: first alternate (typically end events or short paths)
- BOTTOM: second alternate

### Parallel Gateway (Fork/Join)

```
                 (TOP exit)
                 [Task A in upper lane] ──→
                                            → (TOP enter) <+Join> (RIGHT) → [Next]
[Task] → (LEFT) <+Fork>
                 (BOTTOM exit)             → (BOTTOM enter) ↗
                 [Task B in lower lane] ──→
```

- Fork gateway at column N
- Parallel tasks at column N+1, spread across DIFFERENT LANES
- TOP exit → upper lane task, BOTTOM exit → lower lane task
- Join gateway at column N+2
- Lines entering the join gateway use TOP and BOTTOM sides

### Loop-Back Pattern

For retry loops (e.g., wait & retry contact):

```
     (TOP exit from gateway)
     ↓
     [Timer: Wait]
     ↓ (dashed, goes LEFT then DOWN)
     ↓
     ←←←←←←←←←←←←←←←←←←←
     ↓
     [Target task] (enters TOP or LEFT)
```

- Loop-back line exits gateway from TOP (not RIGHT — RIGHT is the forward path)
- Timer/wait element sits ABOVE the gateway
- Return line uses DASHED style to distinguish from forward flow
- Return line enters the target task from TOP (not LEFT, to avoid overlap with the task's incoming forward-flow line)
- Leave enough vertical space for the return line to route without crossing other shapes

---

## No-Pool Layout (Simple Flowchart)

For processes with a single role or when roles don't matter:

- Skip the pool entirely
- Place shapes directly on the canvas
- Start at `x: 100, y: 200`
- Use `COL_SPACING` for horizontal progression
- Use `ROW_SPACING` for branches

This is simpler but loses the role-based clarity that swim lanes provide.

---

## Layout Checklist

Before generating the final JSON, verify ALL of the following:

### Shapes
- [ ] Every shape has a unique ID
- [ ] Every shape's boundingBox is within its lane boundaries
- [ ] No shapes overlap (minimum 40px gap between any two shapes)
- [ ] Start event is leftmost, end event(s) rightmost
- [ ] Pool width accommodates all columns
- [ ] Lane height sum equals pool height
- [ ] Every task's fill color matches its lane's fill color

### Routing
- [ ] Multi-output shapes have targets placed in different directions (right, above, below) so auto-routing uses distinct sides
- [ ] Elbow lines used for all flow connections (straight only for annotations)
- [ ] No connection line passes through any shape or text
- [ ] Loop-back lines use dashed style
- [ ] Gateway output lines have decision labels
- [ ] No explicit endpoint positions used (except loop-backs where auto-routing fails)

### Annotations
- [ ] Annotations are placed in clear space (not on connection paths)
- [ ] At least 30px gap between annotation and nearest connection route
- [ ] Dashed annotation connectors don't cross flow lines
