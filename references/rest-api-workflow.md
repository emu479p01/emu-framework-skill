# REST API workflow

Use this reference for authentication, discovery, change-set validation, proposals, status codes, and Record API boundaries. Endpoint paths below were verified against EmuFramework v1.4.0 (`server.ts`, `designer.ts`, `aiApi.ts`). The running instance's capabilities remain authoritative.

## Choose one path

| | Designer API | AI REST proposal API |
| --- | --- | --- |
| Base path | `/api/designer/*` | `/api/v1/ai/*` |
| Credential | Session cookie from `POST /api/login` for the dedicated user | `Authorization: Bearer emu_ai_<secret>` |
| Scope | Apps where the user has `canCustomize` | Apps, scopes, and expiry fixed at token creation |
| AI can apply? | No. `capabilities.ai.apply=false` | No. There is no apply endpoint |
| Executable artifacts | Rejected for `source: "ai"` (`ai.scripts=false`) | Allowed in proposals as reviewed code (`executableArtifacts: true`) |
| Human step | Apply with `confirmation: true` as the same username that validated | Review and approve in **Web Designer, AI Proposals** |

Use the path the user provides. Do not mix credentials, and do not request a second credential to escape a limit. Prefer the AI REST path when a token exists: proposals are persistent, audited, and reviewable by any customizer with access to every affected App.

Neither path may create Apps, change Model definitions, apply changes, or read business records.

## Authenticate without exposing secrets

Use the base URL the user supplies. The framework default is `http://localhost:3399` unless `PORT` or the Docker port mapping changes it. Production deployment is Docker-only.

- Designer path: `POST /api/login` with `username` and `password` returns an HTTP-only session cookie. Keep one cookie jar. Read credentials only from a user-prepared secure mechanism; never print them or place literals in commands, files, logs, or chat.
- AI path: read the token from an environment variable or secret store and send it only in the Bearer header. Never place it in a URL, file, log, or chat. A System Administrator creates it under **Settings, Users & Security, AI REST tokens**; the secret is shown once and only its SHA-256 hash is stored. If it may have leaked, tell the user to revoke it and create a replacement.
- An unauthenticated or invalid request is denied (`401`). A valid session still needs `canCustomize` (or `FW_SystemAdminRole`) for Designer metadata; otherwise `403`.

Never call `/api/system/ai-tokens*`, `/api/designer/ai-proposals/:id/approve`, `/reject`, or `DELETE /api/designer/ai-proposals/:id`. Token management is administrator-only, and approval or removal of a proposal is the human review step even if the session technically permits it.

## Run preflight discovery

Designer path, in order:

```text
GET /api/designer/capabilities
GET /api/designer/snapshot?app=<App>
```

- `capabilities` returns `version`, `revision`, `changeSets` (`version`, `previewTtlSeconds`, `humanConfirmationRequired`), the `ai` flags (`inspect`, `validate`, `apply`, `businessData`, `scripts`), and the `schemas.artifact` and `schemas.changeSet` JSON Schemas.
- `snapshot?app=` returns `revision`, the visible App manifests in `apps`, and the Designer-stored `artifacts` for that App. A request for an App outside the account's scope returns empty `apps` and `artifacts` instead of an error; treat that as a scope mismatch and stop.

Optional Designer reads: `GET /api/designer/artifacts?app=&model=&kind=&cursor=&limit=&includeCatalog=false` (paginated, ETag, `revision` query for `304`; follow `nextCursor`), `GET /api/designer/catalog?app=&kind=`, and `GET /api/designer/customization/:kind/:name?app=&model=` (inherited layers and the effective merged artifact for table, enum, form, report, view, chart, menu, privilege, duty, role, script, function, dataEntity; layers marked `editable` are the only ones the fixed scope may touch).

AI path, in order:

```text
GET /api/v1/ai/capabilities              (inspect)
GET /api/v1/ai/schemas/artifact          (inspect)
GET /api/v1/ai/schemas/change-set        (inspect)
GET /api/v1/ai/workspace?app=&model=&kind=&cursor=&limit=   (inspect)
```

- `capabilities` returns `version`, `changeSetVersion`, the token's `scopes` and `apps`, `apply: false`, `businessData: false`, and `executableArtifacts`. Stop if the target App is not in `apps` or the needed scope (`validate`, `propose`) is missing.
- `workspace` returns `revision`, `artifacts`, `nextCursor`, `total`. Default page 100, maximum 500. Follow `nextCursor` until `null`; the first page is not the whole workspace. Request with `app=<App>` and `model=<Model>` filters. It lists Designer-stored artifacts only; framework or file-based artifacts are not included, so a missing base artifact means stop and ask.

Verify the App exists, the Model exists in it, the Model Layer equals the user's Layer, and nothing outside the scope contract is needed. Stop on any mismatch.

## Identity and placement

Every non-App artifact must carry `kind`, `name`, `app`, and `model`; include `layer` equal to the Model's Layer (the effective Layer always comes from the Model, so changing the payload's `layer` alone never moves an artifact). Names match `^[A-Za-z_][A-Za-z0-9_.-]*$`, start with the App prefix (`sales` uses `SALES_`; `erp.credit` uses `ERP_`), and are global identities. `kind` cannot change after creation. Schemas set `additionalProperties: false`, so a misspelled property is an error. A missing or unknown Model returns `422`.

