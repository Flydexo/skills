# Dokploy API and deployment notes

Checked during a deployment to `https://dokploy.flydexo.com` on November 12, 2026, using server version **v0.30.6**. These are instance/version-specific starting points, not a replacement for the server's current schema. Updating this skill does not authorize a deployment or changes to unrelated applications.

## Authentication and schema discovery

API base: `https://dokploy.flydexo.com/api/`. Authenticate with **`x-api-key`**, using the resolved `DOKPLOY_API_TOKEN` from the trusted `~/.codex/secrets.env` file. The file can contain a shell command substitution; a regex that extracts the right-hand side will send the wrong credential. Do not print or persist the resolved token, enable shell tracing, or place the token in command arguments.

An authenticated, read-only discovery example:

```bash
set -a
source "$HOME/.codex/secrets.env"
set +a
node --input-type=module <<'JS'
const response = await fetch(
  'https://dokploy.flydexo.com/api/settings.getOpenApiDocument',
  { headers: { 'x-api-key': process.env.DOKPLOY_API_TOKEN.trim() } }
);
if (!response.ok) throw new Error(`Schema discovery: HTTP ${response.status}`);
const schema = await response.json();
console.log(schema.info);
JS
```

- `GET /api/settings.getOpenApiDocument` returns the actual OpenAPI document. Its `paths` are named `/compose.create`, etc.; prepend `/api` when calling the instance.
- `GET /api/settings.getDokployVersion` returns the installed version.
- Swagger UI is at `/swagger`; its availability depends on authentication and server settings. Do not guess `/openapi.json` paths when the authenticated schema endpoint works.
- Send GET parameters through the URL query string; POST inputs are JSON with `Content-Type: application/json`. Check response status before interpreting the body.
- Some OpenAPI success schemas are empty object placeholders. Inspect the actual response for returned IDs, without logging sensitive fields from whole configuration objects.
- If authentication still returns 401 after resolving the stored credential and checking the documented header, stop mutations and request a replacement through the private secret-entry mechanism; do not repeatedly try literal extraction or alternative auth schemes.

## Inspect before creating resources

`GET /api/project.all` returns projects with nested `environments[]`. Each environment contains `environmentId`, `compose[]` and `applications[]`. Reuse a matching project/environment/service rather than creating a duplicate. Read the selected service with `GET /api/compose.one?composeId=...` before updates.

Check host conflicts with `GET /api/domain.byComposeId?composeId=...` for relevant existing Compose services; use the corresponding application-domain endpoint for application services when needed. Inspect only fields relevant to the task, rather than printing whole records that might contain configuration secrets.

`POST /api/domain.validateDomain` accepts `{ "domain": "cloudline.flydexo.com" }` and optionally `serverId`. Cloudflare proxy addresses and a CDN warning do not by themselves mean invalid DNS: the observed response had `isValid: true` plus an `error` text explaining that Cloudflare masks the origin. Check the validity flag, CDN context and actual public reachability together.

## Create or update a Compose deployment

Examples below illustrate verified payload shapes. Substitute the intended app, returned IDs and actual Compose content; do not reuse Cloudline's deployment IDs for a different project.

| Endpoint | Relevant input / response |
| --- | --- |
| `POST /api/project.create` | `{ "name": "cloudline", "description": "..." }`; observed response contains `project.projectId` and `environment.environmentId`, not only flat IDs. |
| `POST /api/compose.create` | `name`, `environmentId`; optional `appName`, `description`, `sourceType`, `composeType`, `composeFile`, `serverId`. |
| `POST /api/compose.update` | Required `composeId`, plus only fields being changed. Supports `composeFile`, `command`, `sourceType`, `composePath`, `isolatedDeployment`, Git provider settings, etc.; inspect current schema for exact names. |
| `GET /api/compose.getDefaultCommand?composeId=...` | Returns the rendered command including Dokploy's added `docker` prefix. |
| `POST /api/compose.deploy` | `{ "composeId": "RETURNED_ID", "title": "...", "description": "..." }`; observed response acknowledges queueing, not completion. |

For raw Compose, use `sourceType: "raw"`, `composeType: "docker-compose"`, and the Compose document in `composeFile`. Choose a Git-backed source when appropriate; this reference does not require raw source for every app. A local project folder is not automatically a build context on the remote server. Raw builds need available images or self-contained build instructions; Git builds need published source and the correct Compose path.

### Returned names and command prefix

Even when an `appName` was supplied at creation, the server returned a suffixed name such as `cloudline-ccgt55`. Use the **returned** `appName` for Compose project matching and container inspection. Do not infer or strip its suffix.

Preserve the existing/default command when possible. If a custom command is needed, `compose.command` contains arguments **after** the executable `docker`:

```text
Stored command:   compose -p RETURNED_APP_NAME up -d --build --remove-orphans
Rendered command: docker compose -p RETURNED_APP_NAME up -d --build --remove-orphans
```

