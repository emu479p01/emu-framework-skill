---
name: build-emu-framework-apps
description: Operate an already installed and running EmuFramework instance through its Web Designer to create, customize, debug, and verify metadata-driven Apps. Use when the user wants AI to open an EmuFramework endpoint and build or modify Apps, Models, tables, enums, forms, menus, reports, security artifacts, Extensions, Scripts, Functions/actions, hooks, or related metadata in Web Designer. Also use to inspect change-set previews and test generated Apps. Do not use for installing EmuFramework or editing framework source unless the user explicitly asks.
---

# Build EmuFramework Apps in Web Designer

Develop against the user's running EmuFramework instance. Use Web Designer as the primary authoring surface and the generated App as the verification surface.

## Connect to the instance

1. Reuse an EmuFramework URL or relevant open browser tab already supplied for this task.
2. If none is available, ask for the EmuFramework base URL and whether the instance is development, staging, or production. This is required before UI work.
3. Open the URL with an available browser-control surface that can preserve the user's signed-in session.
4. If sign-in is required, ask the user to sign in in that browser and tell you when it is ready. Never request, inspect, or enter their password, token, cookie, setup code, or secret.
5. Confirm that the visible instance is the intended environment and that the signed-in user can open Web Designer for the target App.

Do not bypass Web Designer with CLI, source-file edits, direct database changes, or Metadata API mutations unless the user explicitly requests that different workflow.

## Establish the requested outcome

Inspect the current Web Designer state before asking questions that the UI can answer. Determine:

- whether to create a new App or customize an existing App
- target App, Model, and ownership layer
- stable naming prefix and user-facing labels
- tables, relationships, forms, menus, reports, and actions
- business rules and explicit user operations
- user personas and allowed table operations, Functions, reports, and App access
- acceptance cases, including denied and failure paths

Ask only for missing choices that materially change the design. State any consequential assumption before applying it.

## Load focused guidance

- Read [references/web-designer-workflow.md](references/web-designer-workflow.md) for browser navigation, Web Designer object order, Simple Builder, saving, previewing, and applying changes.
- Read [references/metadata-design.md](references/metadata-design.md) for Apps, Models, layers, naming, metadata identity, and Extensions.
- Read [references/business-logic.md](references/business-logic.md) for hooks, events, Scripts, Functions/actions, transactions, HTTP, or email.
- Read [references/security-and-verification.md](references/security-and-verification.md) for permissions, testing, schema risk, or release readiness.
- Read [references/official-sources.md](references/official-sources.md) only when visible behavior differs from the bundled baseline or a current framework fact must be verified.

Do not load every reference by default.

## Inspect before changing

Within Web Designer:

1. Select the target App and Model.
2. Inspect the App dependencies, Model layer, existing objects, and naming conventions.
3. Open each artifact that will be changed and identify its current effective definition.
4. Check whether an existing higher-layer Extension already targets it.
5. Avoid reading production business records unless verification requires specific test data and the user has authorized that access.

Prefer additive changes. Never recreate or copy a base artifact merely to add supported fields, layout, menu, security, or Script behavior.

## Design in dependency order

Plan the smallest complete object graph:

```text
App -> Model/layer -> Enums -> Tables/fields/indexes
    -> Forms/reports -> Menus -> Privileges -> Duties -> Roles -> App access
Tables -> Hooks/Scripts/Functions -> Privileges
```

For an existing App, use an Extension when the change is additive and independently removable. Put the Extension in a strictly higher layer and declare any required cross-App dependency.

Choose the smallest behavior mechanism:

- Use a field default or hook for record defaults and validation.
- Use a data event for lifecycle reactions.
- Use a Function for one explicit named operation.
- Use a Script for a small set of related registrations.
- Report that reviewed TypeScript is required when Web Designer cannot safely express complex native or reusable integration logic.

## Implement through Web Designer

1. Use Simple Builder when it can create the required table, form, and menu together.
2. Otherwise create objects in dependency order so each reference already exists.
3. Use names as stable identifiers and labels as display text.
4. Configure field types, requirements, references, delete behavior, lookups, form groups/lines/actions, menus, and reports from visible controls.
5. Add Privileges, Duties, Roles, and App access; do not rely on hidden menu items or buttons for authorization.
6. Add Scripts or Functions only after their tables and security design are known.
7. Save the draft and inspect the generated change-set preview, warnings, and high-risk flags.
8. Correct every unexpected dependency, layer, reference, schema, or permission issue before applying.

Do not invent UI controls or metadata fields that are not present in the running version. Reinspect the page after navigation or save because Web Designer state may change.

## Apply with the right authority

Treat the user's request to build or customize a named development/staging instance as authorization for in-scope, additive Web Designer changes when the preview contains no unexpected or high-risk diff.

Ask for explicit approval before applying when any of these is true:

- the instance is production or its environment is unknown
- the preview is destructive or high risk
- the change expands beyond the requested App or feature
- the target, layer, dependency, or migration effect is ambiguous

Never carry approval from one endpoint, environment, App, or preview to another.

## Verify the generated App

After applying:

1. Open the generated App through its normal navigation.
2. Verify menus, lists, forms, lookups, defaults, actions, reports, and responsive behavior relevant to the change.
3. Test valid input, validation failure, update/delete/reference behavior, and rollback.
4. Test with realistic allowed and denied roles; a System Administrator result does not prove least privilege.
5. Test an Extension enabled and disabled when the UI supports that lifecycle.
6. For async Functions, test service failures, timeouts, non-success responses, and database state around explicit transactions.
7. Return to Web Designer and fix defects through the same preview/apply workflow.

Do not create or alter real production business data for testing without explicit authorization.

## Report the result

Summarize the endpoint and environment, App/Model/layer, objects created or changed, Extension decisions, security coverage, preview/apply result, generated-App checks, and any remaining migration, backup, permission, or Web Designer limitation.
