---
name: opencode-mcp-registry
description: Register new MCP servers in opencode. Use when the user wants to install, register, or add a new MCP (Model Context Protocol) server to their opencode configuration — including after pip/npm install of an MCP package, or when a user asks to "connect", "integrate", "register", or "set up" an MCP tool in opencode.
---

# OpenCode MCP Registry

Register new MCP (Model Context Protocol) servers in opencode.

## Configuration File Location

**Do NOT search for config files.** The sole MCP configuration file is always at:

```
~/.config/opencode/opencode.json
```

No other locations are used (not `~/.opencode/`, not `~/.local/share/opencode/`, not `~/.config/Claude/`). If the file doesn't exist, create it with:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {}
}
```

## MCP Config Format

OpenCode supports two MCP server types:

### `"type": "local"` — Locally-run MCP servers

For servers started as a local process (stdio transport). Use `"command"` for the executable and `"environment"` for env vars.

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "my-server": {
      "type": "local",
      "command": ["/absolute/path/to/binary", "arg1", "arg2"],
      "environment": {
        "API_KEY": "sk-xxx",
        "DEBUG": "true"
      }
    }
  }
}
```

- `"command"` (required): array of executable + args. Use absolute paths.
- `"environment"`: key-value pairs set as env vars when spawning the process. Use this for auth tokens, API keys, etc.
- `"enabled"`: set to `false` to disable on startup (default: `true`).
- `"timeout"`: request timeout in ms (default: 5000).

### `"type": "remote"` — HTTP-based remote MCP servers

For servers accessed via HTTP URL. Note: compatibility varies — some remote MCP endpoints (e.g., GitHub's `api.githubcopilot.com`) are designed for specific IDEs and may not work with OpenCode. Always test and fall back to `"local"` if needed.

```json
{
  "type": "remote",
  "url": "https://mcp.example.com/mcp",
  "headers": {
    "Authorization": "Bearer <token>"
  }
}
```

- `"url"` (required): the remote MCP endpoint URL.
- `"headers"`: HTTP headers sent with each request.
- `"oauth"`: OAuth config object (with `clientId`, `clientSecret`, `scope`, `redirectUri`).

### Common patterns

**Python MCP (pip/uv):**
```json
{
  "type": "local",
  "command": ["/home/user/.venv/bin/mcp-server-name"]
}
```

**Node.js MCP (npm/npx):**
```json
{
  "type": "local",
  "command": ["npx", "-y", "@scope/package-name@latest"]
}
```

**Docker-based MCP:**
```json
{
  "type": "local",
  "command": ["docker", "run", "--rm", "-i", "image-name:latest"],
  "environment": {
    "API_KEY": "<token>"
  }
}
```

**Pre-built binary from GitHub Releases:**
```json
{
  "type": "local",
  "command": ["/home/user/.local/bin/mcp-server-binary", "stdio"],
  "environment": {
    "API_KEY": "<token>"
  }
}
```

## Anti-Exploration Rules

- **Never** use glob/grep/Task agent to "find the config file" — it's always `~/.config/opencode/opencode.json`
- **Never** search other directories like `~/.local/share/opencode/`, `~/.config/Claude/`, or system paths for MCP settings
- Read the config file directly with `Read`, edit with `Edit` — no exploration needed
- If the user says "install an MCP", install it AND register it in the same turn

## Workflow

### Step 0: Determine installation method

In order of preference:
1. **Pre-built binary** — check GitHub Releases for compiled binaries (common for Go/Rust MCP servers). Download to `~/.local/bin/`.
2. **Package manager** — pip/uv for Python, npm/npx for Node.js.
3. **Docker** — when a Docker image is available and Docker is installed.
4. **Remote HTTP** — try `"type": "remote"` if the server offers an HTTP endpoint. Test immediately; if it fails, fall back to a local method.

### Step 1: Install the MCP server

Install using the appropriate method. Prefer uv for Python packages, npm/npx for Node packages.

**Pre-built binary (GitHub Releases):**
```bash
# Check latest release for the right platform/arch
# Download and install
mkdir -p ~/.local/bin
curl -fsSL "$DOWNLOAD_URL" -o /tmp/mcp-server.tar.gz
tar -xzf /tmp/mcp-server.tar.gz -C /tmp
cp /tmp/<binary-name> ~/.local/bin/<binary-name>
chmod +x ~/.local/bin/<binary-name>
```

**Python:**
```bash
# Create venv if needed
uv venv ~/.venv-mcp-server
source ~/.venv-mcp-server/bin/activate
uv pip install <mcp-package>
# Find the binary
ls ~/.venv-mcp-server/bin/<name>*
```

**Node.js (npx, no install needed):**
```json
{"command": ["npx", "-y", "@scope/package@latest"]}
```

### Step 2: Locate the binary

Find the exact path to the MCP server binary:

```bash
# For Python venv
ls ~/.venv-<name>/bin/<server-name>
# For pre-built
ls ~/.local/bin/<name>
# Or
which <server-name>  # if on PATH
```

Always use **absolute paths** in the config — opencode may not have the same PATH.

### Step 3: Handle auth / environment variables

Many MCP servers require authentication tokens. In OpenCode's `"local"` type, use the `"environment"` key (NOT `"env"`):

```json
{
  "type": "local",
  "command": ["/path/to/binary"],
  "environment": {
    "API_KEY": "sk-xxx"
  }
}
```

For `"remote"` type, use `"headers"`:

```json
{
  "type": "remote",
  "url": "https://...",
  "headers": {
    "Authorization": "Bearer <token>"
  }
}
```

> **Security**: Tokens in `opencode.json` are plaintext. Run `chmod 600 ~/.config/opencode/opencode.json` to restrict access.

### Step 4: Edit opencode.json

Add the entry under `"mcp"`. Read the existing file first, then edit it to add the new server entry alongside any existing ones.

### Step 5: Verify

After editing, read the config back to confirm the JSON is valid and the entry is correct.

### Step 6: Restart opencode

The new MCP server will be available after restarting opencode. The tool names will appear as `<server-name>_<tool-name>` (e.g., `markitdown_convert_to_markdown`).

### Step 7: Test

Test the new tool with a simple input to confirm it works end-to-end.

## Example: Registering markitdown-mcp

```bash
# Install
uv venv ~/.venv-markitdown
source ~/.venv-markitdown/bin/activate
uv pip install markitdown-mcp

# Verify binary exists
ls ~/.venv-markitdown/bin/markitdown-mcp

# Edit ~/.config/opencode/opencode.json to add:
#   "markitdown": {
#     "type": "local",
#     "command": ["/home/cits/.venv-markitdown/bin/markitdown-mcp"]
#   }

# Restart opencode, then test:
# markitdown_convert_to_markdown(uri="file:///tmp/test.html")
```
