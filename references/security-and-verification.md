# Security and verification

Use this reference for authorization design, testing, beta risks, and release readiness.

## Authorization model

```text
User -> App Access canOpen --------------------------+
User -> Role -> Duty -> Privilege -> object access -+-> runtime request
User -> App Access canCustomize ------------------------> Designer only
```

A Role may also reference Privileges directly. Normal runtime authorization is deny-by-default and requires both `canOpen` for the owning App and the matching object permission. App Access is separate from Roles and Privileges.

- Grant table read/create/update/delete operations explicitly.
- Grant form access separately from table operations.
- Add every named Function and Report to an appropriate Privilege.
- Add every View to an appropriate Privilege and grant read permission to all source tables. Chart access is inherited from its View.
- Assign privileges through duties/roles to real user personas.
- Test direct API denial; filtered menus and disabled buttons are not security boundaries.

`FW_SystemAdminRole` is the only global bypass in v0.1.1.0. The username `admin` has no inherent privilege. `FW_FrameworkUser` is a legacy marker, not a Designer bypass. Do not use an unrestricted context to bypass authorization.

## Dedicated AI-assistance user

Require an ordinary dedicated user with `FW_AppAccess.canCustomize=true` only for the target App. This account scopes Designer API reads and writes to that App without granting runtime data access.

Do not use `FW_SystemAdminRole` or `FW_FrameworkUser` for routine AI work.

App customization permission does not grant App entry or business-table CRUD. For explicitly authorized generated-App verification, assign `canOpen=true` and separate least-privilege Roles only for the objects being tested.

## Minimum test matrix

Test with both an administrator and realistic non-administrator accounts.

| Area | Required cases |
| --- | --- |
| Metadata | shape, names, App/Model scope, dependencies, layers, cross-references |
| Tables | allowed and denied CRUD, validation, defaults, update/delete/reference behavior |
| UI | generated lists/forms, lookups, menu filtering, disabled operations, direct navigation denial |
| Functions/reports | privilege allowed and denied, direct API denial, invalid inputs, rollback |
| Views/Charts | App + View + source-read gates, row scopes, parameters, wrong token scope, missing bindings, responsive rendering |
| Extensions | enabled/disabled, interaction with other Extensions, clean removal |
| Async integration | success, rejection, timeout, limits, explicit transaction boundaries |

Run focused checks through the generated App while iterating. Reopen affected objects in Web Designer after save/apply and verify the effective result rather than trusting the preview alone.

## Release checklist

- Confirm naming prefix, App, Model, and layer.
- Confirm dependencies for every cross-App reference or Extension.
- Preview the change set and review every high-risk diff.
- Test permissions with real user roles, not only a system administrator.
- Test no Role/no App Access, Role only, App only, Customize only, App plus matching Role, and System Administrator.
- Test empty and changing dynamic lookup sources plus deleted references.
- Handle a create-page Function action when no `recordId` exists.
- Verify async failure and explicit-transaction behavior.
- Export the App/Model package when the installed Web Designer supports packaging, or document how the customization is backed up and promoted.
- Back up `data.db` and `designer.db` before risky schema or deployment changes.
- If SMTP is configured, preserve `.emu-secret.key` or `EMU_SECRET_KEY_PATH` separately; `.emubackup` excludes it.

## Beta cautions

- Deleting metadata may leave physical business data until deliberately purged.
- Additive schema synchronization does not make destructive changes safe.
- Scripts and Functions are executable administrative code.
- Large lookup datasets need a search/picker rather than an oversized dropdown.
- Artifact names act as global identities; avoid reuse across Apps.
- Never delete SQLite `-wal` or `-shm` files while the application is running.

Report any skipped check, untested role, migration prerequisite, or backup requirement at handoff.
