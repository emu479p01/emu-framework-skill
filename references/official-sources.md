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
- Testing: https://github.com/emu479p01/emu-framework-docs/blob/main/developer/testing.md

## Bundled baseline

These references were derived from official EmuFramework documentation version `0.1.0.2 (Beta)`, documentation commit `4d08d944049e8ea77c7bdf163a47efdb50654867`, and framework source commit `8bf9a238aa45541b35803b81f2c6603d122bbbc1`, both inspected on 2026-07-18.

Before relying on version-sensitive details such as Web Designer controls, metadata fields, supported artifact kinds, service limits, or security behavior, verify them against the running instance. If the UI does not reveal the answer, consult current official documentation and state the version used.

## Known v0.1.0.2 documentation mismatch

The security documentation describes `FW_FrameworkUser` as self-service, but server source grants `FW_FrameworkUser` the same all-App Designer scope check used for `FW_SystemAdminRole`. For AI-assisted work, avoid both roles and use an ordinary account with App-scoped `FW_AppAccess.canCustomize` until the framework resolves or documents this behavior differently.
