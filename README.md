# Claude Code Frontend Skills

> **Work in progress** — skills and guides are actively being developed. Feedback, suggestions, and contributions are welcome.

A collection of reusable Claude Code skills and reference guides for building frontend applications. Each skill is a structured prompt that directs Claude through a specific development workflow — from generating API integrations to building form UIs from Figma designs.

---

## Skills

Skills live in the `skills/` directory. Each one is a `SKILL.md` file you can install into your Claude Code setup.

| Skill                                                            | What it does                                                                             |
| ---------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| [`api-integration`](./skills/api-integration/SKILL.md)           | Generates a typed service file and custom TanStack Query hook from an API endpoint       |
| [`component-builder`](./skills/component-builder/SKILL.md)       | Converts a Figma design reference into a production-grade reusable UI component          |
| [`design-system-setup`](./skills/design-system-setup/SKILL.md)   | Extracts, normalizes, and implements design tokens from Figma into a Tailwind v4 project |
| [`form-ui`](./skills/form-ui/SKILL.md)                           | Builds a styled form component using PrimeReact inputs and React Hook Form               |
| [`form-api-integration`](./skills/form-api-integration/SKILL.md) | Wires an existing form component to an API with Zod validation and a mutation hook       |
| [`hydration-safe-render`](./skills/hydration-safe-render/SKILL.md) | Eliminates SSR hydration mismatches and visual flickering for client-only storage reads |

### Recommended workflow

The form skills are designed to run in sequence:

```
form-ui → form-api-integration
```

`form-ui` generates the visual component and typed props. `form-api-integration` picks up from there and handles the Zod schema, service file, mutation hook, and container wiring — without touching the form component itself.

The `api-integration` skill is standalone and can be run independently for any endpoint.

---

## Reference Guides

### API Architecture

[`API_ARCHITECTURE/API_ARCHITECTURE.md`](./API_ARCHITECTURE/API_ARCHITECTURE.md)

Documents the layered API pattern the `api-integration` and `form-api-integration` skills follow. Covers the separation across services, custom hooks, and TanStack Query wrappers, and defines the fixed infrastructure files that skills never modify.

### Figma MCP Integration

[`figma-mcp/`](./figma-mcp/)

Guides for connecting Claude Code to Figma's MCP server, which the `form-ui`, `component-builder`, and `design-system-setup` skills rely on for design references.

| Guide                                                                                                                  | When to use                                           |
| ---------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| [`figma-mcp-connection-approaches.md`](./figma-mcp/figma-mcp-connection-approaches.md)                                 | Overview comparing both connection methods            |
| [`connect-claude-code-figma-mcp-local-connection.md`](./figma-mcp/connect-claude-code-figma-mcp-local-connection.md)   | Local setup via Figma Desktop (zero config, solo dev) |
| [`connect-claude-code-figma-mcp-shared-connection.md`](./figma-mcp/connect-claude-code-figma-mcp-shared-connection.md) | Shared setup via npm + Personal Access Token (teams)  |

---

## Contributing

This project is a work in progress and open for contributions. If you'd like to help:

- **New skills** — open a PR with a new `skills/<skill-name>/SKILL.md` following the existing structure
- **Improvements** — edits to existing skills, guides, or architecture docs are welcome
- **Bug reports / feedback** — open an issue describing what's wrong or what's missing

There's no strict contribution guide yet — keep the skill format consistent and include a clear frontmatter `description` field so Claude knows when to invoke it.
