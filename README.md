# OpenCode MCP Registry

**OpenCode skill** — register new MCP (Model Context Protocol) servers in OpenCode configuration.

When an OpenCode agent needs to install and register an MCP server, this skill guides it through:
- Installing the server (pre-built binary, pip, npm, Docker)
- Configuring `~/.config/opencode/opencode.json`
- Handling auth tokens and environment variables
- Testing the connection

## Install

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/LDLZM/opencode-mcp-registry ~/.claude/skills/opencode-mcp-registry
```

Then restart OpenCode. The skill auto-loads when you ask to install/register an MCP server.

## Quick Start

Ask your OpenCode agent:
> Install the GitHub MCP server

## Supported MCP Types

| Type | Example |
|------|---------|
| Pre-built binary | Go/Rust MCP servers from GitHub Releases |
| Python (pip/uv) | `uv pip install markitdown-mcp` |
| Node.js (npx) | `npx -y @scope/package@latest` |
| Docker | `docker run ghcr.io/org/server` |
| Remote HTTP | HTTP-based MCP endpoints |

## License

MIT