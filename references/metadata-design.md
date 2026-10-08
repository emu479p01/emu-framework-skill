# Metadata design

Use this reference for Apps, Models, layers, metadata identity, field rules, Extensions, and change sets. Rules are verified against EmuFramework v1.4.0.

## Core model

```text
App
└── Model
    └── Layer
        └── Metadata artifacts
```

- App: top-level runtime access/navigation boundary plus identity, dependencies, display information, optional `defaultLocale`, and Models.
- Model: coherent development-metadata group within an App; it is not a security boundary. A Model may carry `license: { vendor }` (see below).
- Layer: ownership and precedence when multiple sources contribute to the same logical artifact.
- `name`: stable global identifier; do not treat it as display text.
- `label`: user-facing text. Localized variants live in Translations ([localization-and-data-entities.md](localization-and-data-entities.md)).

Declare cross-app dependencies explicitly. App dependency order controls load order and whether cross-app Extensions and cross-app Translations are allowed.

Every App created in current versions starts with `models: []`, including names such as `erp`, `erp.credit`, and `web`. The user must add a Model and choose its Layer before creating any business artifact. Put the exact `app`, `model`, and matching `layer` on every supported artifact. Never upsert the App manifest or a Model definition; those belong to the user.

### Licensed Models

A Model with `license: { vendor }` is a licensed ISV Model (only the `ISV` Layer may carry it). Its requirement cannot be removed or changed through ordinary customization, and deleting the Model is rejected. Treat the property as read-only: never add, edit, or copy it, and never touch license files, trust keys, or seller keys. When the license is missing, invalid, not yet valid, or expired, the owning App and every dependent App go read-only: reads and exports continue, business writes, Functions, imports, and attachment changes fail. A failed verification step can therefore be a license problem, not a defect in your metadata. Report it and let the administrator renew.

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

A higher-layer base artifact replaces a lower-layer base artifact with the same logical identity. An Extension accumulates supported additions into its target. Framework metadata (`system` App, names starting `FW_`) is read-only and never a valid target.

## Extension rules

Use an Extension when a feature adds fields, indexes, enum values, form behavior, menu items, View columns, Chart measures, permissions, Data Entity fields, or Script/Function behavior and must remain independently removable.

Supported Extension kinds in v1.4.0:

```text
tableExtension, enumExtension, formExtension, menuExtension,
privilegeExtension, dutyExtension, roleExtension, scriptExtension,
viewExtension, chartExtension, functionExtension, dataEntityExtension
```

There is no `reportExtension` or `translationExtension`. Translations are additive base artifacts; see the localization reference.

Enforce all of these constraints:

- Give the Extension a unique, traceable stable name that ends `_Extension`.
- Set its layer strictly higher than the target's layer.
- Declare a direct or transitive dependency when targeting another App.
- Allow only one Extension of a given kind for the same `(app, model, kind, target)`. Check the workspace first and update the existing one.
- Add only the required contributions; do not copy the base definition.
- Treat inherited layers as read-only and save only the current Layer delta.
- Target Form groups/actions/Charts/lines and menu items by stable ID when overriding labels, visibility, order, or icons. Every `targetId` must exist in the effective inherited artifact.
- Test with the Extension enabled and disabled.
- Ensure removal leaves no orphaned references.

Delta properties:

| Extension | Appends | Overrides |
| --- | --- | --- |
| `tableExtension` | `fields`, `indexes` | `fieldOverrides`: `label`, `readOnly`, `allowEdit`, `allowEditOnCreate`, `multiline`, `encrypted` |
| `formExtension` | `listFields`, `filterFields`, `groups`, `charts`, `actions`, `lines` | `elementOverrides` (`label`, `hidden`, `order`); `lineOverrides` (`targetId` plus `label`, `hidden`, `order`, `fields`, `aggregates`, `actions`) |
| `menuExtension` | `items` (with `parentId` to attach under an inherited group), `insertions` | `itemOverrides` |
| `enumExtension` | `values` | `valueOverrides` |
| `viewExtension` / `chartExtension` | joins, columns, filters, ordering / measures | `columnOverrides` / `measureOverrides` |
| `dataEntityExtension` | `fields`, `lines`, `lineExtensions` | none (append-only) |

