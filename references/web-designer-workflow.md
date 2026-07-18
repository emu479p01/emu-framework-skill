# Web Designer workflow

Use this reference for prerequisite setup, human apply, and generated-App verification around the REST API workflow.

## Provision the AI-assisted workspace

Require the user to complete these steps in Web Designer:

1. Create or select the target App.
2. Create a dedicated Model and choose its Layer.
3. Create an ordinary dedicated AI-assistance user.
4. Add one `FW_AppAccess` row for the target App with `canOpen=true` and `canCustomize=true`.
5. Add only the least-privilege roles needed for later generated-App verification.

The user must give AI the exact endpoint, environment, App, Model, and Layer. AI verifies but does not create or alter these prerequisites.

Avoid `FW_SystemAdminRole` and `FW_FrameworkUser` for routine AI work because the v0.1.0.2 implementation gives them all-App Designer scope.

## Confirm the current design

The AI inspects capabilities and the App-scoped snapshot through REST. In Web Designer, the user can confirm:

- App identity and dependencies
- Model and layer ownership
- existing tables, enums, forms, menus, reports, Functions, Scripts, and security objects
- existing Extensions targeting the object
- visible naming and labeling conventions

Use the API schemas returned by the running instance as the machine contract and the UI state as human confirmation. Do not assume a control or field exists because it appears in a newer document.

## Review and apply a validated change set

When AI returns a valid preview:

1. Confirm the endpoint, environment, App, Model, Layer, and preview owner.
2. Review every diff, warning, schema effect, destructive flag, and high-risk item.
3. Submit apply as a human using the same dedicated username. Use Web Designer when the installed version exposes the preview; otherwise use a human-controlled API request.
4. Do not approve a changed, expired, or differently scoped preview.
5. Tell AI when apply completes so it can refresh the snapshot.

## Configure visible objects

For fields, configure the visible type, label, required/read-only behavior, create/update editability, default, and reference settings. For a reference, configure its target table, display fields, delete behavior, copy fields, and lookup filters when required.

For forms, configure list fields, filter fields, groups, master-detail lines, aggregates, and actions. A create-time Function action may not receive a `recordId`; design it around the current unsaved record values.

For menus, target an accessible Form, Function, Report, route, or submenu. Menu visibility follows effective permissions but does not replace authorization.

## Customize with Extensions

Use an Extension for an additive change to an existing object. The source layer must be strictly higher than the target layer. A cross-App Extension requires the dependency to be declared. Use the Designer-generated name unless the existing installation requires a preserved legacy identity.

Do not copy the base object into CUS merely to add a field or form action. Check whether the same App/Model already contributes an Extension of that kind to the target before creating another.

## Verify through the generated App

Open the App from normal navigation and test the user-visible workflow. Verify menus, list/detail pages, responsive layout, lookups, master-detail lines, reports, and actions affected by the change.

Test both allowed and denied roles. Direct navigation or action invocation must still be denied when a menu item or button is hidden. Use safe test records only in an authorized non-production context; otherwise limit verification to metadata and non-mutating UI behavior.

Return to Web Designer for corrections and repeat preview/apply. Report any UI control, native integration, destructive migration, or deployment requirement that Web Designer cannot safely handle.
