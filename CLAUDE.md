# CLAUDE.md (Template)

> Guidance for Claude Code (and compatible agents) working in this repository.
> This is a vendor-neutral template for an **MCP server that bridges AI
> assistants to a backend content or data platform**. Copy it to `CLAUDE.md` at
> your repo root and replace the bracketed placeholders.
>
> Keep this file short and high-signal. Claude reads it on every session, so
> prefer durable facts and conventions over anything that changes often or can
> be derived from the code.

## What this project is

A **protocol translator** (MCP server) between AI assistants and a backend
platform (CMS, DAM, catalog, or other REST API). The server exposes MCP tool
definitions; it performs **no AI processing itself**. All reasoning happens in
the calling model.

- MCP server: `servers/<server-name>/`
- Client / editor extension (optional): `clients/<client-name>/`
- Backend client: `servers/<server-name>/src/services/<backend>Client.*`

## Architecture rules (do not violate)

- The server is a **stateless data gateway**. Do not add embeddings, NLP,
  prompt engineering, or RAG to it.
- Add only: MCP tool definitions, backend API calls, and
  security/caching/logging middleware.
- Any client/extension is a **pure transport layer** — no business logic.

## How to work here

- Prefer architecture, clarity, and correctness over quick fixes. Explain
  non-obvious design decisions briefly before implementing.
- Keep changes focused and minimal; do not edit more than the task needs.
- Fail fast with explicit errors. No silent fallbacks that mask failures.
- Validate inputs at the boundary; avoid magic values and hardcoded defaults.
- Match the surrounding code's style, naming, and idioms.
- Write and run tests before considering a change done. Never disable a check
  or assertion just to make a test pass.

## Adding an MCP tool

1. Define the tool in `src/mcp/tools.*` with: `name`, a natural-language
   `description` (the model uses it to decide when to call), a validated
   `inputSchema`, and an `async execute(args, services)`.
2. In `execute`: check cache (if enabled) -> call the backend client -> format
   the response -> cache the result.
3. Accept common parameter aliases across AI platforms (for example `limit`
   and `take`) and allow unknown input keys so extra parameters do not fail
   validation.
4. Keep all backend access inside the backend client module, not in the tool.

## Backend client

- Centralize authentication: check token expiry and refresh via an interceptor;
  never refresh tokens inside tool code.
- Route every backend call through one HTTP client with retry/backoff.
- Use deterministic cache keys (sort/stringify objects before hashing) and set
  an explicit TTL on every cache write.

## Commands

```bash
# Server
cd servers/<server-name>
npm install
cp .env.example .env    # configure backend credentials; never commit .env
npm run dev             # hot reload
npm test                # run tests

# Client / extension (if present)
cd clients/<client-name>
npm install && npm run compile
```

Health/diagnostics: `/health`, `/health/full`. MCP: `/mcp/tools/list`,
`/mcp/tools/call`.

## Security and secrets

- Never commit secrets. Keep credentials in `.env` (gitignored) locally and in
  a managed secret store in deployed environments.
- Use expiring, rotatable keys; enforce TLS everywhere, including caches.
- Never log tokens or credentials; redact sensitive fields.
- Do not document known, unmitigated vulnerabilities of a live deployment in
  any file that may become public. Track and fix them privately.

## Conventions

- Structured errors: `{ status, message, code, details, correlationId }`.
- Structured logging with deliberate levels (error, warn, info, debug).
- Keep documentation in `docs/` as Markdown; file names in kebab-case.
- No emojis or icons in code, docs, or commit messages.
