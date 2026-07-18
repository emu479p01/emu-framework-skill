# Build EmuFramework Apps through the Designer API

An Agent Skill for people who already have [EmuFramework](https://github.com/emu479p01/emu-framework) installed and running and want AI-assisted development inside an existing App and Model.

The Skill uses the Web Designer REST API to inspect schemas and metadata, prepare atomic change sets, validate them, and verify the result. Web Designer remains the human setup and approval surface.

## Required setup

Complete these steps in Web Designer before asking AI to work.

### 1. Create the target App and Model

Create the App and a dedicated Model, then choose its Layer:

- `SYS` — framework-owned
- `ISV` — reusable product/vendor
- `LOC` — localization or site customization
- `DEV` — development customization
- `CUS` — customer/implementation customization

Record the exact App name, Model name, and Layer. The AI will not create or guess them.

In v0.1.1.0, every new App starts with zero Models. Add the Model explicitly; Apps named `erp`, `erp.credit`, or `web` receive no special default.

### 2. Create a dedicated AI-assistance user

Create an ordinary user specifically for AI-assisted customization. In that user's App Access, grant only the target App and set:

```text
canCustomize = true
```

This is sufficient for Designer inspection and validation. `canCustomize` does not grant App entry or business-data access.

Avoid assigning `FW_SystemAdminRole` or `FW_FrameworkUser` for routine AI work. `FW_SystemAdminRole` is the only global bypass; `FW_FrameworkUser` is a legacy marker in v0.1.1.0.

Only when generated-App verification is genuinely required, add `canOpen=true` plus the minimum Role/Privilege permissions for the Forms, tables, Functions, Reports, or Views being tested.

### 3. Prepare authentication securely

Sign in using a browser session or configure the dedicated username/password in environment variables or a secret store outside the AI conversation. Never paste passwords, session cookies, API tokens, setup codes, or integration keys into a prompt.

## Information to give the AI

Provide this contract with every task:

```text
Endpoint: http://localhost:3399
Environment: development
App: sales
Model: AICustomizations
Layer: CUS
AI user: created with canCustomize=true for sales; no runtime access
Request: Add a delivery note field to the order table and form.
```

Port `3399` is the framework default; use the actual endpoint configured for your installation.

## Install the Skill

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

Start a new Codex task after installation.

## AI workflow

1. Authenticate without exposing credentials.
2. Read `/api/designer/capabilities`.
3. Read `/api/designer/snapshot?app=<App>`.
4. Verify the App, Model, Layer, and account scope.
5. Build a version-1 change set with `source: "ai"`.
6. Validate it through `/api/designer/change-sets/validate`.
7. Present the preview ID, expiry, diff, warnings, schema effects, and risks.
8. Let the human submit apply confirmation with the same dedicated username when the runtime disallows AI apply, through Web Designer if supported or a human-controlled API request.
9. Refresh the snapshot and visually verify the generated App.

## API support

EmuFramework v0.1.1.0 supports:

- `POST /api/designer/artifacts` — create one metadata artifact; success `201`, duplicate `409`
- `PUT /api/designer/artifacts/:kind/:name` — idempotent upsert
- `GET /api/designer/capabilities` — version, revision, schemas, and AI capability flags
- `GET /api/designer/snapshot` — current metadata revision and artifacts
- `POST /api/designer/change-sets/validate` — atomic validation and preview
- `POST /api/designer/change-sets/apply` — human-confirmed apply when policy permits
- `POST /api/data/:table` — create a business Record when table permissions permit
- `GET /api/views/:name/schema` — inspect a permitted View contract
- `GET /api/views/:name/data` — execute a permitted declarative View with typed parameters and paging
- `GET /api/views/:name/export?format=csv` — export a permitted View under the configured row cap

The Skill prefers the change-set workflow because several direct artifact calls can leave a partially completed design if a later call fails.

View and Chart are supported metadata artifact kinds in v0.1.1.0. Interactive View verification requires all three runtime gates: `canOpen`, a View Privilege, and read permission for every source table. Chart access is inherited from its View. Power BI service tokens remain human-administered and outside the Skill's secret handling.

In v0.1.1.0, the capabilities response declares AI inspection and validation support while AI apply, business-data access, and executable Scripts are disabled. The Skill obeys those runtime flags rather than bypassing them. The human owns apply confirmation.

## Documentation baseline

The bundled guidance is based on official [EmuFramework documentation](https://github.com/emu479p01/emu-framework-docs) and framework source version `0.1.1.0 (Beta)`. The running instance's capabilities and schemas are authoritative.
