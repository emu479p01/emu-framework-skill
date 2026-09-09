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

## Bundled baseline

These references target official EmuFramework documentation and framework source version `0.1.4.0 (Beta)`, inspected together on 2026-08-14. The running instance's capabilities and schemas remain authoritative.

Before relying on version-sensitive details such as Web Designer controls, metadata fields, supported artifact kinds, service limits, or security behavior, verify them against the running instance. If the UI does not reveal the answer, consult current official documentation and state the version used.

## Important v0.1.1.0 security change

`FW_FrameworkUser` no longer grants all-App Designer scope. Use an ordinary account with App-scoped `FW_AppAccess.canCustomize`; add `canOpen` and object Privileges separately only for authorized runtime verification.