## Build and validate a change set

```json
{
  "version": 1,
  "baseRevision": "<revision from snapshot or workspace>",
  "source": "ai",
  "description": "Add the approved feature",
  "operations": [
    {
      "op": "upsert", "kind": "enum", "name": "SALES_OrderPriority",
      "artifact": {
        "kind": "enum", "name": "SALES_OrderPriority",
        "app": "sales", "model": "AICustomizations", "layer": "CUS",
        "values": [{ "name": "Normal", "value": 0 }]
      }
    }
  ]
}
```

- Operations are `upsert` (artifact identity must equal the operation's `kind`/`name`) or `delete`. Include every new dependency in the same set.
- Always set `source: "ai"`. Never use `designer` or `cli` to get past the executable-artifact restriction.
- Designer path: `POST /api/designer/change-sets/validate`. A valid response carries `previewId`, `expiresAt` (10 minutes), `diff`, `schemaEffects`, `warnings`, `diagnostics`, `registryErrors`, and the next revision. Invalid returns `422`.
- AI path: `POST /api/v1/ai/change-sets/validate` returns `200` for valid and `422` for invalid, with the safe diff, but no `previewId`.
- Diagnostics carry a JSON path and code. A `stale_revision` code means refetch the revision and rebuild. Review `highRisk` items: delete of a table or App, and every Script, Function, or their Extensions.
- Validation does not execute or compile Script or Function code (bodies are blanked for the preview). Review that code yourself for syntax and for the synchronous-handler rule in [business-logic.md](business-logic.md).
- Do not use `DELETE` operations or table removal unless the user explicitly asked; deleting metadata leaves physical tables as orphans, and purging is a Framework Administrator action.

## Submit for human review

AI path: `POST /api/v1/ai/proposals` (needs `propose`) with the same validated change set. It returns `201` with `{ id, status: "pending", preview }`, or `422` with diagnostics. Give the user the proposal `id`, the diff summary, warnings, and high-risk items. A customizer with `canCustomize` on every affected App approves or rejects it in **Web Designer, AI Proposals** (the Proposal Inbox). Approval revalidates against the current workspace; a stale revision returns `409`. Rebuild and resubmit after any rejection or conflict.

Designer path: give the user the `previewId`, expiry, and diff. The human submits `POST /api/designer/change-sets/apply` as the same username with `confirmation: true` (and `confirmHighRisk: true` when any item is high-risk) before expiry. Errors: `400` missing confirmation, `403` another user's preview, `409` changed workspace, `410` expired preview.

If `ai.apply` is ever reported `true`, still require explicit user approval for production, unknown environments, destructive changes, or scope expansion.

## Direct artifact endpoints

`POST /api/designer/artifacts` (create-only, `201`, `409` duplicate), `PUT /api/designer/artifacts/:kind/:name` (idempotent upsert; URL kind and name win; send the complete artifact), and `DELETE /api/designer/artifacts/:kind/:name` exist on the Designer path. Do not call them: AI apply is disabled in 1.4.0, and a series of direct calls leaves partial designs. Use a change set. Model definitions (`PUT/DELETE /api/designer/artifacts/model/:app/:model`) belong to the user.

## Status codes

| Status | Meaning |
| ---: | --- |
| `200`/`201` | OK / created (artifact or proposal) |
| `400` | Missing input, missing confirmation, unknown scope, or unsupported kind |
| `401` | Missing, invalid, expired, or revoked session or token |
| `403` | No Customize/App access, token lacks scope or App, or Framework metadata (read-only) |
| `404` | Unknown route resource |
| `409` | Duplicate, stale workspace or proposal, concurrent job |
| `410` | Preview expired |
| `422` | Schema, placement, or registry validation failed; read `diagnostics` and `registryErrors` |

## Separate metadata and Record APIs

Use Designer or AI endpoints for metadata. `ai.businessData=false` always holds in 1.4.0 and the AI token has no record endpoint, so do not call `/api/data/*` at all unless the user separately authorizes a verification step and a session with runtime permissions. When allowed, `POST /api/data/:table/drafts` plus `POST /api/data/:table/drafts/:token/save` is the create flow the generated UI uses; `POST /api/data/:table` also creates a record. Hooks, field rules, and permissions still apply, and you must never use real production data casually. Never call business-data exchange routes either: `/api/data-entities/*`, `/api/attachments/*`, import/export, archive, or data-management routes.

Never call generic Data, import, or export endpoints for security and framework tables: `FW_User`, `FW_UserRole`, `FW_AppAccess`, `FW_Session`, `FW_WebArtifact`, `FW_NavigationItem`, `FW_Blob`, `FW_Attachment`, `FW_Migration`, `FW_ViewToken`, `FW_ViewTokenScope`, `FW_DataJob`, `FW_ArchivePolicy`, or the AI token and proposal tables. User administration, password changes, View-token lifecycle, license import, and AI-token lifecycle use dedicated human-administered screens.
