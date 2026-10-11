# scene-mcp skill

A Claude skill for building 3D scenes, architectural models, and diagrams
through [scene-mcp](https://github.com/fluid-notion-labs/scene-mcp) — an MCP
server that exposes a parametric 3D scene graph (groups, primitives with real
dimensions, part metadata, materials) with a live browser viewer the agent can
observe and debug through.

## Structure

Follows the [claude-skills](https://github.com/Jeffallan/claude-skills)
convention:

```
skills/scene-mcp/
  SKILL.md              — entry point: workflow, rules, reference index
  references/
    tools.md            — full tool surface with parameters
    patterns.md         — diagram/architecture authoring patterns
    viewer.md           — live sync, logs, photos, camera poses
    generation.md       — scene-gen: photos → massing models, scale normalization
    materials-bom.md    — material vocabulary, part info, BOM aggregation
```

## Install

### Claude Code / Claude Desktop

Copy or symlink the skill folder into your skills directory:

```bash
ln -s "$(pwd)/skills/scene-mcp" ~/.claude/skills/scene-mcp
```

### pi

Install the scene-mcp MCP server (see the scene-mcp repo README, "MCP
setup"), then make the skill available. The skill's workflow assumes the
MCP tools are present as `mcp__scene-mcp__*`.

## Prerequisites

- The scene-mcp server connected over MCP (stdio).
- Optional but recommended: `scene-api` running (`cargo run -p scene-api`)
  for the live viewer, photo round-trip, and log relay.
- Optional: `scene-gen` built (`cargo build -p scene-gen`) and API keys in
  `env/.env` for photo-to-massing generation.
