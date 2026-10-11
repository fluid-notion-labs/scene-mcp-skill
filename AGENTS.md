# scene-mcp-skill

Skill repo for using the scene-mcp MCP server. Skill content lives under
`skills/scene-mcp/` (SKILL.md + references/) following the claude-skills
convention. Keep SKILL.md a lean entry point; push detail into references/.

- The source of truth for the tool surface is the scene-mcp repo
  (`nursery/scene-mcp`): `crates/scene-mcp/src/server.rs` and
  `docs/architecture.md`. Update the skill when the server changes.
- Commit after every content change. Push after each logical unit.
