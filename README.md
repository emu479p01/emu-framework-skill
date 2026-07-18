# Build EmuFramework Apps

An Agent Skill for building, extending, reviewing, and testing secure metadata-driven applications with [EmuFramework](https://github.com/emu479p01/emu-framework).

The Skill is designed for AI coding agents that need to work efficiently with EmuFramework Apps, Models, layers, metadata, Extensions, Functions, Scripts, security, CLI/MCP tooling, and Web Designer or Metadata API customizations.

## Why use this Skill?

- Uses the target workspace's live schema and diagnostics instead of guessing metadata properties.
- Builds artifacts in dependency order.
- Selects the smallest appropriate business-logic mechanism.
- Preserves App, Model, layer, dependency, and Extension boundaries.
- Treats client visibility as usability rather than authorization.
- Validates and previews multi-artifact change sets before applying them.
- Loads detailed references only when relevant to reduce token usage.

## Install

Clone the repository into your Codex skills directory using the Skill name as the folder name.

macOS or Linux:

```sh
git clone https://github.com/emu479p01/emu-framework-skill.git \
  "${CODEX_HOME:-$HOME/.codex}/skills/build-emu-framework-apps"
```

Windows PowerShell:

```powershell
git clone https://github.com/emu479p01/emu-framework-skill.git `
  "$env:USERPROFILE\.codex\skills\build-emu-framework-apps"
```

Start a new Codex task after installation so the Skill can be discovered.

## Use

Invoke it explicitly with `$build-emu-framework-apps`:

```text
Use $build-emu-framework-apps to build a source-controlled inventory App
with transfer actions and clerk/manager permissions.
```

```text
Use $build-emu-framework-apps to add a removable CUS Extension to the
existing sales App and verify its security and migration risks.
```

The Skill may also be selected implicitly when a task clearly involves EmuFramework application development.

## Runtime endpoint behavior

The Skill does not ask for an endpoint for local repository work, planning, or review.

When customization requires a running Web Designer or Metadata API environment, the Skill first tries to discover the target from the provided context. If it cannot, it asks for:

- the App base URL
- whether the environment is development, staging, or production
- the target App, Model, and layer
- an existing supported authentication/session method
- whether the requested scope is inspection, preview, or apply

Do not paste passwords, API tokens, cookies, setup codes, or integration keys into the conversation. Use an existing authenticated browser/session, connected tool, or secrets configured outside AI context.

Production remains read-only until the user explicitly approves the exact target and validated preview.

## How it stays token-efficient

`SKILL.md` contains the core workflow and routes the agent to focused references only when needed:

- `references/project-workflow.md` — repository, CLI/MCP, authoring channels, and runtime endpoints
- `references/metadata-design.md` — Apps, Models, layers, schemas, and Extensions
- `references/business-logic.md` — hooks, events, Scripts, Functions, and transactions
- `references/security-and-verification.md` — authorization, tests, and release safety
- `references/official-sources.md` — source priority, version baseline, and freshness

## Documentation baseline

The bundled guidance was derived from the official [EmuFramework documentation](https://github.com/emu479p01/emu-framework-docs) for version `0.1.0.2 (Beta)`. The installed workspace version, live schemas, CLI help, and diagnostics always take priority over bundled examples.
