# Build EmuFramework Apps in Web Designer

An Agent Skill for people who already have [EmuFramework](https://github.com/emu479p01/emu-framework) installed and running, and want AI to build or customize an App through Web Designer.

This Skill is not an EmuFramework installer and is not primarily a source-code or CLI workflow. It guides an AI agent to connect to your running EmuFramework endpoint, use your authenticated browser session, make metadata changes in Web Designer, review the generated change set, and verify the result in the generated App.

## What it can do

- Create an App and Model through Web Designer.
- Create tables, enums, fields, references, indexes, forms, menus, reports, and security objects.
- Extend an existing App with removable higher-layer Extensions.
- Add Scripts, Functions/actions, hooks, validation, HTTP, or email behavior supported by Web Designer.
- Review warnings and high-risk changes before applying them.
- Test generated menus, lists, forms, actions, permissions, and failure paths.

## Prerequisites

- A running EmuFramework instance.
- Its base URL, such as a local, development, staging, or production endpoint.
- A user account with Web Designer permission for the target App.
- An AI client with access to an interactive browser session.

Sign in yourself when the browser asks. Never paste passwords, API tokens, cookies, setup codes, or integration keys into the conversation.

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

Give the AI your EmuFramework endpoint and the outcome you want:

```text
Use $build-emu-framework-apps with http://localhost:3399 to create an
inventory App in Web Designer with products, stock transfers, and
separate clerk and manager permissions.
```

```text
Use $build-emu-framework-apps to customize our running sales App through
Web Designer. Add a removable CUS field and show it on the order form.
```

If you do not provide an endpoint and no relevant browser tab is open, the Skill asks for the EmuFramework base URL and whether the environment is development, staging, or production before it starts.

## Workflow

1. Connect to the running EmuFramework instance.
2. Let you sign in if needed.
3. Inspect the existing App, Model, layer, dependencies, and objects.
4. Plan the smallest complete metadata change.
5. Build it through Web Designer.
6. Review the generated change-set preview and warnings.
7. Apply changes with environment-appropriate approval.
8. Open the generated App and verify the result.

For a named development or staging instance, an in-scope additive change can proceed after a clean preview. Production, unknown environments, destructive/high-risk previews, and scope expansions require explicit approval before apply.

## Token-efficient design

The core workflow stays in `SKILL.md`. Detailed guidance is loaded only when needed:

- `references/web-designer-workflow.md` — browser and Web Designer procedure
- `references/metadata-design.md` — Apps, Models, layers, naming, and Extensions
- `references/business-logic.md` — Scripts, Functions, hooks, and transactions
- `references/security-and-verification.md` — authorization and testing
- `references/official-sources.md` — version baseline and current sources

## Documentation baseline

The bundled guidance was derived from the official [EmuFramework documentation](https://github.com/emu479p01/emu-framework-docs) for version `0.1.0.2 (Beta)`. The behavior and controls visible in the running instance take priority over bundled examples.
