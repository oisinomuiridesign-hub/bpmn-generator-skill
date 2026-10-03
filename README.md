# BPMN Generator — Claude Skill

Turn a plain-language process description (or an SOP / handbook / procedure doc) into a professional **BPMN 2.0 swim-lane diagram in LucidChart**.

The skill encodes the rules so Claude doesn't improvise notation: correct BPMN element types, grid-based layout, lane-matched colours, clean connector routing, and labelled gateway branches.

## Requirements

- Claude (Claude Code, Claude Desktop, or claude.ai) with the **Lucid / LucidChart connector** enabled — the skill calls `lucid_create_diagram_from_specification`.

## Install

### Claude Code (plugin marketplace)

```
/plugin marketplace add oisinomuiridesign-hub/bpmn-generator-skill
/plugin install bpmn@oo-bpmn-skills
```

### Manual (Claude Code)

Copy the skill folder into your skills directory:

```bash
git clone https://github.com/oisinomuiridesign-hub/bpmn-generator-skill.git
cp -r bpmn-generator-skill/plugins/bpmn/skills/bpmn ~/.claude/skills/bpmn-generator
```

### claude.ai / Claude Desktop

Zip `plugins/bpmn/skills/bpmn/` and upload it under **Settings → Capabilities → Skills**.

## Use

Just ask:

- "Map our customer onboarding process"
- "Turn this SOP into a BPMN diagram" (attach the doc)
- "Make a swim-lane diagram of how a refund gets approved"

Claude asks a few targeted questions if the input is thin, then builds the diagram in your LucidChart account.

## What's inside

```
plugins/bpmn/skills/bpmn/
├── SKILL.md                     # Entry point & workflow
├── preferences/preference.md    # Default style & behaviour
└── references/
    ├── bpmn-elements.md         # BPMN → LucidChart type mapping + colour palette
    ├── layout-strategy.md       # Grid, spacing, swim lanes, routing rules
    ├── patterns.md              # Reusable JSON patterns (approval, parallel, loop-back…)
    └── lucidchart-api.md        # Standard Import JSON format & gotchas
```

Tweak `preferences/preference.md` to change default colours, flow direction, or title format.

## License

MIT
