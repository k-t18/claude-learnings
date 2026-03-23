## Prerequisites

- Claude Code installed (`npm install -g @anthropic-ai/claude-code` or via the Claude desktop app)
- Node.js >= 18
- A Figma account with API access
- A Figma Personal Access Token (PAT)

---

## Step 1 — Generate a Figma Personal Access Token

1.  Log in to [figma.com](https://figma.com)
2.  Go to **Account Settings** → **Security** → **Personal access tokens**
3.  Click **Generate new token**, give it a name (e.g., `claude-mcp`)
4.  Copy the token — you will not be able to view it again

---

## Step 2 — Install the Figma MCP Server

The official Figma MCP server is available as an npm package:

```bash
npm install -g @figma/mcp-server
```

> **Alternative (npx, no global install):** You can use `npx @figma/mcp-server` directly in the config below — Claude Code will invoke it on demand.

---

## Step 3 — Configure Claude Code to Use the MCP Server

Claude Code reads MCP server configurations from a `settings.json` file. You can configure it at two scopes:

| Scope             | File location                       | When to use                                     |
| ----------------- | ----------------------------------- | ----------------------------------------------- |
| **User (global)** | `~/.claude/settings.json`           | Personal machine, all projects                  |
| **Project**       | `.claude/settings.json` (repo root) | Shared team config, checked into source control |

### Add the Figma MCP block

Open (or create) the relevant `settings.json` and add:

```json
{
  "mcpServers": {
    "figma": {
      "command": "npx",
      "args": ["@figma/mcp-server"],
      "env": {
        "FIGMA_ACCESS_TOKEN": "<your-figma-pat>"
      }
    }
  }
}
```

> **Security note:** Never commit a real token to source control. For project-level configs, use an environment variable reference or a `.env` approach and add `.claude/settings.local.json` to
> `.gitignore`.

#### Recommended: token via environment variable

Keep your PAT out of any config file by exporting it in your shell profile:

```bash
# ~/.bashrc or ~/.zshrc
export FIGMA_ACCESS_TOKEN="your-token-here"
```

Then reference it in settings without hardcoding the value:

```json
{
  "mcpServers": {
    "figma": {
      "command": "npx",
      "args": ["@figma/mcp-server"],
      "env": {
        "FIGMA_ACCESS_TOKEN": "${FIGMA_ACCESS_TOKEN}"
      }
    }
  }
}
```

---

## Step 4 — Verify the Connection

Start Claude Code in your project directory:

```bash
claude
```

Run the following prompt to confirm the MCP server is live:

```
/mcp
```

You should see `figma` listed as a connected server. If it's missing, see the Troubleshooting section below.

---

## Step 5 — Usage

Once connected, Claude can answer questions about any Figma file you share by URL.

### Example prompts

**Inspect a component:**

```
Using the Figma MCP, fetch the component named "Button/Primary" from this file:
https://www.figma.com/file/<file-id>/<file-name>
```

**Generate code from a design:**

```
Look at the "Card" component in https://www.figma.com/file/<file-id>/<file-name>
and generate a React + Tailwind implementation that matches its layout and spacing.
```

**Audit design tokens:**

```
List all color styles defined in https://www.figma.com/file/<file-id>/<file-name>
and output them as CSS custom properties.
```

> **Tip:** You can paste a Figma share link directly — Claude will extract the file ID automatically via the MCP server.

---

## Troubleshooting

| Symptom                       | Likely cause              | Fix                                                                  |
| ----------------------------- | ------------------------- | -------------------------------------------------------------------- |
| `figma` not shown in `/mcp`   | Config not picked up      | Confirm `settings.json` path; restart Claude Code                    |
| `401 Unauthorized` from Figma | Invalid or expired PAT    | Regenerate token in Figma → Account Settings                         |
| `npx: command not found`      | Node.js not in PATH       | Ensure Node >= 18 is installed; use absolute path to `npx`           |
| File fetch returns empty      | Wrong file ID / no access | Check the share URL; ensure the token has read access to the file    |
| Slow first response           | npx cold-start            | Install globally (`npm i -g @figma/mcp-server`) to skip the download |

---

## Security Best Practices

- Store your PAT in a password manager or secrets vault (e.g., 1Password, AWS Secrets Manager)
- Rotate tokens periodically; Figma supports multiple tokens per account
- Use project-scoped tokens with the minimum required permissions when possible
- Add `settings.local.json` to `.gitignore` if it contains tokens

---

## References

- [Claude Code MCP documentation](https://docs.anthropic.com/en/docs/claude-code/mcp)
- [Figma REST API — Authentication](https://www.figma.com/developers/api#access-tokens)
- [@figma/mcp-server on npm](https://www.npmjs.com/package/@figma/mcp-server)
- [Model Context Protocol spec](https://modelcontextprotocol.io)

---
