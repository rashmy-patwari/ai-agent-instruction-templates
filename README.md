# AI Agent Instruction Templates

Vendor-neutral starter templates for the instruction files that AI coding
agents read — **GitHub Copilot** (`.github/copilot-instructions.md`) and
**Claude Code** (`CLAUDE.md`).

These templates are written for a common project shape: an **MCP (Model Context
Protocol) server that bridges AI assistants to a backend content or data
platform** (a CMS, DAM, product catalog, or other REST API). They capture
reusable conventions — the MCP tool pattern, backend client patterns, caching,
error handling, logging, and security practices — without tying you to any
specific product, client, or deployment.

## What's here

| File | For | Purpose |
|------|-----|---------|
| [`.github/copilot-instructions.md`](.github/copilot-instructions.md) | GitHub Copilot | Repository-wide agent instructions |
| [`CLAUDE.md`](CLAUDE.md) | Claude Code | Session guidance Claude reads on every run |

## How to use

1. Copy one or both files into your repository at the paths above.
2. Replace the bracketed placeholders (`<server-name>`, `<backend>`, endpoints,
   commands) with your project's specifics.
3. Trim anything that does not apply, and add project-specific rules.
4. Keep the files short and high-signal — agents read them often.

## Principles baked in

- The MCP server is a **stateless data gateway**; AI processing stays in the
  calling model, not the server.
- Any editor extension or client is a **pure transport layer**.
- Validate inputs at the boundary; fail fast with explicit errors.
- Never commit secrets; never log credentials; never publish known, unmitigated
  vulnerabilities of a live deployment.

## License

Released under the [MIT License](LICENSE) — use, copy, and adapt freely.
