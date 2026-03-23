# Figma MCP Connection Approaches: Local SSE vs npm + settings.json

## Overview

There are two ways to connect Claude Code with Figma MCP. Choosing the right one depends on your environment and team setup.

---

## Approach 1 — Local SSE via Figma Desktop (Recommended for Solo Dev)

### Setup
Run this one-time command in your terminal:

```bash
claude mcp add --transport sse figma-local http://127.0.0.1:3845/sse
```

### How it works
- Figma Desktop automatically runs a local MCP server on port `3845`
- Claude Code connects to it via SSE (Server-Sent Events)
- The config is saved to **your machine's** `~/.claude/settings.json` — not the repo

### When to use
- You are working solo on your local machine
- You have Figma Desktop installed and open
- You want the fastest, zero-config setup
- You are doing real-time design-to-code work

### Pros & Cons

| ✅ Pros | ❌ Cons |
|---|---|
| Zero config — no token, no npm | Figma Desktop must be open |
| Works instantly | Config lives on your machine only |
| Figma Desktop handles auth | Teammates must each run the command |
| Great for real-time design work | Doesn't work in CI/CD or headless |

---

## Approach 2 — npm Package + settings.json (Recommended for Teams)

### Setup

**1. Install the Figma MCP server:**
```bash
npm install -g @figma/mcp-server
```

**2. Generate a Figma Personal Access Token:**
- Log in to [figma.com](https://figma.com)
- Go to **Account Settings** → **Security** → **Personal access tokens**
- Click **Generate new token**, copy it

**3. Add config to your repo at `.claude/settings.json`:**
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

**4. Each teammate sets their token as an environment variable:**
```bash
# Add to ~/.zshrc or ~/.bashrc
export FIGMA_ACCESS_TOKEN="your-personal-token-here"
```

### How it works
- The `.claude/settings.json` file is committed to the repo
- When any teammate clones the repo and opens Claude Code, it automatically picks up the config
- Each developer provides their own `FIGMA_ACCESS_TOKEN` via environment variable — no token is ever committed to the repo
- Works without Figma Desktop being open

### When to use
- You are on a team and want a shared, consistent setup
- You want the config version-controlled in the repo
- You are in a CI/CD pipeline or headless environment
- You are on Linux or Windows where Figma Desktop may not expose port `3845`

### Pros & Cons

| ✅ Pros | ❌ Cons |
|---|---|
| Config shared via repo — teammates just clone | Requires a Figma Personal Access Token |
| Works without Figma Desktop | Each dev must set env var manually |
| Works in CI/CD and headless environments | Slightly more setup upfront |
| Cross-platform (Mac, Linux, Windows) | Token must be kept secure |

---

## Side-by-Side Comparison

| | Local SSE (Approach 1) | npm + settings.json (Approach 2) |
|---|---|---|
| **Setup time** | ~30 seconds | ~5 minutes |
| **Config location** | `~/.claude/settings.json` (machine) | `.claude/settings.json` (repo) |
| **Shared with team** | ❌ No — each dev runs command | ✅ Yes — committed to repo |
| **Requires Figma Desktop** | ✅ Yes, must be open | ❌ No |
| **Requires API token** | ❌ No | ✅ Yes |
| **Works in CI/CD** | ❌ No | ✅ Yes |
| **Cross-platform** | ⚠️ Primarily macOS | ✅ Mac, Linux, Windows |
| **Best for** | Solo dev, quick setup | Teams, shared projects |

---

## Quick Decision Guide

```
Are you working solo on your local machine?
  └── Yes → Use Approach 1 (Local SSE)

Are you on a team or want shared config in the repo?
  └── Yes → Use Approach 2 (npm + settings.json)

Do you need it to work without Figma Desktop open?
  └── Yes → Use Approach 2 (npm + settings.json)

Do you need CI/CD or headless support?
  └── Yes → Use Approach 2 (npm + settings.json)

Just want to try it out fast?
  └── Use Approach 1 (Local SSE)
```

---

## Security Notes

- **Never commit a real token** to source control
- Always use `${FIGMA_ACCESS_TOKEN}` in `settings.json` and set the actual value via environment variable
- Add `.claude/settings.local.json` to `.gitignore` if you store any local overrides

---

## References

- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Figma Desktop Downloads](https://www.figma.com/downloads/)
- [Figma Personal Access Tokens](https://www.figma.com/developers/api#access-tokens)
- [Anthropic MCP Overview](https://docs.anthropic.com/en/docs/mcp)
