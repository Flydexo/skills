# Tools and MCP

Use when selecting an access path or troubleshooting tool access. Default to the Dokploy API at `https://dokploy.flydexo.com`, with credential handling and version-specific discovery in [dokploy-api.md](dokploy-api.md#authentication-and-schema-discovery).

## API-first, capability-aware MCP

An already configured Dokploy MCP connection is optional, not an automatic installation step:

1. Discover actual tool names, input schemas, and read/write effects. Listing MCP resources alone does not establish that management tools are available.
2. Verify that the connection targets this Dokploy instance and the intended resource. If the target is hidden and cannot be verified safely, use the instance-scoped API instead.
3. Use only tools that express the requested operation with the resolved IDs and scope. For missing capabilities, use the documented API rather than inventing tools or assuming Railway's tool names exist.

Both paths require the same destructive-change confirmation and deployment checks. A successful MCP response does not prove an asynchronous deployment is healthy. Do not alternate paths to repeat a write with an unknown outcome; inspect state first.

Report unavailable tools or access failures specifically. Use a safe equivalent only if it preserves the target, operation, and credential protection. Do not expand an application task into installing a CLI/MCP server, modifying global client settings, or configuring SSH.

## Official Dokploy MCP

The official [Dokploy MCP documentation](https://docs.dokploy.com/docs/core/mcp), reviewed November 16, 2026, describes `@dokploy/mcp` with `DOKPLOY_URL` and `DOKPLOY_API_KEY` configuration variables. These belong to that integration; this user's existing local API credential remains `DOKPLOY_API_TOKEN`, sent as `x-api-key` for direct calls.

If explicitly asked to configure MCP, refresh official setup instructions and check the client's supported secret-input mechanism. Resolve the existing trusted credential only when needed, and pass it through a protected environment/input channel. Do not copy the value into skill files, chat, command arguments, or committed configuration. Do not assume a client expands environment variables in JSON or deploy literal placeholder credentials.

This skill update neither installs that package nor configures or tests a live MCP connection. The existing API reference was checked on Dokploy v0.30.6; current MCP documentation does not establish this server's compatibility with every operation.

## Railway inspiration and boundaries

Official sources reviewed November 16, 2026:

- [Railway's `use-railway` skill](https://github.com/railwayapp/railway-skills/blob/main/plugins/railway/skills/use-railway/SKILL.md): intent routing, explicit project/environment/service context, focused references, and evidence-based reporting.
- [Deployment guidance](https://github.com/railwayapp/railway-skills/blob/main/plugins/railway/skills/use-railway/references/deploy.md): correlate the submitted deployment, not a queue acknowledgement or another release's success.
- [Configuration guidance](https://github.com/railwayapp/railway-skills/blob/main/plugins/railway/skills/use-railway/references/configure.md): inspect current configuration and apply related changes coherently.
- [Operations guidance](https://github.com/railwayapp/railway-skills/blob/main/plugins/railway/skills/use-railway/references/operate.md): bounded logs and failure-layer diagnosis before recovery.
- [Official Railway MCP documentation](https://docs.railway.com/ai/mcp-server): tools expose particular capabilities, not necessarily every platform operation.

Borrow the workflow, not the provider contract. Railway CLI commands, linked-directory context, staged changes, service references, private DNS suffixes, builders, status enums, and MCP tool names are not interchangeable with Dokploy. This skill does not require Railway or authorize installing its tools or migrating to it.