`lineOverrides` was added in 0.5.0.0. It replaces the inherited line's presentation, `fields`, `aggregates`, or `actions` wholesale for the listed keys, so send the full desired list, and it can never change the inherited `table` or `refField`. Add a brand-new Line grid with `lines` instead. Read the effective line through `GET /api/designer/customization/form/<name>` first.

Canonical names use `<AppPrefix>_<ModelName>_<BaseName>_Extension`. A name that ends `_Extension` but differs from the canonical form loads with a warning; preserve any warned legacy name until a human reviews the rename and references.

## Field rules

Apply these when defining or extending table fields:

- Field names are unique per table and cannot be system fields. System field names are reserved case-insensitively: `id`, `createdAt`, `createdBy`, `modifiedAt`, `modifiedBy`, and the audit aliases `sys_createdBy`, `sys_createdAt`, `sys_modifiedBy`, `sys_modifiedAt`. `Status` is fine; `CreatedBy`, `SYS_CREATEDAT`, and `Id` are rejected. Schema sync stops with no changes if an existing stored column collides with an alias.
- The audit aliases are virtual, read-only fields mapped onto the existing audit columns. Never define them as table fields. They are available automatically in list filters, sorting, `Query`, Record reads, and View field references (for example `r.sys_createdBy`), and `/api/metadata` table metadata now includes the system fields.
- Datetimes are stored and returned in UTC; see [business-logic.md](business-logic.md) for write handling.
- `multiline: true` renders a string field as a multi-line editor and preserves newlines. It is valid only on `string` fields.
- `encrypted: true` stores a `string` field encrypted at rest. Generic APIs return only a configured-value mask (`••••••••`); sending the mask back on update leaves the value unchanged, and it is rejected as a new secret on create. An encrypted field cannot define a `default`, be a `titleField`, be indexed, be a lookup display field, lookup filter field, or `copyFields` source, appear in Form `filterFields`, be filtered/searched/sorted, appear in a View, be rendered in or used as a parameter of a Report, or take part in Data Entity exchange. Toggling `encrypted` on an existing field migrates stored values in place. Treat that as high-risk, require explicit user approval, and remind the user that encrypted values depend on `.emu-secret.key`.
- `enum` fields must be optional. A `readOnly` field cannot be `mandatory`: the 1.4.0 release notes say a read-only field may be mandatory when `initValue` supplies it, but the 1.4.0 schema validator still returns `readonly_optional` for `readOnly` plus `mandatory` and registry load drops `mandatory` from read-only fields. Do not work around a `readonly_optional` diagnostic. Use `readOnly` with an `initValue` hook alone, enforce the value in `validateWrite`, and tell the user.
- Trusted Functions and Scripts may populate read-only fields; REST writes and generated Forms cannot edit them.

## Schema and storage safety

Supported base concepts: Apps, enums, tables, Forms, menus, Privileges, Duties, Roles, Scripts, Functions, Reports, Views, Charts, Translations, and Data Entities. Use only object kinds and properties exposed by the live schema.

Schema synchronization is additive for new tables, fields, and indexes. Removing or changing existing structures requires an explicit migration and verified backup. Never edit generated SQLite structure manually. Artifact names are global identities; avoid reusing a name across Apps.

## Atomic change-set flow

Use one ChangeSet for dependent tables, Views, Charts, Forms, menus, Translations, and security artifacts inside the user-provided App/Model contract:

1. Inspect the current App, Model, layer, and objects before editing.
2. Build the smallest ordered set of operations.
3. Validate and read the preview diff, warnings, and high-risk flags.
4. Submit for human apply or Proposal Inbox approval with authority appropriate to the environment and risk.

## Design review

Before implementation, confirm:

- ownership App, Model, and layer
- stable naming prefix
- dependencies and target layers
- enum/table definitions before their consumers
- form/menu/action targets and stable IDs
- View sources, joins, parameters, grouping, Chart output fields, and Form parameter bindings
- Translation keys for every new user-visible label when the App has more than one locale
- security artifacts for each table operation, Function, Report, and View
- migration impact and rollback path
