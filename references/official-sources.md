# Official sources and freshness

Use this reference only when exact current behavior is not visible in the running Web Designer or when the installed framework version differs from the baseline below.

## Source priority

1. Controls, validation, diagnostics, and effective objects visible in the target Web Designer.
2. The running instance's displayed version and behavior.
3. Official documentation matching that installed version.
4. Official framework source and tests for that version when documentation is insufficient.

Do not silently combine contracts from different versions. If they disagree, follow the running instance and explain the mismatch.

## Canonical repositories

- Documentation: https://github.com/emu479p01/emu-framework-docs
- Framework source: https://github.com/emu479p01/emu-framework
- Developer overview: https://github.com/emu479p01/emu-framework-docs/blob/main/developer/development-guide.md
- Metadata: https://github.com/emu479p01/emu-framework-docs/blob/main/developer/metadata.md
- Apps, Models, Layers: https://github.com/emu479p01/emu-framework-docs/blob/main/developer/app-model-layer.md
- Extensions: https://github.com/emu479p01/emu-framework-docs/blob/main/developer/extensions.md
- Functions/actions: https://github.com/emu479p01/emu-framework-docs/blob/main/developer/functions.md
- Security: https://github.com/emu479p01/emu-framework-docs/blob/main/developer/security.md
- Views and Charts: https://github.com/emu479p01/emu-framework-docs/blob/main/developer/views-and-charts.md
- Power BI View API: https://github.com/emu479p01/emu-framework-docs/blob/main/admin/power-bi-view-api.md
- Testing: https://github.com/emu479p01/emu-framework-docs/blob/main/developer/testing.md
- AI REST proposal API: https://github.com/emu479p01/emu-framework-docs/blob/main/developer/ai-rest-api.md
- Artifact API, kinds, and nested structures: https://github.com/emu479p01/emu-framework-docs/blob/main/developer/artifact-api.md, `artifact-types.md`, and `artifact-components.md` in the same folder
- Release notes 1.0.0 to 1.4.0 and developer guides (localization and Data Entity extensions, model deployment and ISV licenses, record lifecycle and images): `release-notes/` and `docs/` in https://github.com/emu479p01/emu-framework

The documentation repository can lag the framework source. When a docs page names an older version (for example 0.5.0.0) or disagrees with the running instance, follow the instance and the matching source.

## Bundled baseline

These references target EmuFramework **v1.4.0**, inspected against the v1.4.0 framework source (`@emu/core` and `@emu/server`), the framework release notes 1.0.0 to 1.4.0, the framework `docs/` developer guides, and the official documentation repository on 2026-10-08. The running instance's reported version, capabilities, and schemas remain authoritative.

Versioning: releases use `Major.Minor.Patch`. An FU (Framework Update) is a breaking change (`Major`) or important backward-compatible functionality (`Minor`); a PU (Proactive Update) is a bug, hotfix, or security fix (`Patch`). Legacy four-component versions (`0.1.4.0`, `0.5.0.0`) are immutable history and are not extended; `0.5.0.0` is the last release before `1.0.0`.

Before relying on version-sensitive details such as Web Designer controls, metadata fields, supported artifact kinds, service limits, or security behavior, verify them against the running instance (`/api/designer/capabilities` or `/api/v1/ai/capabilities` report `version`). If the instance is older or newer than 1.4.0, follow the instance, say which version you used, and do not apply 1.4.0 features such as `translation`, `dataEntity`, `dataEntityExtension`, `imageInput`, or audit aliases to an older runtime.

## Version changes since the 0.1.4.0 baseline

- 0.1.5.0 to 0.1.6.1: Tablix report layouts, Menu Extension `parentId` and overrides, paginated report fixes.
- 0.5.0.0: scoped AI REST endpoints (`inspect`, `validate`, `propose`) and the Proposal Inbox replace the removed MCP package; Docker-only production deployment (the user CLI, Windows launchers, and host scripts are gone); Form Extension `lineOverrides`; paginated Designer artifact listing for 5,000+ artifacts.
- 1.0.x: first three-component release; unauthorized form and line actions are omitted from `/api/metadata`; AI Proposal Inbox filtering and removal; guarded container updates and restores.
- 1.1.0: BCP-47 label translations, record attachments, Data Entities with XLSX/CSV-ZIP exchange, opt-in archiving, report paper sizes, units, borders, images, and layout version 2.
- 1.2.0: App `defaultLocale`, Translation and Data Entity editors, `dataEntityExtension`, framework UI in English and Thai, image attachment preview.
- 1.3.0: selected-model deployment packages (schema version 2), Apps & Models administration, optional offline ISV licensing with read-only expiry, app-local Recent navigation.
- 1.4.0: draft-based record creation, virtual audit aliases, UTC datetimes, Function `imageInput`, `EMU_APP_TITLE`. Breaking: lifecycle hooks and data event handlers must be synchronous; system field names are reserved case-insensitively.

Field `multiline` and `encrypted` and Model `license` are present in the v1.4.0 schema; encrypted fields, SMTP secrets, and drafts share one installation key. Confirm each in the live schema.

## Important security changes

- v0.1.1.0: `FW_FrameworkUser` no longer grants all-App Designer scope. Use an ordinary account with App-scoped `FW_AppAccess.canCustomize`; add `canOpen` and object Privileges separately only for authorized runtime verification.
- v0.5.0.0: AI tokens are hashed, scoped, expiring, and revocable. There is no AI apply endpoint and no AI business-record endpoint.
- v1.3.0: expired ISV licenses make the owning App and dependents read-only. License material is administrator-owned.
