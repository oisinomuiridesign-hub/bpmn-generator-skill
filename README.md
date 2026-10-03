# BPMN Generator: Claude Skill

Turns a plain-language process description, or an SOP, handbook or procedure document, into a **BPMN 2.0 swim-lane diagram in LucidChart**.

The skill gives Claude fixed rules so it doesn't make up its own notation. It covers BPMN element types, grid-based layout, task colours that match their lane, connector routing, and labels on every gateway branch.

Follows the open [Agent Skills](https://agentskills.io) format, so it works in Claude Code, claude.ai, Claude Desktop and other agents that support skills.

## Requirements

You need the **Lucid / LucidChart connector** enabled in Claude. The skill calls `lucid_create_diagram_from_specification`.

## Install

**Skills CLI (Claude Code, Codex, Cursor, and other agents)**

Install for all your projects:

```bash
npx skills add oisinomuiridesign-hub/bpmn-generator-skill -g
```

Or install for the current project only:

```bash
npx skills add oisinomuiridesign-hub/bpmn-generator-skill
```

Needs Node.js 18+. The CLI asks which agents to install to.

**claude.ai / Claude Desktop**

1. Download `bpmn-generator.zip` from the [latest release](https://github.com/oisinomuiridesign-hub/bpmn-generator-skill/releases/latest).
2. Go to **Settings → Capabilities → Skills → Upload skill** and pick the zip.

**Manual (Claude Code)**

```bash
git clone https://github.com/oisinomuiridesign-hub/bpmn-generator-skill.git ~/.claude/skills/bpmn-generator
```

## Use

Ask in plain language:

- "Map our customer onboarding process"
- "Turn this SOP into a BPMN diagram" (with the doc attached)
- "Make a swim-lane diagram of how a refund gets approved"

If the input is thin, Claude asks a few targeted questions first. Then it builds the diagram in your LucidChart account.

## What's inside

```
SKILL.md                     # Entry point and workflow
preferences/preference.md    # Default style and behaviour (edit to customise)
references/
├── bpmn-elements.md         # Mapping from BPMN elements to LucidChart types, plus colour palette
├── layout-strategy.md       # Grid, spacing, swim lanes, routing rules
├── patterns.md              # Reusable JSON patterns: approval, parallel, loop-back and more
└── lucidchart-api.md        # Standard Import JSON format and known pitfalls
```

## License

MIT
