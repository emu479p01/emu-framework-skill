# Metadata design

Use this reference for Apps, Models, layers, metadata identity, Extensions, schemas, and change sets.

## Core model

```text
App
└── Model
    └── Layer
        └── Metadata artifacts
```

- App: top-level identity, dependencies, display information, and Models.
- Model: coherent group of definitions within an App.
- Layer: ownership and precedence when multiple sources contribute to the same logical artifact.
- `name`: stable global identifier; do not treat it as display text.
- `label`: user-facing text.

Declare cross-app dependencies explicitly. App dependency order controls load order and whether cross-app Extensions are allowed.

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

Use an Extension when a feature adds fields, indexes, enum values, form behavior, menu items, permissions, or Script behavior and must remain independently removable.

Supported v0.1.0.2 kinds are:

```text
tableExtension, enumExtension, formExtension, menuExtension,
privilegeExtension, dutyExtension, roleExtension, scriptExtension
```

Enforce all of these constraints:

- Give the Extension a unique, traceable stable name.
- Set its layer strictly higher than the target's layer.
- Declare a direct or transitive dependency when targeting another App.
- Allow only one Extension of a given kind for the same `(app, model, kind, target)`.
- Add only the required contributions; do not copy the base definition.
- Test with the Extension enabled and disabled.
- Ensure removal leaves no orphaned references.

Recent canonical names use `<AppPrefix>_<ModelName>_<BaseName>_Extension`. Preserve legacy names rather than silently renaming them.

## Schema and storage safety

Supported base concepts include apps, enums, tables, forms, menus, privileges, duties, roles, Scripts, Functions, and reports. Obtain the exact kinds and properties from the live schema.

Schema synchronization is additive for new tables, fields, and indexes. Removing or changing existing structures requires an explicit migration and verified backup. Never edit generated SQLite structure manually. Do not declare framework audit fields as application fields.

Artifact names are global identities during beta; avoid reusing a name across Apps.

## Atomic change-set flow

Use a change set for an App plus its dependent tables, forms, menus, and security artifacts:

1. Capture the current workspace revision or Designer snapshot revision.
2. Build the smallest ordered set of create/update/delete operations.
3. Validate against the current schema and workspace.
4. Review the preview diff, warnings, and high-risk flags.
5. Apply the exact preview only after human confirmation.

Designer API installations expose a similar flow through snapshot, validate, and apply endpoints. Do not call remote mutation endpoints unless the user has placed that environment in scope and authorized the change.

## Design review

Before implementation, confirm:

- ownership App, Model, and layer
- stable naming prefix
- dependencies and target layers
- enum/table definitions before their consumers
- form/menu/action targets
- security artifacts for each table operation, Function, and report
- migration impact and rollback path
