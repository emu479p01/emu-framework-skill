---
name: build-emu-framework-apps
description: Develop and customize Apps in an already installed and running EmuFramework 1.x instance through its Designer REST API or its scoped AI REST proposal API, with Web Designer used for human setup, review, approval, and visual verification. Use when AI must inspect capabilities and metadata, then validate or propose change sets for tables, enums, forms, menus, Views, Charts, reports, translations, Data Entities, security artifacts, or supported behavior inside a user-provided App, Model, and Layer. Require the user to pre-create the App, Model/Layer, and a dedicated least-privilege account or AI token. Do not use for installing or upgrading EmuFramework, editing framework source, deployment packages, licenses, or business data.
---

# Build EmuFramework Apps through the Designer and AI REST APIs

Use EmuFramework's REST APIs as the AI development surface: the Designer API with a customization session, or the AI REST proposal API with a scoped token. Neither applies changes. Use Web Designer for prerequisite setup, human apply or approval, and visual verification of the generated App. Targets EmuFramework v1.4.0; the version reported by the running instance is authoritative.

## Require a complete target contract

Before any API work, require the user to provide all of these:

- EmuFramework base URL and environment: development, staging, or production
- exact App name
- exact Model name
- exact Model Layer: `SYS`, `ISV`, `LOC`, `DEV`, or `CUS`
- confirmation that the App and Model/Layer already exist
- which credential path applies: a dedicated customization user, or an AI REST token created by a System Administrator for the target App
- for a user: confirmation that it has `FW_AppAccess.canCustomize=true` for the target App
- for a token: its scopes (`inspect`, `validate`, `propose`) and confirmation that the target App is in its App list
- when runtime/business-data verification is requested, confirmation that the user separately has `canOpen=true` and the minimum Role/Privilege permissions for that verification
- a secure authenticated session, or a token held in an environment variable or secret store, outside the conversation

If any item is missing, stop and give the user the setup checklist. Do not create the App, Model, Layer, user, role assignment, App Access row, or token on the user's behalf.

Treat App, Model, and Layer as a fixed scope contract. A token or customize permission scopes only to Apps, so you are the Model/Layer guard. Never infer, substitute, create, rename, or cross into another scope without the user updating that contract explicitly.

## Require least-privilege credentials

- Customization user: ordinary dedicated account, `canCustomize=true` for the target App only. `canOpen=false` unless the task explicitly includes generated-App verification; `canCustomize` never grants runtime or business-data access.
- AI REST token: existing non-system Apps only, minimum scopes, short expiry. It cannot apply changes or read records.
- When runtime verification is required, add `canOpen=true` plus the smallest Role/Duty/Privilege set for the specific Forms, tables, Functions, Reports, or Views being tested.
- Do not recommend `FW_SystemAdminRole` for routine AI work. `FW_FrameworkUser` is a legacy marker and grants no Designer bypass.
- Never use generic Data or import/export endpoints for `FW_User`, `FW_UserRole`, `FW_AppAccess`, sessions, migration records, Designer storage, AI tokens, or View tokens.

Never request or expose passwords, cookies, tokens, setup codes, secret keys, license files, or license and seller keys. Prefer an existing signed-in browser session or credentials configured where the HTTP client can use them without printing them.

## Discover capabilities before designing

After authentication, make only read-only requests first.

- Designer path: `GET /api/designer/capabilities`, then `GET /api/designer/snapshot?app=<App>`.
- AI path: `GET /api/v1/ai/capabilities`, the artifact and change-set schemas under `/api/v1/ai/schemas/`, then every page of `GET /api/v1/ai/workspace?app=<App>&model=<Model>` until `nextCursor` is `null`.

Use the capabilities response as the live contract for framework version, artifact schema, change-set schema, revision, and permissions. Obey every flag. If the reported version is not 1.4.0, follow the instance and drop features it lacks.

Verify the exact App exists, the exact Model exists in that App, the Model's Layer equals the user-provided Layer, and the account or token sees only the intended scope. Treat **Framework — Read-only** metadata as inspection-only. Stop on any mismatch; never repair prerequisites through the API.

## Load focused guidance

- [references/rest-api-workflow.md](references/rest-api-workflow.md): authentication, both API paths, endpoints, change sets, proposals, status codes, Record API boundaries.
- [references/web-designer-workflow.md](references/web-designer-workflow.md): provisioning, human apply, Proposal Inbox review, visual verification.
- [references/metadata-design.md](references/metadata-design.md): identity, layers, Extensions, field rules, licensed Models.
- [references/localization-and-data-entities.md](references/localization-and-data-entities.md): `defaultLocale`, `translation`, `dataEntity`, `dataEntityExtension`.
- [references/reports.md](references/reports.md): paper, units, layout version 2, borders, images, assets.
- [references/business-logic.md](references/business-logic.md): hooks, drafts, Scripts, Functions, `imageInput`, services. Load before writing any code.
- [references/views-and-charts.md](references/views-and-charts.md): Views, View privileges, Charts, Form embedding, service-token boundaries.
- [references/security-and-verification.md](references/security-and-verification.md): least privilege, secrets and licenses, testing, release readiness.
- [references/official-sources.md](references/official-sources.md): only when runtime behavior differs from the bundled 1.4.0 baseline.

Do not load every reference by default.

## Inspect the target before proposing changes

Use the snapshot or workspace, and `GET /api/designer/customization/:kind/:name` when available, to inspect existing objects, dependencies, naming conventions, layers, Extensions, and existing Scripts. Read only artifacts relevant to the request.