Storing `docker compose ...` produces `docker docker compose ...` and fails before building. Inspect `compose.getDefaultCommand` after a necessary command change. Use the app's existing options rather than applying the example blindly; do not enable fresh-volume operations to fix an ordinary build error.

### Raw Compose source fidelity and size

Nested Dockerfile/source embedding passes through several interpolation layers. Plain `$$` escaping did not preserve literal currency prefixes such as `$${value}` inside JavaScript templates during the observed deployment, although the application still built and started.

If embedding sources is necessary:

- Encode source bytes instead of placing arbitrary JavaScript/HTML directly into Dockerfile heredocs. Gzip-compressed base64 decoded using Node's built-in `zlib` and `Buffer` worked for a Node image without adding build dependencies. Use an appropriate decoder for other runtimes.
- Base64 is **not encryption**. Never embed credentials, private environment files or other secrets in the Compose document or image layers.
- Keep raw exports compact. An approximately 98 KB export failed during initialization with only a generic log error; the compact approximately 33 KB export worked. Shell transport/argument-size limits are a possible cause, not a documented API payload limit. A target below 64 KiB provides useful headroom for this workflow; it is not a universal Dokploy quota.
- Test decoding locally against the original bytes. After deployment, verify served JavaScript/CSS hashes against local source. HTML can legitimately differ because Cloudflare injects challenge scripts; check page identity and critical assets rather than requiring an exact HTML hash.
- Keep runtime-specific networking relationships intact. For example, `network_mode: service:app` sidecars need their owning app to exist; recreating that app may require recreating the sidecar. Do not enable isolated/randomized deployments if they would break those relationships.

## Domain routing for this instance

The upstream proxy terminates public HTTPS. Existing apps used Dokploy HTTP origin routing. Match that setup without changing global Cloudflare settings:

```json
{
  "host": "cloudline.flydexo.com",
  "path": "/",
  "port": 3000,
  "https": false,
  "certificateType": "none",
  "composeId": "RETURNED_COMPOSE_ID",
  "serviceName": "app",
  "domainType": "compose",
  "stripPath": false
}
```

- Create with `POST /api/domain.create`; retain its returned `domainId`. Update with `POST /api/domain.update`, supplying `domainId`, required `host`, and only intended changes.
- The service name and port must match the actual app listener, not necessarily this example's `app:3000`. Route only the public service, never a database or emulator admin API.
- Redeploy the Compose service after changing domain labels so the running container receives the updated routing configuration.
- Enabling Dokploy HTTPS on this setup caused a redirect to the **same public HTTPS URL** repeatedly, because Cloudflare connected to the origin using HTTP. Correct this app's domain settings, not global proxy SSL settings or unrelated applications.
- Still report and test the public `https://` URL. This arrangement is edge TLS, **not** end-to-end TLS between Cloudflare and the origin.
- Check Secure/HttpOnly/SameSite cookie flags. An app relying only on the origin's `X-Forwarded-Proto` can miss `Secure`; use the app's supported production/public-HTTPS configuration rather than weakening cookie protection.

## Status, logs and acceptance checks

| Endpoint | GET query parameters |
| --- | --- |
| `/api/deployment.allByCompose` | `composeId`; inspect the intended/latest deployment ID, status and timestamps. |
| `/api/deployment.readLogs` | `deploymentId`, optional `tail`. |
| `/api/docker.getContainersByAppNameMatch` | Returned `appName`, `appType: "docker-compose"`; optional `serverId` for a remote server. |
| `/api/compose.readLogs` | `composeId`, returned `containerId`, optional `tail` / `since` / supported search parameters. |

Observed deployment statuses include `running`, `done`, and `error`. A deployment can be `done` while container health is still starting. Traefik can initially return 404 until a health-checked app becomes healthy. Inspect container health and logs before changing DNS or routing based solely on that initial 404.

After success, verify public HTTPS health, page identity and assets, then exercise the app's principal flow using a disposable session. For Cloudline this included real emulator provisioning, launch, pause, reset/cleanup, and confirming the emulator admin route was not publicly exposed. Adapt the smoke test to the actual app; do not mutate existing user data to prove functionality.

When a deployment fails, read that deployment's logs and relevant container logs, fix the specific in-scope cause, then make a deliberate retry. If the same cause repeats without a new corrective action, stop retrying and report the blocker; do not delete volumes/projects, prune the server, or change unrelated/global settings as a fallback.

## Primary references

- API usage and `x-api-key`: https://docs.dokploy.com/docs/api
- Compose domains: https://docs.dokploy.com/docs/core/docker-compose/domains
- Exact deployed-version router: https://github.com/Dokploy/dokploy/blob/v0.30.6/apps/dokploy/server/api/routers/compose.ts
- Command construction: https://github.com/Dokploy/dokploy/blob/v0.30.6/packages/server/src/utils/builders/compose.ts
- Refresh with the authenticated server OpenAPI document rather than assuming upstream `canary` matches the installed version.
