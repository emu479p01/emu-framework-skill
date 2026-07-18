# Project workflow

Use this reference for repository discovery, a new application, authoring-channel selection, and implementation order.

## Inspect efficiently

Start at the repository root and inspect, in order:

1. repository instructions and working-tree state
2. root `package.json` and `pnpm-workspace.yaml`
3. `pnpm emu --help`
4. `pnpm emu inspect --json`, when supported
5. target `apps/<app>/app.json`
6. only the target metadata directories and directly referenced artifacts

Recent EmuFramework workspaces store application manifests at `apps/<app>/app.json`, metadata JSON below `apps/<app>/metadata/`, and reviewed TypeScript behavior below the app package's `src/`. Confirm paths locally because versions can differ.

The recent CLI exposes AI-friendly `inspect`, `schema`, `validate`, and `apply` operations. Use only commands shown by local help. Prefer `inspect --json` to broad file reads and `schema` to guessing properties.

## Prefer connected Emu MCP when available

Use these read-only surfaces to reduce context and avoid accidental mutation:

- resources: `emu://schema/metadata`, `emu://schema/change-set`, `emu://workspace/apps`
- templated app resource: `emu://workspace/app/{name}`
- tools: `inspect_workspace`, `inspect_app`, `validate_change_set`, `explain_diagnostics`

The framework MCP intentionally does not apply changes, delete artifacts, execute Scripts, or query business data. Keep mutation in reviewed workspace edits or an explicitly confirmed CLI/API apply flow.

## Choose an authoring channel

| Need | Default channel |
| --- | --- |
| Reviewed, repeatable application source | Files, CLI, or workspace change set |
| Customer-owned runtime customization | Web Designer or Metadata API change set |
| One simple scaffold | Interactive CLI |
| Several interdependent artifacts | Atomic change set |
| Complex reusable/native behavior | Reviewed TypeScript |

Do not mix ownership casually. Keep the base solution source-controlled and use an appropriate higher layer for customer-specific customization.

## Build in reference order

1. Create the App manifest and declare `dependsOn`.
2. Define Models and layer ownership.
3. Create enums.
4. Create tables, fields, references, and indexes.
5. Create forms and reports.
6. Create menus.
7. Create privileges, duties, and roles.
8. Configure user role assignments and app access.
9. Add hooks, Scripts, Functions, and actions.
10. Validate cross-references, preview changes, and exercise generated UI/API behavior.
11. Export a package or commit source-controlled metadata.

If a file-based app is added while the server is running, reload metadata from Designer or restart the service.

## Development baseline

The v0.1.0.2 documentation specifies Node.js 24.18.0 and pnpm 11.12.0. Always prefer the target repository's declared `packageManager`, toolchain, and engine constraints. A typical local loop uses `pnpm install`, `pnpm dev`, focused tests, then the full verification gate.
