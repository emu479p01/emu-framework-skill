# Designer REST API workflow

Use this reference for secure authentication, discovery, artifact endpoints, atomic change sets, status codes, and Record API boundaries.

## Authenticate without exposing secrets

The default server URL is `http://localhost:3399` unless `PORT` or deployment routing changes it.

`POST /api/login` accepts `username` and `password`, then returns an HTTP-only session cookie. Keep one cookie jar or WebRequestSession for subsequent calls. Read credentials only from a user-prepared secure mechanism; never print them or place literal secrets in generated commands, files, logs, or chat.

An unauthenticated request is denied. A valid session still needs Designer/customize scope for metadata and table permissions for Record API calls.

## Run preflight discovery

Call in this order:

```text
GET /api/designer/capabilities
GET /api/designer/snapshot?app=<App>
```

Capabilities returns framework version, current revision, artifact/change-set schemas, preview TTL, human-confirmation requirements, and AI permissions. Treat it as the live API contract.

The App-scoped snapshot returns the visible App manifest and Designer artifacts. Verify the exact user-provided App, Model, and Layer. Stop if the App/Model is missing, the Layer differs, or the account sees an unexpected scope.

## Understand direct artifact endpoints

`POST /api/designer/artifacts` creates one complete supported artifact when AI mutation is permitted. It returns:

- `201` on success
- `400` for missing body or unsupported kind
- `403` outside the user's customization scope
- `409` when the global artifact name already exists
- `422` for schema, reference, registry, dependency, or layer validation failure

`PUT /api/designer/artifacts/:kind/:name` performs an idempotent upsert. URL `kind` and `name` are authoritative. Successful create/update normally returns `200`.

Changes rebuild runtime metadata and additive schema immediately without a restart. Do not use direct mutation endpoints when `capabilities.ai.apply=false`. Do not use a series of direct calls for a multi-artifact feature unless partial completion is acceptable and explicitly authorized.

Always include the fixed target `app`, `model`, and `layer` on artifacts that support them. Omitting `app` targets the default `web` scope and fails for an App-scoped `canCustomize` account.

## Prefer atomic change sets

Build this shape using the exact schema returned by capabilities:

```json
{
  "version": 1,
  "baseRevision": "<snapshot revision>",
  "source": "ai",
  "description": "Add the approved feature",
  "operations": [
    {
      "op": "upsert",
      "kind": "enum",
      "name": "SALES_OrderPriority",
      "artifact": {
        "kind": "enum",
        "name": "SALES_OrderPriority",
        "app": "sales",
        "model": "AICustomizations",
        "layer": "CUS",
        "values": [{ "name": "Normal", "value": 0 }]
      }
    }
  ]
}
```

Validate with `POST /api/designer/change-sets/validate`. Review `diagnostics`, `registryErrors`, `warnings`, `diff`, `schemaEffects`, `destructive`, `previewId`, and expiry.

Never change `source` from `ai` to `designer` to bypass executable-code restrictions. Re-fetch the snapshot and rebuild after a stale-revision error.

`POST /api/designer/change-sets/apply` requires a valid unexpired preview owned by the same username, `confirmation: true`, separate `confirmHighRisk: true` for high-risk items, and an unchanged base revision. Obey `capabilities.ai.apply`; when false, give the preview details to the human and require a human-controlled apply request under that same username. Web Designer may be used when the installed version exposes the preview.

## Separate Metadata and Record APIs

Use Designer endpoints for metadata. `POST /api/data/:table` creates a business Record and returns `201`, but the authenticated user's table `create` permission, field rules, hooks, and validation still apply.

Obey `capabilities.ai.businessData`. When false, do not call Record APIs. Even when true, require explicit authorization before creating test data and never use real production data casually.
