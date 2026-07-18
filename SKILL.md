---
name: build-emu-framework-apps
description: Develop and customize Apps in an already installed and running EmuFramework instance through its Web Designer REST API, with Web Designer used for human setup, approval, and visual verification. Use when AI must inspect Designer capabilities and snapshots, design or validate metadata change sets, create or extend tables, enums, forms, menus, reports, security artifacts, or supported behavior inside a user-provided App, Model, and Layer. Require the user to pre-create the App, Model/Layer, and a dedicated App-scoped customization account. Do not use for installing EmuFramework or editing framework source.
---

# Build EmuFramework Apps through the Designer API

Use EmuFramework's REST API as the primary AI development surface. Use Web Designer for prerequisite setup, human confirmation where required, and visual verification of the generated App.

## Require a complete target contract

Before any API work, require the user to provide all of these:

- EmuFramework base URL and environment: development, staging, or production
- exact App name
- exact Model name
- exact Model Layer: `SYS`, `ISV`, `LOC`, `DEV`, or `CUS`
- confirmation that the App and Model/Layer already exist
- confirmation that a dedicated AI-assistance user already exists
- confirmation that this user has `FW_AppAccess.canOpen=true` and `canCustomize=true` for the target App
- a secure authenticated session or credential mechanism outside the conversation

If any item is missing, stop and give the user the setup checklist. Do not create the App, Model, Layer, user, role assignment, or App Access row on the user's behalf.

Treat App, Model, and Layer as a fixed scope contract. Never infer, substitute, create, rename, or cross into another scope without the user updating that contract explicitly.

## Require a least-privilege AI user

Tell the user to create an ordinary dedicated account for AI-assisted customization and scope it through `FW_AppAccess` to the target App.

- Set both `canOpen` and `canCustomize` so the account can inspect Designer metadata and verify the generated App.
- Do not recommend `FW_SystemAdminRole` for routine AI work.
- Do not recommend `FW_FrameworkUser` for routine AI work in v0.1.0.2 because the implementation grants that role all-App Designer scope even though the security documentation describes it as self-service.
- Grant additional table, Function, or report privileges only when a named verification task requires them.

Never request or expose passwords, cookies, tokens, setup codes, or secret values. Prefer an existing signed-in browser session or credentials configured in environment variables or a secret store that the HTTP client can use without printing them.

## Discover capabilities before designing

After authentication, make only read-only requests first:

1. `GET /api/designer/capabilities`
2. `GET /api/designer/snapshot?app=<App>`

Use the capabilities response as the live contract for framework version, artifact schema, change-set schema, revision, and AI permissions. Obey every `ai` capability flag.

Verify from the snapshot that:

- the exact App exists
- the exact Model exists in that App
- the Model's Layer equals the user-provided Layer
- the authenticated account can see only the intended customization scope

Stop on any mismatch. Do not repair prerequisites through the API.

## Load focused guidance

- Read [references/rest-api-workflow.md](references/rest-api-workflow.md) for authentication, capabilities, snapshots, artifact endpoints, change sets, status codes, and Record API boundaries.
- Read [references/web-designer-workflow.md](references/web-designer-workflow.md) for user provisioning, App/Model/Layer setup, human apply, and visual verification.
- Read [references/metadata-design.md](references/metadata-design.md) for metadata identity, layers, naming, and Extensions.
- Read [references/business-logic.md](references/business-logic.md) for hooks, Scripts, Functions, transactions, HTTP, or email.
- Read [references/security-and-verification.md](references/security-and-verification.md) for least privilege, testing, schema risk, or release readiness.
- Read [references/official-sources.md](references/official-sources.md) only when runtime behavior differs from the bundled baseline.

Do not load every reference by default.

## Inspect the target before proposing changes

Use the App-scoped snapshot and effective metadata to inspect existing objects, dependencies, naming conventions, layers, and Extensions. Read only artifacts relevant to the requested feature.

Never query business records merely to understand metadata. Do not use `/api/data/:table` unless a separate, explicitly authorized verification step and the runtime's AI capabilities allow business-data access.

## Design a scoped change set

Create the smallest complete artifact graph in dependency order:

```text
Enums -> Tables/fields/indexes -> Forms/reports -> Menus
      -> Privileges -> Duties -> Roles
Tables -> Extensions/hooks/Functions -> Privileges
```

The App and Model already exist and are not operations in the change set. Put the exact user-provided `app`, `model`, and `layer` on every artifact that supports them.

Prefer an Extension for additive customization of an existing artifact. Require the Extension Layer to be strictly higher than the target Layer and preserve the App dependency boundary. Never copy a base artifact merely to add supported fields, layout, menu, security, or Script behavior.

Obtain exact artifact shapes from `/api/designer/capabilities`; never guess fields from bundled examples.

## Validate through the REST API

Use a version-1 MetadataChangeSet with the snapshot revision and `source: "ai"`. Never claim `source: "designer"` for AI-generated operations.

1. Build ordered `upsert` operations with matching operation and artifact identities.
2. `POST /api/designer/change-sets/validate`.
3. Resolve all diagnostics and registry errors.
4. Review warnings, diff, schema effects, destructive flags, and every high-risk item.
5. Revalidate after any change or stale revision.

Prefer this atomic preview flow over repeated `POST /api/designer/artifacts` calls. Direct create/upsert endpoints are supported, but use them only when the user explicitly requests that integration path, `ai.apply` or an equivalent runtime policy permits AI mutation, and the risk policy permits mutation without a multi-object preview.

## Respect runtime AI limits

If capabilities report `ai.apply=false`, do not call `/api/designer/change-sets/apply` or mutate through direct artifact POST/PUT/DELETE endpoints. Give the user the preview ID, expiry, and a concise diff/risk summary. Require the human to submit the apply confirmation using the same dedicated username, either through Web Designer when that version exposes the preview or through a human-controlled API request. Continue only after the user confirms that apply completed.

If capabilities report `ai.scripts=false`, do not create a Script, Script Extension, or Function through an AI change set and never relabel the source to bypass the restriction. Provide the reviewed design/code for the user to enter and approve through Web Designer.

If capabilities report `ai.businessData=false`, do not call `/api/data/:table`, including for test records.

For any future runtime that enables an AI mutation capability, still require explicit approval for production, unknown environments, destructive/high-risk changes, or scope expansion.

## Verify after human apply

After the user confirms apply:

1. Fetch a fresh App-scoped snapshot and confirm the revision changed as expected.
2. Verify the effective metadata contains only the intended artifacts and contributions.
3. Open the generated App using the dedicated account.
4. Verify menus, lists, forms, lookups, defaults, actions, reports, and responsive behavior affected by the change.
5. Test allowed and denied roles only with authorization and without creating real production data.
6. Return to validation for any correction; never patch the database directly.

## Report the result

Summarize the endpoint/environment, fixed App/Model/Layer contract, authenticated account scope, capabilities observed, artifacts proposed, validation result, human-apply status, effective metadata verification, UI verification, and any remaining permission, migration, backup, or capability limitation.
