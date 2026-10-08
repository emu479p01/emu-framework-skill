# Web Designer workflow

Use this reference for prerequisite setup, human apply, and generated-App verification around the REST API workflow.

## Provision the AI-assisted workspace

Require the user to complete these steps in Web Designer:

1. Create or select the target App. A new App starts with `models: []`; no App name creates a default Model.
2. Add a dedicated Model and choose its Layer explicitly.
3. Create an ordinary dedicated AI-assistance user (Designer path), or an AI REST token (AI path, see below).
4. For the user, add one `FW_AppAccess` row for the target App with `canCustomize=true`. A token needs no App Access row; its App list is chosen at creation.
5. Only when runtime verification is requested, add `canOpen=true` and the least-privilege Roles needed for the named verification.

The user must give AI the exact endpoint, environment, App, Model, and Layer. AI verifies but does not create or alter these prerequisites.

Avoid `FW_SystemAdminRole` and `FW_FrameworkUser` for routine AI work. System Administrator is the only global bypass and `FW_FrameworkUser` is a legacy marker.

To use the AI REST path instead of a session, a System Administrator creates a token under **Settings, Users & Security, AI REST tokens**: existing non-system Apps only, minimum scopes (`inspect`, `validate`, `propose`), and a short expiry. The secret is shown once and goes straight into a secret store, never into a prompt.

## Confirm the current design

The AI inspects capabilities and the App-scoped snapshot through REST. In Web Designer, the user can confirm:

- App identity and dependencies
- Model and layer ownership
- existing tables, enums, Forms, Views, Charts, menus, Reports, Functions, Scripts, Translations, Data Entities, and security objects
- existing Extensions targeting the object
- visible naming and labeling conventions
- the App's Default language and available locales

Use the API schemas returned by the running instance as the machine contract and the UI state as human confirmation. Do not assume a control or field exists because it appears in a newer document.

## Review and apply a validated change set

When AI returns a valid preview or proposal:

1. Confirm the endpoint, environment, App, Model, Layer, and preview owner or proposal token.
2. Review every diff, warning, schema effect, destructive flag, and high-risk item.
3. Designer path: submit apply as a human using the same dedicated username, through Web Designer when the installed version exposes the preview, otherwise through a human-controlled API request.
4. AI REST path: open **Web Designer, AI Proposals**, filter to pending, expand the proposal, review the complete diff, and approve or reject. The reviewer needs Customize permission for every affected App. Approval revalidates against the current workspace; a stale revision returns a conflict and the AI rebuilds. Reviewed proposals can be removed from the Inbox without removing applied metadata or audit records.
5. Do not approve a changed, expired, or differently scoped preview or proposal.
6. Tell AI when apply or approval completes so it can refresh the snapshot.

## Configure visible objects

For fields, configure the visible type, label, required/read-only behavior, create/update editability, default, and reference settings. For a reference, configure its target table, display fields, delete behavior, copy fields, and lookup filters when required.

For forms, configure list fields, filter fields, groups, master-detail lines, aggregates, and actions. A create-time Function action may not receive a `recordId`; design it around the current unsaved record values.

For menus, target an accessible Form, Function, Report, route, or submenu. Menu visibility follows effective permissions but does not replace authorization.

For App-level settings the AI must not change through a ChangeSet, ask the user to use the Designer: the App **Default language**, Model definitions and Layers, `license` requirements, and dependencies. For translations, use the Designer's Translation editor or a `translation` artifact; its key list is generated from the same builders as the runtime resolver. Selected-model deployment packages, App/Model export and import, Apps & Models inventory, license import, Storage & Archive, and System Maintenance are administrator tasks that the AI does not perform unless the user explicitly asks.

## Customize with Extensions

Use an Extension for an additive change to an existing object, including Data Entities and inherited Form Lines. The source layer must be strictly higher than the target layer. A cross-App Extension requires the dependency to be declared. Use the Designer-generated name unless the existing installation requires a preserved legacy identity.

Do not copy the base object into CUS merely to add a field or form action. Check whether the same App/Model already contributes an Extension of that kind to the target before creating another.

## Verify through the generated App

When runtime verification was explicitly authorized, open the App from normal navigation using an account with both `canOpen` and the required object privileges. Verify menus, list/detail pages, embedded Charts, responsive layout, lookups, master-detail lines, Reports, and actions affected by the change.

Test both allowed and denied Roles. Direct navigation, View calls, or action invocation must still be denied when a menu item, Chart, or button is hidden. Use safe test records only in an authorized non-production context; otherwise limit verification to metadata and non-mutating UI behavior.

Return to Web Designer for corrections and repeat preview/apply. Report any UI control, native integration, destructive migration, or deployment requirement that Web Designer cannot safely handle.
