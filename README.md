# Build EmuFramework Apps through the Designer and AI REST APIs

An Agent Skill for people who already have [EmuFramework](https://github.com/emu479p01/emu-framework) installed and running and want AI-assisted development inside an existing App and Model.

The Skill targets EmuFramework **v1.4.0**. It uses either the Designer REST API (a customization session) or the scoped AI REST proposal API (a Bearer token) to inspect schemas and metadata, prepare atomic change sets, validate them, and hand them to a human. Neither path lets AI apply changes or read business records. Web Designer remains the human setup, review, approval, and verification surface.

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

Every new App starts with zero Models. Add the Model explicitly; Apps named `erp`, `erp.credit`, or `web` receive no special default.

### 2. Create a dedicated AI-assistance user

Create an ordinary user specifically for AI-assisted customization. In that user's App Access, grant only the target App and set:

```text
canCustomize = true
```

This is sufficient for Designer inspection and validation. `canCustomize` does not grant App entry or business-data access.

Avoid assigning `FW_SystemAdminRole` or `FW_FrameworkUser` for routine AI work. `FW_SystemAdminRole` is the only global bypass; `FW_FrameworkUser` has been a legacy marker since v0.1.1.0.

**Alternative: an AI REST token.** A System Administrator can instead create a token under **Settings → Users & Security → AI REST tokens**. Choose the existing target App, the minimum scopes (`inspect`, `validate`, `propose`), and a short expiry. The secret is shown once and only its hash is stored. Tokens are scoped to Apps, not to Models or Layers, so the Skill enforces the Model/Layer you give it. There is no AI apply endpoint: a customizer reviews each proposal in **Web Designer → AI Proposals**.

Only when generated-App verification is genuinely required, add `canOpen=true` plus the minimum Role/Privilege permissions for the Forms, tables, Functions, Reports, or Views being tested.

### 3. Prepare authentication securely

Sign in using a browser session, or configure the dedicated username/password or the AI token in environment variables or a secret store outside the AI conversation. Never paste passwords, session cookies, API or AI tokens, setup codes, secret keys, or license material into a prompt.

## Information to give the AI

Provide this contract with every task:

```text
Endpoint: http://localhost:3399
Environment: development
App: sales
Model: AICustomizations
Layer: CUS
Credential: dedicated user with canCustomize=true for sales (or an AI REST token with inspect, validate, propose); no runtime access
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
2. Read capabilities: `/api/designer/capabilities`, or `/api/v1/ai/capabilities` plus the schemas.
3. Read the snapshot (`/api/designer/snapshot?app=<App>`) or every page of the workspace (`/api/v1/ai/workspace`).
4. Verify the App, Model, Layer, and account or token scope.
5. Build a version-1 change set with `source: "ai"`.
6. Validate it (`/api/designer/change-sets/validate` or `/api/v1/ai/change-sets/validate`).
7. AI REST path: submit it to `/api/v1/ai/proposals` and let a customizer approve it in AI Proposals. Designer path: present the preview ID, expiry, diff, warnings, schema effects, and risks, and let the human apply with the same dedicated username.
8. Refresh the snapshot and visually verify the generated App.

## What the Skill covers

Tables, enums, fields (including `multiline` and `encrypted`), Forms and Form Lines, menus, Views, Charts, paginated reports (paper sizes, units, layout version 2, borders, images), Translations and App-level default language awareness, Data Entities and append-only Data Entity Extensions, security artifacts, Scripts, and Functions (including `imageInput`) where capabilities allow.

It does not install or upgrade EmuFramework, edit framework source, build or import deployment packages, handle ISV licenses or keys, change users or tokens, or read or write business data unless you separately authorize a verification step.

Notable 1.4.0 behavior it enforces: lifecycle hooks and data event handlers must be synchronous (use an async Function for awaited work); new records open as drafts whose `initValue` runs at draft creation; `sys_createdBy`, `sys_createdAt`, `sys_modifiedBy`, and `sys_modifiedAt` are reserved virtual fields; datetimes are UTC; unauthorized actions are omitted from `/api/metadata`; and a licensed Model goes read-only when its license expires, so a failed verification can have a license cause.

## API support

EmuFramework v1.4.0 endpoints used by the Skill:

Designer API (session cookie from `POST /api/login`, `canCustomize` required):

- `GET /api/designer/capabilities` — version, revision, schemas, and AI capability flags
- `GET /api/designer/snapshot?app=` — current metadata revision and App artifacts
- `GET /api/designer/artifacts`, `/catalog`, `/customization/:kind/:name` — paginated listing, catalog, and inherited layers
- `POST /api/designer/change-sets/validate` — atomic validation and preview
- `POST /api/designer/reports/validate` — report layout validation
- `GET /api/designer/translations/diagnostics` — duplicate and stale translation keys
- `POST /api/designer/change-sets/apply` — human-confirmed apply; not called by the Skill

AI REST proposal API (Bearer token):

- `GET /api/v1/ai/capabilities`, `/schemas/artifact`, `/schemas/change-set`
- `GET /api/v1/ai/workspace?app=&model=&kind=&cursor=&limit=` — paginated metadata (default 100, maximum 500)
- `POST /api/v1/ai/change-sets/validate` — validate and receive a safe diff
- `POST /api/v1/ai/proposals` — place a validated change set in the Proposal Inbox

Runtime endpoints used only for separately authorized verification:

- `POST /api/data/:table/drafts` and `/drafts/:token/save`, `POST /api/data/:table` — business Records when table permissions permit
- `GET /api/views/:name/schema`, `/data`, `/export?format=csv` — permitted View contract, rows, and CSV export

The Skill prefers the change-set workflow because several direct artifact calls can leave a partially completed design if a later call fails. Direct artifact create/upsert/delete and deployment-package endpoints exist but are not part of the AI workflow.

View and Chart are supported metadata artifact kinds, with delta-only View, Chart, Function, and Data Entity Extensions. Interactive View verification requires all three runtime gates: `canOpen`, a View Privilege, and read permission for every source table. Chart access is inherited from its View. Power BI service tokens remain human-administered and outside the Skill's secret handling.

The capabilities response declares the live AI inspection, validation, apply, business-data, and executable-metadata policies. The Skill obeys those flags rather than bypassing them. The human owns apply or approval.

## Documentation baseline

The bundled guidance targets EmuFramework **v1.4.0** (`Major.Minor.Patch` versioning: FU Framework Updates and PU Proactive Updates), verified against the v1.4.0 framework source, the 1.0.0 to 1.4.0 release notes, and the official [EmuFramework documentation](https://github.com/emu479p01/emu-framework-docs). Legacy four-component versions such as `0.1.4.0` and `0.5.0.0` are history. The version, capabilities, and schemas reported by the running instance are authoritative.
