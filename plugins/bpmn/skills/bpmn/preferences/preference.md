# BPMN Generator Preferences

## Diagram Style
- Flow direction: Left to right (horizontal)
- Line type: Elbow connectors
- Color coding: Enabled (green start, red end, blue tasks, yellow gateways)
- Lane colors: Alternating pastels from standard palette

## Default Behavior
- Always use swim lanes when 2+ roles are identified
- Always label gateway branches (Yes/No, Approved/Rejected, etc.)
- Default to exclusive gateways unless parallel is explicitly needed
- Include start and end events even if not explicitly mentioned
- Use smart lines (no explicit endpoint positions) for auto-routing

## Output
- Tool: lucid_create_diagram_from_specification (primary)
- Fallback: lucid_create_diagram_from_description
- Product: lucidchart
- Title format: "[Process Name] - BPMN"

## Quality Checks
- Validate all branches reconverge before end
- Verify lane width sum matches pool height
- Confirm no duplicate shape/line IDs
- Ensure all shapes within pool bounds
