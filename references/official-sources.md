# Official sources and freshness

Use this reference only when exact current behavior is not available from the target workspace or when the installed framework version differs from the baseline below.

## Source priority

1. Target repository instructions, installed version, CLI help, live schemas, diagnostics, and existing valid artifacts.
2. Connected Emu MCP schemas and workspace resources from the same checkout.
3. Official documentation matching the installed version.
4. Official framework source and tests matching the installed version.

Do not silently combine contracts from different versions. If they disagree, follow the target workspace and explain the mismatch.

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

Before relying on version-sensitive details such as CLI syntax, schema fields, supported artifact kinds, service limits, or security behavior, verify them against the target checkout. When no checkout is present, consult the current official repository and state the version used.
