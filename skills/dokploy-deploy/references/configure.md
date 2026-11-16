# Configuration changes

Use for variables, domains, source/build settings, and storage. Resolve the target using `SKILL.md`; read the relevant payload guidance in [dokploy-api.md](dokploy-api.md) before API writes.

## Read, change, verify

Read current configuration and the current schema for this resource type. Change only requested fields; some settings accept a whole document rather than a key-level patch. Inspect selectively and report redacted differences, not complete records containing credentials.

Batch related requested changes where supported instead of deploying after every field. Check whether a setting applies immediately, requires a redeploy, or starts one itself. Do not assume Railway's staged-change behavior exists in Dokploy. Re-read affected fields after writing; verify any resulting deployment using [operate.md](operate.md).

## Variables and secrets

- Determine whether the value is needed at build time, runtime, or both. Compose interpolation does not automatically inject a variable into a container; check `environment`, `env_file`, build arguments, and the app's configuration.
- Inspect key names and presence without displaying secret values. Use Dokploy's environment settings or the established secret store; do not commit private environment files or bake credentials into images. Frontend build variables may become public assets.
- Discover the target's save operation from the current schema. If it replaces an entire environment document, merge only requested keys in protected memory and preserve the rest. Do not guess an endpoint or assume null deletes a value.
- Request missing secret values through the available private secret-entry mechanism. Never deploy placeholder credentials. Supplying an application secret does not authorize changing the Dokploy API token.
- Verify key presence or application behavior without echoing credentials, then perform the deployment/rebuild required by the setting.

## Domains and networking

Check existing host ownership before creating a domain. For this instance retain `https: false` and `certificateType: "none"` in Dokploy, and verify public HTTPS. Do not change global proxy/Cloudflare settings to repair app routing.

Match the route to the actual service and internal listener. Distinguish container ports from published host ports; avoid exposing databases or admin services just to connect containers. Check the listener's bind address if routing cannot reach it.

Domain labels may require a Compose redeploy. Check DNS, route configuration, container health, and public HTTPS separately. Use the [domain payload and Secure-cookie guidance](dokploy-api.md#domain-routing-for-this-instance); an edge redirect loop is not a reason to enable origin TLS blindly.

## Source, build context, and storage

- Preserve source/provider, branch, Compose path, and generated command unless the request needs a change. Redeploying a Git branch does not necessarily deploy unpublished local edits; verify the source revision.
- For monorepos, identify the service and build context, including needed shared workspace packages and lockfiles. Raw Compose cannot build from files available only on the local machine.
- Use the returned `appName` for Compose identity. Changing it, volume names, or project scoping can attach an app to new empty storage without deleting old volumes.
- Identify persistent volumes, bind mounts, and owning services before changing storage. Explain migration, downtime, or data loss, and obtain confirmation for destructive changes. Volume deletion and server pruning are not ordinary configuration repairs.

Railway inspiration: [configuration workflow](https://github.com/railwayapp/railway-skills/blob/main/plugins/railway/skills/use-railway/references/configure.md). Dokploy contracts remain governed by [dokploy-api.md](dokploy-api.md), not Railway commands.
