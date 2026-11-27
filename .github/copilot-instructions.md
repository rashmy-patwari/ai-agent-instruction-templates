# AI Agent Instructions (Template)

> Template instructions for AI coding agents (GitHub Copilot, Claude, and
> compatible tools) working on an **MCP server that bridges AI assistants to a
> backend content or data platform**. Copy this file to
> `.github/copilot-instructions.md` in your project and replace the bracketed
> placeholders with your specifics.

## Project Overview

This project is a **protocol translator** between AI services (GitHub Copilot,
Azure OpenAI, Claude, and others) and a backend platform (for example, a
headless CMS, DAM, product catalog, or other REST API). It is **not an AI
solution itself** — it provides MCP (Model Context Protocol) tool definitions
that AI services call to access backend data.

**Key architecture:**
- **MCP Server** (`servers/<server-name>/`): the service exposing MCP tools
- **Client / transport layer** (`clients/<client-name>/`): optional editor
  extension or thin client that forwards requests to the server
- **Backend integration**: a REST API client with authentication and caching

Replace these with your module names and a one-line description of the backend
you integrate with.

## Critical Development Rules

### 1. No AI Processing in Server Code

The server is a **stateless data gateway** — all AI processing happens in
external services (the calling assistant or model).

Do NOT add to the server:
- Embeddings generation
- NLP / prompt engineering
- RAG implementations

ONLY add to the server:
- Tool definitions (`src/mcp/tools.*`)
- Backend API calls (`src/services/<backend>Client.*`)
- Middleware for security, caching, and logging

### 2. MCP Tool Pattern

All tools follow a consistent structure. Define each tool with a name, a
natural-language description the model uses to decide when to call it, a
validated input schema, and an `execute` function that talks to the backend:

```javascript
const toolName = {
  name: 'tool_name',
  description: 'Natural-language description for the AI to know when to use this',
  inputSchema: schema.object({
    param: schema.string().required().description('AI-readable param description'),
  }),
  async execute(args, services) {
    const { backend, cache } = services;
    // 1. Check cache (if enabled)
    // 2. Call backend API
    // 3. Format response
    // 4. Cache result
    return result;
  },
};
```

**Suggested conventions:**
- Accept common parameter aliases (for example both `limit` and `take`) when
  different AI platforms send different names.
- Use a `useCache` boolean for per-call cache control.
- Allow unknown input keys so extra parameters from some AI platforms do not
  fail validation.
- Centralize cache-key construction so keys are deterministic.

### 3. Backend Client Patterns

Keep all backend access in one client module (`src/services/<backend>Client.*`).

**Authentication flow** — manage tokens centrally and refresh on expiry:

```javascript
await this.ensureAuthenticated(); // checks expiry, refreshes if needed
// token auto-injected into request headers by an interceptor
```

**API call pattern** — route every backend call through one HTTP client with
retry/backoff so behavior is consistent:

```javascript
const response = await this.httpClient.post('<backend query endpoint>', {
  query: { /* backend query structure */ },
});
```

**Cache integration** — check the cache before calling, and store results with
an explicit TTL:

```javascript
const cacheKey = Cache.createSearchKey(query, options);
const cached = await cache.get(cacheKey);
if (cached) return cached;

const result = await backend.search(query, options);
await cache.set(cacheKey, result, 1800); // seconds
```

## Development Workflows

### Local Development

```bash
# Server
cd servers/<server-name>
npm install
cp .env.example .env   # configure backend credentials (never commit .env)
npm run dev            # runs with hot reload

# Client / extension (if present)
cd clients/<client-name>
npm install
npm run compile        # build step for compiled languages
```

### Testing

Provide a script or test suite that exercises every tool plus the health
endpoints, and run it before committing.

**Typical endpoints:**
- `/health` — simple health check
- `/health/full` — detailed diagnostics (backend connection, cache, metrics)
- `/mcp/tools/list` — list available MCP tools
- `/mcp/tools/call` — execute a tool (POST with `{ name, arguments }`)

### Deployment

Document your deployment commands here. Keep credentials out of source — pass
them as environment variables or, better, inject them from a managed secret
store at deploy time.

**Environment variables (example):**
- `BACKEND_BASE_URL` — backend instance URL
- `BACKEND_USERNAME` / `BACKEND_PASSWORD` — basic auth, or
- `BACKEND_CLIENT_ID` / `BACKEND_CLIENT_SECRET` — OAuth2 client credentials

## Integration Points

### MCP Server to Backend

Document the backend's auth method(s) and the core endpoints your tools use,
for example:
- Entity search: `POST <query endpoint>`
- Item retrieval: `GET <item endpoint>/{id}`
- Metadata: `GET <item endpoint>/{id}/properties`

### Client to MCP Server

Keep any editor extension or client a **pure transport layer** — forward the
request to the server and render the response. No business logic on the client:

```typescript
async callTool(toolName: string, args: unknown) {
  return httpPost(this.serverUrl, { name: toolName, arguments: args });
}
```

## Project-Specific Conventions

### Error Handling

Return structured errors with a stable shape so clients can handle them
predictably:

```javascript
{
  status: 'error',
  message: 'User-friendly message',
  code: 'ERROR_CODE',
  details: { /* additional context */ },
  correlationId: 'uuid',
}
```

### Logging

Use a structured logger and never log secrets, tokens, or full credentials.
Redact sensitive fields before logging. Use log levels deliberately: error
(always), warn, info (normal operation), debug (verbose/local only).

### Security

Treat security as a first-class requirement:
- Store credentials in a managed secret store; never commit them.
- Use expiring, rotatable API keys rather than static ones.
- Enforce TLS for all network connections, including to caches and datastores.
- Validate and constrain all tool inputs at the boundary.
- Scrub tokens and credentials from logs and error output.

> Do not document known, unmitigated vulnerabilities of a live deployment in a
> public file. Track those privately and fix them.

## Common Gotchas

1. **Cross-platform parameter names:** different AI platforms send different
   parameter names and extra keys — normalize aliases and allow unknown keys.
2. **Cache keys must be deterministic:** sort and stringify objects before
   hashing.
3. **Token refresh belongs in the client/interceptor**, not in tool code.
4. **Rebuild after changes** in compiled clients before testing.
5. **Update the client's server URL** after each deployment.

## Key Files Reference

- Tool definitions: `servers/<server-name>/src/mcp/tools.*`
- Backend client: `servers/<server-name>/src/services/<backend>Client.*`
- Cache service: `servers/<server-name>/src/services/cacheService.*`
- Client entry point: `clients/<client-name>/src/...`
- Deployment scripts: `servers/<server-name>/deploy/`