Never query business records merely to understand metadata. Do not use `/api/data/*`, `/api/data-entities/*`, or attachment, archive, or import/export endpoints unless a separate, explicitly authorized verification step allows business-data access.

## Design a scoped change set

Create the smallest complete artifact graph in dependency order:

```text
Enums -> Tables/fields/indexes -> Views -> Charts -> Forms/reports -> Menus
      -> Privileges (including Views) -> Duties -> Roles
Tables -> Extensions/hooks/Functions -> Privileges
Tables -> Data Entities -> Data Entity Extensions
Everything labeled -> Translations (last, so keys target live artifacts)
```

The App and Model already exist and are not operations. Put the exact `app`, `model`, and `layer` on every artifact that supports them. Do not upsert the App manifest or a Model definition; ask the user to change the App `defaultLocale`, Model `license`, dependencies, or Layer.

Prefer an Extension for additive customization of an existing artifact: Layer strictly higher than the target, App dependency declared, current Layer delta only, stable IDs for presentation overrides (including Form `lineOverrides`). Never copy a base artifact merely to add supported behavior. Supported Extension kinds are table, enum, form, menu, privilege, duty, role, script, view, chart, function, and dataEntity; there is no report or translation Extension.

Apply these field and metadata rules (details in the references):

- Never define `sys_createdBy`, `sys_createdAt`, `sys_modifiedBy`, or `sys_modifiedAt` as fields; they are virtual and already usable in Views, Queries, filters, and sorting. System field names are reserved case-insensitively.
- `multiline` and `encrypted` apply only to `string` fields; encrypted fields cannot be defaulted, indexed, titled, filtered, viewed, or reported.
- Reject `mandatory: true` on Enum fields. Treat a read-only mandatory field as unsupported when the live validator returns `readonly_optional`, even though 1.4.0 documents it as allowed when `initValue` supplies the value.
- New labels in an App with several locales get a Translation; never write `ui.*` keys.
- Write only the report geometry the page validator accepts; emit `layoutVersion: 2`.
- Lifecycle hooks and data event handlers are synchronous. Use an async Function for awaited work.

For a View, grant the View name and read permission to every source table. For a Chart, secure its referenced View. Obtain exact artifact shapes from the live schema; never guess fields from bundled examples.

## Validate through the REST API

Use a version-1 MetadataChangeSet with the current revision and `source: "ai"`. Never claim `source: "designer"` for AI-generated operations.

1. Build ordered `upsert` operations with matching operation and artifact identities.
2. Validate: `POST /api/designer/change-sets/validate` or `POST /api/v1/ai/change-sets/validate`.
3. Resolve all diagnostics and registry errors.
4. Review warnings, diff, schema effects, and every high-risk item.
5. Revalidate after any change or `stale_revision`.

Validation does not run Script or Function code. Review it yourself.

## Respect runtime limits and hand off to a human

AI cannot apply on either path. Do not call `/api/designer/change-sets/apply`, direct artifact POST/PUT/DELETE, or the proposal approve, reject, or delete endpoints.

- AI path with `propose`: submit the validated set to `POST /api/v1/ai/proposals`, then give the user the proposal ID and a concise diff and risk summary. A customizer approves it in **Web Designer, AI Proposals**.
- Designer path: give the user the preview ID, expiry, and diff. The human applies it as the same username within the 10-minute window.
- Continue only after the user confirms apply or approval.
- If `ai.scripts=false` (Designer path), do not create a Script, Script Extension, Function, or Function Extension and never relabel the source to bypass it. Provide the reviewed code for the user to enter through Web Designer. On the AI path, propose executable artifacts only when `executableArtifacts` is true and the task needs them; they are high-risk and must be reviewed as code.
- If `ai.businessData=false`, never call record endpoints, including for test data.
- Require explicit approval for production, unknown environments, destructive or high-risk changes, and scope expansion.

## Stay inside the scope contract

Do not, unless the user explicitly asks and the credentials allow it:

- build, export, import, or preview deployment packages (selected-model schema v2, App or Model packages); these are administrator workflows
- handle ISV license files, license keys, vendor or seller private keys, installation IDs, or the seller CLI, or change a Model's `license`
- change App `defaultLocale`, dependencies, Model Layers, users, role assignments, App Access, AI tokens, SMTP, fonts, archive policies, or System Maintenance
- run Data Entity import or export, archive, attachment, or data-management operations

Unauthorized actions are omitted from `/api/metadata`, and a licensed Model goes read-only when its license expires, so a failed verification can come from permissions or license state rather than your metadata. Report it; do not work around it.

## Verify after human apply

After the user confirms apply or approval:

1. Fetch a fresh App-scoped snapshot or workspace and confirm the revision changed as expected.
2. Verify the effective metadata contains only the intended artifacts and contributions.
3. If runtime verification was explicitly authorized, open the generated App using an account that has both `canOpen` and the required object privileges.
4. Verify menus, lists, forms, drafts and `initValue` results, lookups, defaults, actions, reports, translations, and responsive behavior affected by the change.
5. Test allowed and denied roles only with authorization and without creating real production data.
6. Return to validation for any correction; never patch the database directly.

## Report the result

Summarize the endpoint/environment and framework version, the credential path, the fixed App/Model/Layer contract, observed capabilities, artifacts proposed, validation result, human apply or approval status, effective metadata verification, UI verification, license or permission findings, and any remaining migration, backup, or capability limitation.
