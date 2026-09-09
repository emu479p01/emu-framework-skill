# Metadata design

Use this reference for Apps, Models, layers, metadata identity, Extensions, and Web Designer change sets.

## Core model

```text
App
└── Model
    └── Layer
        └── Metadata artifacts
```

- App: top-level runtime access/navigation boundary plus identity, dependencies, display information, and Models.
- Model: coherent development-metadata group within an App; it is not a security boundary.
- Layer: ownership and precedence when multiple sources contribute to the same logical artifact.
- `name`: stable global identifier; do not treat it as display text.
- `label`: user-facing text.

Declare cross-app dependencies explicitly. App dependency order controls load order and whether cross-app Extensions are allowed.

Every App created in current versions starts with `models: []`, including names such as `erp`, `erp.credit`, and `web`. The user must add a Model and choose its Layer before creating any business artifact. Put the exact `app`, `model`, and matching `layer` on every supported artifact.

## Layer order

```text
SYS < ISV < LOC < DEV < CUS
```

| Layer | Owner and intended use |
| --- | --- |
| `SYS` | Framework system definitions |
| `ISV` | Reusable vendor/product definitions |
| `LOC` | Localization or site changes |
| `DEV` | Development-time customization |
| `CUS` | Customer/implementation customization |

A higher-layer base artifact replaces a lower-layer base artifact with the same logical identity. An Extension accumulates supported additions into its target.

## Extension rules

Use an Extension when a feature adds fields, indexes, enum values, form behavior, menu items, View columns, Chart measures, permissions, or Script/Function behavior and must remain independently removable.

Supported Extension kinds in v0.1.4.0 are:

```text
tableExtension, enumExtension, formExtension, menuExtension,
privilegeExtension, dutyExtension, roleExtension, scriptExtension,
viewExtension, chartExtension, functionExtension
```

Enforce all of these constraints:

- Give the Extension a unique, traceable stable name.
- Set its layer strictly higher than the target's layer.
- Declare a direct or transitive dependency when targeting another App.
- Allow only one Extension of a given kind for the same `(app, model, kind, target)`.
- Add only the required contributions; do not copy the base definition.
- Treat inherited layers as read-only and save only the current Layer delta.
- Target Form groups/actions/Charts/lines and menu items by stable ID when overriding labels, visibility, order, or icons.
- Test with the Extension enabled and disabled.
- Ensure removal leaves no orphaned references.

Canonical names use `<AppPrefix>_<ModelName>_<BaseName>_Extension`. The v0.1.4.0 migration may canonicalize an unambiguous legacy name; preserve any remaining warned legacy name until a human reviews the rename and references.

## Schema and storage safety

Supported base concepts include Apps, enums, tables, Forms, menus, Privileges, Duties, Roles, Scripts, Functions, Reports, Views, and Charts. Use only object kinds and properties exposed by the running Web Designer version.

Schema synchronization is additive for new tables, fields, and indexes. Removing or changing existing structures requires an explicit migration and verified backup. Never edit generated SQLite structure manually. Do not declare framework audit fields as application fields.

Enum and read-only fields must be optional. Trusted Functions and Scripts may populate read-only fields, but REST writes and generated Forms cannot edit them.

Artifact names are global identities during beta; avoid reusing a name across Apps.

## Atomic change-set flow

Use Web Designer's generated change set for dependent tables, Views, Charts, Forms, menus, and security artifacts inside the user-provided App/Model contract:

1. Inspect the current App, Model, layer, and objects before editing.
2. Build the smallest ordered set of Web Designer changes.
3. Save the draft and let Web Designer validate it.
4. Review the preview diff, warnings, and high-risk flags.
5. Apply the exact preview only with authority appropriate to the environment and risk.

## Design review

Before implementation, confirm:

- ownership App, Model, and layer
- stable naming prefix
- dependencies and target layers
- enum/table definitions before their consumers
- form/menu/action targets
- View sources, joins, parameters, grouping, Chart output fields, and Form parameter bindings
- security artifacts for each table operation, Function, Report, and View
- migration impact and rollback path

Framework/System metadata is visible only to a System Administrator as **Framework — Read-only**. It is never a valid mutation, package, or Extension target.
