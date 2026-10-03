---
name: bpmn-generator
description: Generate professional BPMN 2.0 business process maps in LucidChart from plain-language descriptions or uploaded documents. Use this skill whenever the user mentions business process maps, BPMN diagrams, process flows, workflow diagrams, swim lane process maps, or wants to visualize how a business process works. Also trigger when the user uploads SOPs, handbooks, procedure documents, or workflow descriptions and wants them turned into visual diagrams. Trigger for phrases like "map this process", "create a process diagram", "visualize this workflow", "turn this into a BPMN", "draw the process flow", or "make a swim lane diagram". Even if the user just says "process map" or "flow diagram" without mentioning BPMN specifically, use this skill.
---

# BPMN Generator

Generate professional BPMN 2.0 business process maps directly in the user's LucidChart account using the `lucid_create_diagram_from_specification` tool.

## Activation

When this skill triggers, present EXACTLY this:

---

# BPMN Process Map Generator

I'll turn your business process into a professional BPMN 2.0 diagram in LucidChart.

**Two ways to start:**

1. **Describe it** — Tell me what process you want mapped. What triggers it, what steps happen, who's involved, and what's the end result?
2. **Paste or upload a document** — Share an SOP, handbook, procedure guide, or workflow description and I'll extract the process and map it.

---

## After Receiving Input

### Path A: From Description
1. Parse input for: steps, decisions, roles/participants, triggers, outcomes
2. If thin (just a name), ask 3-4 targeted questions using the ask_user_input tool:
   - What triggers this process? (start event)
   - What are the main steps and who does each?
   - What decisions/branches exist?
   - What marks it complete? (end event)
3. Generate the BPMN diagram

### Path B: From Document
1. Read and extract: sequential steps, roles, decisions, triggers, outcomes
2. Present summary: "I found X steps across Y roles with Z decision points. Ready to generate?"
3. On confirmation, generate the diagram

## How to Build the Diagram

Read these reference files BEFORE generating:

- `references/bpmn-elements.md` — Complete BPMN-to-LucidChart type mapping
- `references/layout-strategy.md` — Positioning, spacing, and swim lane setup
- `references/patterns.md` — Common reusable patterns (approval, escalation, parallel, error)
- `references/lucidchart-api.md` — Standard Import JSON format and gotchas

## Generation Steps

### 1. Identify Elements
Map every process element to a BPMN type:
- Start/end → `bpmnEvent`
- Tasks → `bpmnActivity`
- Decisions → `bpmnGateway`
- Roles → `bpmnPool` with lanes
- Waits → intermediate timer events
- Errors → error events

### 2. Calculate Layout
Use grid-based positioning (see `references/layout-strategy.md`):
- Left-to-right flow
- 220px column spacing, 40px minimum gap between shapes
- Center shapes within swim lanes
- Gateway branches fork to adjacent rows

### 3. Route Connections (CRITICAL)
Apply the routing rules from `references/layout-strategy.md`:
- **Guide auto-routing via placement:** Position target shapes in different directions from multi-output shapes (right, above, below) so LucidChart's Auto Line Routing naturally picks distinct exit sides. Do NOT use explicit endpoint positions — let auto-routing handle it.
- **Only exception:** Loop-back connections (e.g., retry → previous task) may need explicit positions if auto-routing can't infer the path.
- **No overlap:** Routes must not cross through shapes, text, or annotations. Leave clear corridors between columns.
- **Annotations in clear space:** Place annotations above/below flow paths, never straddling connections. At least 30px from nearest route.

### 4. Build JSON
Assemble the Standard Import JSON with:
- All shapes with unique IDs, types, boundingBoxes, and text
- All connections as elbow lines with arrows and explicit endpoint positions where needed
- Decision branch labels on gateway outputs (using distinct exit sides per Rule 1)
- Pool/lane structure wrapping role-based shapes
- Annotations positioned in clear corridors with dashed straight connectors

### 4. Call LucidChart
Use `lucid_create_diagram_from_specification` with:
- `product`: "lucidchart"
- `title`: The process name
- `standard_import_json`: The complete JSON

### 5. Present Result
Share the LucidChart URL:
"Here's your BPMN diagram: [link]. Open it in LucidChart to edit, refine, or share."

Offer to:
- Add more detail or steps
- Adjust layout or colors
- Add annotations or data objects
- Create a second linked diagram for sub-processes

## Error Handling
- If LucidChart tool unavailable → use `lucid_create_diagram_from_description` as AI-layout fallback
- If >50 shapes → suggest splitting into sub-process diagrams
- If roles unclear → default to single-lane flowchart, note lanes can be added later
