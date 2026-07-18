---
name: build-emu-framework-apps
description: Develop, extend, debug, review, and test metadata-driven applications built with EmuFramework. Use when Codex works in an EmuFramework repository or must create or modify Apps, Models, tables, enums, forms, menus, reports, security metadata, Extensions, hooks, Scripts, Functions/actions, Web Designer artifacts, CLI or MCP change sets, or application tests. Also use to diagnose metadata validation, layers, dependencies, permissions, transactions, and generated UI behavior. Do not use for unrelated framework-core maintenance.
---

# Build EmuFramework Apps

Build applications against the workspace's installed EmuFramework version. Treat local schemas and diagnostics as authoritative; use bundled references for workflow and design rules.

## Orient before editing

1. Read repository instructions and inspect `git status`.
2. Find the workspace root, `package.json`, `pnpm-workspace.yaml`, `apps/`, and the target app's `app.json`.
3. Determine the installed framework version from the root and relevant package manifests.
4. Prefer the framework's compact inspection surfaces over reading many files:
   - Run `pnpm emu --help` before assuming command syntax.
   - If supported, run `pnpm emu inspect --json` for the workspace summary.
   - If Emu MCP is connected, use `inspect_workspace`, then `inspect_app` only for the target app.
5. Read only target artifacts and direct dependencies. Expand inspection when diagnostics or cross-references require it.

Never read secrets, production business data, or database files merely to understand metadata.

## Establish the contract

Clarify or infer the smallest complete requirement set:

- app boundary and dependencies
- Model and ownership layer
- stable identifier prefix and user-facing labels
- records, relationships, lifecycle rules, and user operations
- forms, menus, reports, and action placement
- user personas and allowed table operations, Functions, reports, and app access
- acceptance cases, including denied and failure paths

State material assumptions. Avoid inventing fields, permissions, or destructive migrations from vague requirements.

## Load the relevant guidance

- Read [references/project-workflow.md](references/project-workflow.md) for a new app, repository discovery, authoring channel, CLI/MCP use, or build order.
- Read [references/metadata-design.md](references/metadata-design.md) for Apps, Models, layers, naming, metadata, change sets, or Extensions.
- Read [references/business-logic.md](references/business-logic.md) for hooks, events, Scripts, Functions, actions, transactions, HTTP, or email.
- Read [references/security-and-verification.md](references/security-and-verification.md) for permissions, testing, release review, schema risk, or deployment readiness.
- Read [references/official-sources.md](references/official-sources.md) when the local version differs from the documented baseline, the local schema is unavailable, or a current framework fact must be verified.

Do not load every reference by default.

## Design in dependency order

Create an artifact graph before implementation:

```text
App -> Models/layers -> Enums -> Tables/fields/indexes
    -> Forms/reports -> Menus -> Privileges -> Duties -> Roles -> App access
Tables -> Hooks/Scripts/Functions -> Privileges
```

Choose the smallest behavior mechanism:

- Use a field default or hook for record defaults and validation.
- Use a data event for lifecycle reactions.
- Use a Function for one explicit named operation.
- Use a Script for a small set of related registrations.
- Use reviewed TypeScript for complex reusable or native integration logic.

Prefer an Extension for additive, independently removable customization. Replace a lower-layer base artifact only when the higher layer intentionally owns the complete definition.

## Use the live schema

Never guess metadata properties from memory or bundled examples.

1. Prefer `emu://schema/metadata` and `emu://schema/change-set` when Emu MCP resources are available.
2. Otherwise use `pnpm emu schema` when listed by local CLI help.
3. Otherwise inspect the installed schema/types and nearby valid artifacts from the same framework version.
4. Use official current documentation only as a fallback; reconcile it with the installed version.

Generate or edit the minimum set of artifacts. Preserve existing formatting and naming conventions. Do not declare framework audit fields (`id`, `createdAt`, `createdBy`, `modifiedAt`, `modifiedBy`) as application fields.

## Validate before applying

For a multi-artifact change, prefer one atomic MetadataChangeSet.

1. Inspect the current workspace revision.
2. Validate and preview without mutation, using `validate_change_set` or the local CLI equivalent.
3. Resolve every schema, reference, dependency, layer, naming, and permission diagnostic.
4. Review the diff and all high-risk flags.
5. Apply only after the user has authorized mutation and the preview matches the intended scope.

Do not use an apply command as a substitute for file editing when the user asked only for a plan, review, or diagnosis.

## Preserve runtime invariants

- Route data access through the authenticated framework `DataContext`.
- Treat client visibility as usability, never authorization.
- Keep transactional Functions synchronous and atomic.
- Use async Functions for awaited HTTP or email work; place database changes in short synchronous `ctx.tts()` blocks.
- Never await network I/O inside a database transaction.
- Keep credentials out of metadata, source, logs, and responses.
- Treat Scripts and Functions as trusted administrative code requiring review.
- Plan explicit migrations and verified backups for removals or structural changes; schema synchronization is additive.

## Verify proportionally

Run focused checks first, then the repository's complete supported gate. A current framework checkout commonly exposes:

```sh
pnpm check:versions
pnpm typecheck
pnpm test
pnpm build
```

Use only scripts present in the local repository. Test success, validation failure, rollback, authorized and unauthorized users, update/delete/reference behavior, and Extension enabled/disabled behavior. For async Functions, also test service failures, timeouts, non-success responses, limits, and database state around explicit transactions.

## Report the result

Lead with the implemented user outcome. Summarize artifacts added or changed, ownership layer and Extension decisions, security coverage, validation commands and results, and any migration, backup, or beta risk. Mention assumptions or skipped checks explicitly.
