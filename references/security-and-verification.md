# Security and verification

Use this reference for authorization design, secrets and licenses, testing, risk, and release readiness. Rules are verified against EmuFramework v1.4.0.

## Authorization model

```text
User -> App Access canOpen --------------------------+
User -> Role -> Duty -> Privilege -> object access -+-> runtime request
User -> App Access canCustomize ------------------------> Designer only
AI REST token (apps + scopes + expiry) -----------------> inspect / validate / propose only
```

A Role may also reference Privileges directly. Normal runtime authorization is deny-by-default and requires both `canOpen` for the owning App and the matching object permission. App Access is separate from Roles and Privileges.

- Grant table read/create/update/delete operations explicitly.
- Grant form access separately from table operations.
- Add every named Function and Report to an appropriate Privilege.
- Add every View to an appropriate Privilege and grant read permission to all source tables. Chart access is inherited from its View.
- A Data Entity has no Privilege list; its export needs `read` and its import needs `create` and `update` on the root and line tables.
- Assign privileges through duties/roles to real user personas.
- Test direct API denial. Since 1.0.2, form and line actions the caller may not run are omitted from `/api/metadata`, but a direct call still returns `403`. Omission is a convenience, not a boundary.
- Because unauthorized objects are omitted from `/api/metadata`, a missing menu item, action, or Function for a test account usually means a missing Privilege, not a missing artifact. Confirm with the Designer snapshot before concluding the artifact was not applied.

`FW_SystemAdminRole` is the only global bypass. The username `admin` has no inherent privilege. `FW_FrameworkUser` is a legacy marker, not a Designer bypass (since v0.1.1.0). Do not use an unrestricted context to bypass authorization.

## Dedicated AI-assistance credentials

Require one of the following, scoped to the target App only:

- An ordinary dedicated user with `FW_AppAccess.canCustomize=true`. This scopes Designer API reads and validation to that App without runtime data access.
- An AI REST token created by a System Administrator for the existing target App with the minimum scopes (`inspect`, plus `validate` and `propose` as needed) and a short expiry. The token cannot apply changes or read business records. Its App scope does not enforce Model or Layer, so the skill's fixed scope contract is the only Model/Layer guard.

Do not use `FW_SystemAdminRole` or `FW_FrameworkUser` for routine AI work. App customization permission does not grant App entry or business-table CRUD. For explicitly authorized generated-App verification, assign `canOpen=true` and separate least-privilege Roles only for the objects being tested.

## Secrets, license keys, and sensitive material

Never request, read, generate, print, store, or place in metadata or code any of: passwords, session cookies, AI tokens, View service tokens, setup codes, `.emu-secret.key` contents, SMTP credentials, ISV license files and payloads, license keys, vendor private keys, seller key pairs, or output of the `scripts/isv-license.mjs` seller CLI. Never reproduce a value you happen to see in output; redact it.

- ISV licensing is offline and administrator-owned (Ed25519 license files imported in **Settings, Apps & Models**). The skill must never import, issue, renew, or inspect licenses, register vendor keys, or change a `license: { vendor }` requirement. A seller private key must never reach a customer deployment; if you see one, stop and tell the user.
- A licensed Model whose license is missing, invalid, not yet valid, or expired puts its App and dependent Apps into read-only operation with no grace period. Metadata stays loaded; writes, Functions, imports, and archive processing are guarded server-side. Verification that creates records or runs actions can then fail for license reasons rather than design reasons. Report the license state to the user; do not attempt a workaround such as moving artifacts to another Model or removing the requirement.
- Encrypted fields (`encrypted: true`), SMTP passwords, and drafts share the installation's `.emu-secret.key` (or `EMU_SECRET_KEY_PATH`). It is not part of `.emubackup` exports. Losing it makes those values unrecoverable. Remind the user to keep a separate protected copy; never ask for it.
- Do not embed credentials in metadata, Scripts, Functions, Translations, or report assets.

## Deployment packages are a human workflow

Selected-model deployment packages (schema version 2, vendor-update or UAT-to-Prod modes) and App/Model export and import are administrator workflows in Web Designer. Do not build, export, import, or preview packages unless the user explicitly asks for it. Packages carry complete metadata including Scripts and Functions, and an import preview treats them as Designer-sourced, so it bypasses the AI executable-artifact restriction; never use it to deliver code that a ChangeSet would reject. Destination environments need their own licenses; a package never carries business data, users, installation identity, or licenses. A checksum detects corruption and is not a signature.

## Minimum test matrix

Test with both an administrator and realistic non-administrator accounts.

| Area | Required cases |
| --- | --- |
| Metadata | shape, names, App/Model scope, dependencies, layers, cross-references |
| Tables | allowed and denied CRUD, validation, defaults, update/delete/reference behavior, reserved field names, encrypted field masking |
| Drafts | new draft shows `initValue` values, save, retry, expiry, reopen after metadata change |
| UI | generated lists/forms, lookups, menu filtering, disabled operations, direct navigation denial |
| Functions/reports | privilege allowed and denied, direct API denial, invalid inputs, rollback, image input rejection cases |
| Views/Charts | App + View + source-read gates, row scopes, parameters, audit alias columns, wrong token scope, missing bindings, responsive rendering |
| Localization | labels in each locale, fallback to base language and `defaultLocale`, no stale-key warnings, no `ui.*` keys |
| Data Entities | merged entity after Extension, optional versus mandatory fields, no business-data import run by AI |
| Reports | layout version 2 passes, print preview on the target paper, image sources, mixed-language text |
| Extensions | enabled/disabled, interaction with other Extensions, clean removal |
| Async integration | success, rejection, timeout, limits, explicit transaction boundaries |
| Licensed Models | read-only behavior, reported rather than bypassed |

Run focused checks through the generated App while iterating. Reopen affected objects in Web Designer after apply and verify the effective result rather than trusting the preview alone.

## Release checklist

- Confirm naming prefix, App, Model, and layer.
- Confirm dependencies for every cross-App reference, Extension, and Translation.
- Preview the change set or submit the proposal, and review every high-risk diff.
- Test permissions with real user roles, not only a system administrator.
- Test no Role/no App Access, Role only, App only, Customize only, App plus matching Role, and System Administrator.
- Test empty and changing dynamic lookup sources plus deleted references.
- Handle a create-page Function action when no `recordId` exists.
- Verify async failure and explicit-transaction behavior.
- Confirm no lifecycle handler is asynchronous.
- Tell the human how the customization is backed up and promoted. Do not run packaging yourself.
- Back up `data.db` and `designer.db` together before risky schema or deployment changes. Updates and restores are performed through the Docker updater, not by editing files.
- Preserve `.emu-secret.key` or `EMU_SECRET_KEY_PATH` separately; `.emubackup` excludes it.

## Beta and operational cautions

- Deleting metadata may leave physical business data until deliberately purged.
- Additive schema synchronization does not make destructive changes safe. Toggling `encrypted` rewrites stored values.
- Scripts and Functions are executable administrative code; change-set validation does not run them.
- Large lookup datasets need a search/picker rather than an oversized dropdown.
- Artifact names act as global identities; avoid reuse across Apps.
- Never delete SQLite `-wal` or `-shm` files while the application is running.
- External side effects (email, HTTP, uploaded images) cannot be rolled back with SQLite.

Report any skipped check, untested role, migration prerequisite, license state, or backup requirement at handoff.
