# Model Context Protocol (MCP) Specification Reference

**Protocol Revision**: 2025-11-25
**Source**: [modelcontextprotocol.io/specification/2025-11-25](https://modelcontextprotocol.io/specification/2025-11-25)
**Schema**: [schema.ts](https://github.com/modelcontextprotocol/specification/blob/main/schema/2025-11-25/schema.ts)

## What Changed from 2025-06-18

The November 2025 release (MCP's 1-year anniversary) adds:

- **Tasks**: New abstraction for tracking server work, queryable status, results available after creation
- **Enhanced Tool Calling**: Tool calling in sampling requests, server-side agent loops, parallel tool calls
- **Extensions**: Components outside core spec for scenario-specific additions
- **Icons**: Standardized visual identifiers for tools, resources, prompts, implementations
- **`title` field on tools**: Human-readable display name separate from `name`
- **`_meta` property**: Reserved metadata with reverse-DNS key naming
- **MCP-Protocol-Version header**: Required on all HTTP requests after initialization
- **Security Best Practices**: Expanded guidance (session hijacking, token passthrough, scope minimization)

## Core Architecture

MCP uses **client-host-server architecture** with JSON-RPC 2.0 messaging.

- **Hosts**: LLM applications that initiate connections, manage clients, enforce security
- **Clients**: Connectors within host, 1:1 relationship with servers, maintain session isolation
- **Servers**: Provide resources, tools, prompts; operate independently

### Key Design Principles

1. Servers should be easy to build (hosts handle orchestration)
2. Servers should be composable (focused functionality, combinable)
3. Servers cannot see the full conversation or other servers
4. Features added progressively via capability negotiation

## Transports

### stdio

- Client launches server as subprocess
- Messages on stdin/stdout, newline-delimited, no embedded newlines
- stderr for logging (MAY write UTF-8 strings)
- Server MUST NOT write non-MCP content to stdout

### Streamable HTTP (replaces HTTP+SSE from 2024-11-05)

- Single MCP endpoint supporting POST and GET
- POST for client-to-server messages; GET for server-initiated SSE streams
- Session management via `MCP-Session-Id` header (cryptographically secure)
- `MCP-Protocol-Version` header MUST be included on all requests after init

**Security Requirements**:

- MUST validate `Origin` header (DNS rebinding prevention)
- If Origin present and invalid, MUST respond 403 Forbidden
- Local servers SHOULD bind only to localhost (127.0.0.1)
- SHOULD implement proper authentication

**Resumability**: Servers MAY attach SSE event IDs; clients resume via GET with `Last-Event-ID`

## Capability Negotiation

During initialization, clients and servers declare supported features:

**Server capabilities**: resources, tools, prompts (each with optional `listChanged`)
**Client capabilities**: sampling, roots, elicitation

Both parties MUST respect declared capabilities throughout the session.

## Server Features

### Tools

Tools are **model-controlled** functions for interacting with external systems.

**Tool Definition**:

- `name`: Unique identifier (1-128 chars, case-sensitive, alphanumeric + `_-. `)
- `title`: Optional human-readable display name (new in 2025-11-25)
- `description`: Human-readable description
- `inputSchema`: JSON Schema (2020-12 default) for parameters
- `outputSchema`: Optional JSON Schema for structured output validation
- `annotations`: Optional behavioral hints (see below)
- `icons`: Optional array of icon objects

**Tool Annotations** (behavioral hints, NOT security guarantees):

```json
{
  "annotations": {
    "title": "Human-readable title",
    "readOnlyHint": true,
    "destructiveHint": false,
    "idempotentHint": true,
    "openWorldHint": true
  }
}
```

- `readOnlyHint`: Tool does not modify its environment (default: false)
- `destructiveHint`: Tool may perform destructive updates (default: true)
- `idempotentHint`: Repeated calls with same args have no additional effect (default: false)
- `openWorldHint`: Tool interacts with external entities (default: true)

**CRITICAL**: Clients MUST consider annotations untrusted unless from a trusted server.

**Tool Results**: Can be unstructured (`content` array) or structured (`structuredContent` JSON).
Content types: text, image, audio, resource_link, embedded resource.
If `outputSchema` provided, servers MUST return conforming `structuredContent`.

**Error Handling**:

1. Protocol errors: JSON-RPC errors (-32602 for unknown tool, etc.)
2. Tool execution errors: `isError: true` in result (actionable, model can self-correct)

### Resources

- Context and data for users/models
- Application-controlled (attached by client)
- URI-based with subscriptions for updates

### Prompts

- Pre-defined templates/instructions
- User-controlled (invoked by user choice)

## Client Features

- **Sampling**: Server-initiated LLM interactions (requires explicit user approval)
- **Roots**: Server-initiated queries about filesystem/URI boundaries
- **Elicitation**: Server requests additional user input with structured JSON schemas

## Authorization (HTTP transports)

Based on **OAuth 2.1** with:

- **PKCE**: MUST use S256 code challenge method
- **Resource Indicators (RFC 8707)**: MUST include `resource` parameter in auth/token requests
- **Protected Resource Metadata (RFC 9728)**: Servers MUST implement for auth server discovery
- **Client ID Metadata Documents**: Preferred registration mechanism (HTTPS URLs as client IDs)
- **Token audience validation**: Servers MUST validate tokens were issued specifically for them
- **Token passthrough forbidden**: Servers MUST NOT forward received tokens to upstream APIs

**Discovery flow**: Client gets 401 -> extracts resource_metadata from WWW-Authenticate -> fetches Protected Resource Metadata -> discovers authorization server -> proceeds with OAuth flow

## Error Codes

Standard JSON-RPC error codes:

- `-32700`: Parse error
- `-32600`: Invalid request
- `-32601`: Method not found
- `-32602`: Invalid params
- `-32603`: Internal error
- `-32000` to `-32099`: Server errors (MCP-specific)
- `-32001`: MCP transport error

## Security Considerations

### Key Principles

1. **User Consent**: Users must explicitly consent to all data access and operations
2. **Data Privacy**: Hosts must not transmit data without user consent
3. **Tool Safety**: Tools represent arbitrary code execution; annotations are untrusted
4. **LLM Sampling Controls**: Users must approve sampling requests, control prompts

### Implementation Requirements

**Servers MUST**:

- Validate all tool inputs
- Implement proper access controls
- Rate limit tool invocations
- Sanitize tool outputs

**Clients SHOULD**:

- Prompt for user confirmation on sensitive operations
- Show tool inputs before calling server (prevent data exfiltration)
- Validate tool results before passing to LLM
- Implement timeouts for tool calls
- Log tool usage for audit purposes

### Session Security

- Session IDs MUST be cryptographically secure
- Clients MUST handle session IDs securely
- Sessions can be terminated by server at any time (404 response)
- Clients SHOULD send DELETE to terminate sessions they no longer need

## Relevance to Inspector Assessment

### What the Inspector Should Test

| Spec Requirement          | Inspector Module          | Notes                                          |
| ------------------------- | ------------------------- | ---------------------------------------------- |
| Tool input validation     | SecurityAssessor          | Injection payloads test server-side validation |
| Error response format     | ErrorHandlingAssessor     | JSON-RPC error codes, isError field            |
| Tool annotations accuracy | ToolAnnotationAssessor    | readOnlyHint vs actual behavior                |
| JSON-RPC compliance       | MCPSpecComplianceAssessor | Message format, capability negotiation         |
| Tool functionality        | FunctionalityAssessor     | Tools work with valid inputs                   |
| Output schema conformance | MCPSpecComplianceAssessor | structuredContent matches outputSchema         |

### What the Spec Says About Security Testing

- Annotations are **untrusted hints** - cannot rely on them for security decisions
- Tool safety is the **host's responsibility** (not the server's)
- Protocol-level security is about transport (Origin validation, token handling)
- Application-level security is about input validation and access controls
- The spec does NOT define vulnerability categories - that's implementation-specific

### Transport-Aware Testing Implications

- **stdio servers**: Full security testing (command injection, path traversal relevant)
- **HTTP servers**: Transport security matters (Origin, auth), but path traversal less relevant
- **Both**: Input validation, error handling, annotation accuracy always relevant
