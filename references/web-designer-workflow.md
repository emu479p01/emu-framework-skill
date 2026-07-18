# Web Designer workflow

Use this reference for browser interaction, object creation order, preview/apply behavior, and generated-App verification.

## Connect and authenticate

1. Use the EmuFramework base URL supplied by the user or an existing relevant browser tab.
2. Confirm whether the instance is development, staging, or production.
3. Navigate through the visible application shell; do not invent a Web Designer route.
4. If authentication is required, pause and ask the user to sign in in the selected browser.
5. Open Web Designer and confirm Designer permission for the target App.

Keep the user's signed-in browser session isolated to the requested endpoint. Do not inspect credentials, cookies, local storage, or unrelated tabs.

## Inspect the current design

Select the App and Model before editing. Inspect:

- App identity and dependencies
- Model and layer ownership
- existing tables, enums, forms, menus, reports, Functions, Scripts, and security objects
- existing Extensions targeting the object
- visible naming and labeling conventions

Use the UI state as the schema for the installed version. Do not assume a control exists because it appears in a newer document.

## Create in dependency order

Use this order so references resolve:

1. App
2. Model and layer
3. Enums
4. Tables, fields, references, and indexes
5. Forms and reports
6. Menus
7. Privileges
8. Duties
9. Roles
10. App access and user assignment
11. Scripts, Functions, hooks, and form actions

Use Simple Builder when a new table, form, and menu can be created together. Inspect every generated name and object before saving.

## Configure visible objects

For fields, configure the visible type, label, required/read-only behavior, create/update editability, default, and reference settings. For a reference, configure its target table, display fields, delete behavior, copy fields, and lookup filters when required.

For forms, configure list fields, filter fields, groups, master-detail lines, aggregates, and actions. A create-time Function action may not receive a `recordId`; design it around the current unsaved record values.

For menus, target an accessible Form, Function, Report, route, or submenu. Menu visibility follows effective permissions but does not replace authorization.

## Customize with Extensions

Use an Extension for an additive change to an existing object. The source layer must be strictly higher than the target layer. A cross-App Extension requires the dependency to be declared. Use the Designer-generated name unless the existing installation requires a preserved legacy identity.

Do not copy the base object into CUS merely to add a field or form action. Check whether the same App/Model already contributes an Extension of that kind to the target before creating another.

## Save, preview, and apply

1. Save the draft through Web Designer.
2. Read the generated change-set preview completely.
3. Verify object names, kinds, App/Model/layer, targets, dependencies, and security contributions.
4. Resolve warnings and inspect every destructive or high-risk flag.
5. Apply when the task and environment authorize it; otherwise request explicit approval with a concise preview summary.
6. Wait for completion and inspect the resulting Designer state.

Do not apply when the visible target differs from the requested endpoint, App, environment, or preview.

## Verify through the generated App

Open the App from normal navigation and test the user-visible workflow. Verify menus, list/detail pages, responsive layout, lookups, master-detail lines, reports, and actions affected by the change.

Test both allowed and denied roles. Direct navigation or action invocation must still be denied when a menu item or button is hidden. Use safe test records only in an authorized non-production context; otherwise limit verification to metadata and non-mutating UI behavior.

Return to Web Designer for corrections and repeat preview/apply. Report any UI control, native integration, destructive migration, or deployment requirement that Web Designer cannot safely handle.
