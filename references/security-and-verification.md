# Security and verification

Use this reference for authorization design, testing, beta risks, and release readiness.

## Authorization model

```text
User -> Role -> Duty -> Privilege -> table/form/function/report permissions
User -> App access (open/customize)
```

A Role may also reference privileges directly. App access is separate from roles and privileges.

- Grant table read/create/update/delete operations explicitly.
- Grant form access separately from table operations.
- Add every named Function and report to an appropriate privilege.
- Assign privileges through duties/roles to real user personas.
- Test direct API denial; filtered menus and disabled buttons are not security boundaries.

`FW_SystemAdminRole` is the superuser role in v0.1.0.2. The username `admin` has no inherent privilege. Do not use an unrestricted context to bypass authorization.

## Dedicated AI-assistance user

Require an ordinary dedicated user with `FW_AppAccess.canOpen=true` and `canCustomize=true` only for the target App. This account scopes Designer API reads and writes to that App.

Do not use `FW_SystemAdminRole` for routine AI work. Do not use `FW_FrameworkUser` for routine AI work in v0.1.0.2: source grants it all-App Designer scope even though documentation describes its data policy as self-service.

App customization permission does not grant business-table CRUD. Assign separate least-privilege roles only for explicitly authorized generated-App verification.

## Minimum test matrix

Test with both an administrator and realistic non-administrator accounts.

| Area | Required cases |
| --- | --- |
| Metadata | shape, names, App/Model scope, dependencies, layers, cross-references |
| Tables | allowed and denied CRUD, validation, defaults, update/delete/reference behavior |
| UI | generated lists/forms, lookups, menu filtering, disabled operations, direct navigation denial |
| Functions/reports | privilege allowed and denied, direct API denial, invalid inputs, rollback |
| Extensions | enabled/disabled, interaction with other Extensions, clean removal |
| Async integration | success, rejection, timeout, limits, explicit transaction boundaries |

Run focused checks through the generated App while iterating. Reopen affected objects in Web Designer after save/apply and verify the effective result rather than trusting the preview alone.

## Release checklist

- Confirm naming prefix, App, Model, and layer.
- Confirm dependencies for every cross-App reference or Extension.
- Preview the change set and review every high-risk diff.
- Test permissions with real user roles, not only a system administrator.
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
