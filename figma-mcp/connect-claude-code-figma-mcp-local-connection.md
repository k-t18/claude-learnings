# Connecting Claude Code with Figma MCP

## Overview

This guide explains how to connect **Claude Code** with **Figma** using the Model Context Protocol (MCP). Once connected, Claude can read your Figma designs directly and help you generate code, describe layouts, and build components — all from your terminal.

The easiest and recommended approach for local development uses the **Figma Desktop app**, which runs a built-in local MCP server on your machine. No API tokens or npm packages required.

---

## Prerequisites

| Requirement                                                   | Notes                       |
| ------------------------------------------------------------- | --------------------------- |
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | Installed and authenticated |
| [Figma Desktop](https://www.figma.com/downloads/)             | Installed and logged in     |

---

## Setup

### Step 1 — Register the Figma MCP Server

Run the following command once in your terminal:

```bash
claude mcp add --transport sse figma-local http://127.0.0.1:3845/sse
```

**What this does:**

- Registers a new MCP server called `figma-local` in your Claude Code config
- Points it to the local SSE server that Figma Desktop automatically runs on port `3845`
- This is a one-time setup per machine

### Step 2 — Verify the Connection

Open Claude Code in your terminal:

```bash
claude
```

Then run the MCP status command:

```
/mcp
```

You should see `figma-local` listed as a connected server:

```
Connected MCP Servers:
  ✓ figma-local  (sse @ http://127.0.0.1:3845/sse)
```

> **Note:** Figma Desktop must be open and running for the connection to be active.

---

## Usage

### Sharing a Figma Frame with Claude

1. Open your file in the **Figma Desktop app**
2. Select any frame, component, or element
3. Right-click → **Copy link to selection**
4. Paste the link into your Claude Code session

Claude will fetch the design context from Figma and use it to assist you.

### Example Prompts

**Generate a React component from a design:**

```
Here's a Figma frame: https://www.figma.com/file/...
Build a React + Tailwind component that matches this design exactly.
```

**Describe a layout:**

```
Here's a Figma frame: https://www.figma.com/file/...
Describe the layout, spacing, and typography used in this design.
```

**Generate multiple components:**

```
Here's my Figma file: https://www.figma.com/file/...
Generate React components for the Button, Card, and Navbar frames.
```

---

## How It Works

```
Figma Desktop  ──────►  Local MCP Server (port 3845)
                                  │
                                  ▼
                          Claude Code (CLI)
                                  │
                                  ▼
                     Reads design → Generates code
```

The Figma Desktop app exposes a local SSE (Server-Sent Events) endpoint at `http://127.0.0.1:3845/sse`. Claude Code connects to this endpoint via the MCP protocol, allowing it to query design data in real time from whatever file you have open.

---

## Troubleshooting

**`figma-local` not showing up in `/mcp`**

- Make sure Figma Desktop is open and you are logged in
- Try restarting Figma Desktop and re-running `claude`
- Re-run the `claude mcp add` command and check for errors

**Claude says it can't access the Figma link**

- Ensure the link was copied using **Copy link to selection** (not Copy link to file)
- Confirm the frame is in a file you have access to in Figma

**Port 3845 not available**

- Another process may be using the port. Run `lsof -i :3845` to check
- Restarting Figma Desktop usually resolves this

---

## Notes

- This MCP connection is **local to your machine** — each developer on your team needs to run the `claude mcp add` command once
- For a shared team config, you can commit `.claude/settings.json` to your repo (it will contain the server URL but no sensitive credentials)
- This approach works on **macOS** with Figma Desktop. For Linux/Windows or CI environments, consider the token-based npm server approach instead

---

## References

- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Figma Desktop Downloads](https://www.figma.com/downloads/)
- [Anthropic MCP Overview](https://docs.anthropic.com/en/docs/mcp)
- [Figma Developer Platform](https://www.figma.com/developers)
