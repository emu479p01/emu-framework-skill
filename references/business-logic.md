# Business logic

Use this reference for hooks, events, Scripts, Functions/actions, drafts, transactions, and integrations. Behavior below is verified against EmuFramework v1.4.0.

## Select the smallest mechanism

| Requirement | Mechanism |
| --- | --- |
| Field default | Field `default` or `initValue` hook (synchronous) |
| Write/delete validation | `validateWrite`/`validateDelete` hook or cancellable pre-event (synchronous) |
| Lifecycle reaction | Data event handler (synchronous) |
| One explicit user operation | Function/action |
| Several related registrations | Script |
| Awaited work, bounded HTTP or email | Async Function (`executionMode: 'async'`) |
| Complex reusable/native integration | Reviewed TypeScript |

Place validation at the server-side data boundary so API, Script, Function, tests, and internal services share the rule.

## 1.4.0 breaking change: lifecycle handlers are synchronous

`initValue`, `validateWrite`, `validateDelete`, and every data event handler (`onValidating`, `onInserting`, `onInserted`, `onUpdating`, `onUpdated`, `onDeleting`, and `onDeleted`) run inside the database transaction and must be synchronous.

- Never write `async` hooks or handlers, use `await`, or return a Promise or thenable.
- Registering an `async` function, or running a handler that returns a Promise, raises `async lifecycle handlers are not supported`. Existing Scripts with async lifecycle code that worked before 1.4.0 now fail.
- A hook that needs HTTP, email, or any awaited call is the wrong mechanism. Move the work into an async Function and have the user or a Form action invoke it. Keep the hook to local validation and writes through the supplied `ctx`.
- `ctx.tts(fn)` callbacks must also be synchronous; a returned Promise raises the same error.
- Change-set validation does not run or compile Script or Function code (bodies are blanked in the preview). The error appears only when the Script registers at apply or load time, or when the handler runs. Review every hook for `async`, `await`, and returned Promises before submitting.

Audit the existing Scripts of the target Model before extending them, and report any async lifecycle handler you find instead of rewriting it silently.

## Write lifecycle

```text
permission -> field validation -> onValidating -> validateWrite
           -> onInserting/onUpdating -> SQLite write -> onInserted/onUpdated
```

Pre-events can cancel. Post-events cannot cancel a completed operation. Reference deletion can be `restrict`, `setNull`, or `cascade`; child writes still follow their normal lifecycle. Insert, update, and delete outside an explicit transaction open one automatically.

## Drafts and `initValue`

Opening **New** no longer inserts a row. The client calls `POST /api/data/:table/drafts`, which runs field defaults, the supplied initial values, and every `initValue` hook inside a transaction, then stores an encrypted, user-bound snapshot and returns `{ token, expiresAt, record }`. `POST /api/data/:table/drafts/:token/save` merges writable values over the snapshot and inserts the record once.

Design consequences:

- `initValue` runs at draft creation, not at save, and does not run again on save or retry. Do not depend on user-entered values in `initValue`.
- Anything `initValue` writes elsewhere (for example a number-sequence counter) is committed when the draft is created, even if the user never saves. Expect gaps and keep sequence logic idempotent where it matters.
- A read-only field supplied by `initValue`, such as a document number, is shown immediately and cannot be overwritten by the request body. A read-only field may also be `mandatory` when `initValue` supplies it (see the schema caveat in [metadata-design.md](metadata-design.md)).
- Drafts expire after 24 hours and survive a restart. An expired token, another user's token, or a draft created before a table, Script, or Function metadata change is rejected with `409`; the user opens a new draft. Changing those artifacts invalidates every open draft, so batch related changes into one change set.
- Create permission and license state are checked at draft creation and again at save. A failed save rolls back and leaves the draft usable.
- Direct `POST /api/data/:table` also runs `initValue`. Test both paths only when runtime verification is authorized.

## Audit fields and time in logic

- `createdAt/By` and `modifiedAt/By` are framework-managed. The virtual aliases `sys_createdBy`, `sys_createdAt`, `sys_modifiedBy`, and `sys_modifiedAt` map to them and can be read in hooks, `Query.where/orderBy`, list filters, and Views. Never write them; updates preserve the original creation audit and ignore forged values.
- Datetimes are UTC ISO strings (`2026-08-15T15:52:39.000Z`). A value without an offset is interpreted as UTC. Write UTC in Scripts and Functions; do not format local time into stored values.
- `encrypted` string fields cannot be filtered, searched, or sorted with `Query`. Do not copy their values into unencrypted fields, logs, or error messages.

## Scripts

Use a Script as a short deterministic registration unit. Dynamic Script code is compiled as `new Function('kernel', 'ValidationError', 'DataEventCancelled', code)` and receives only those three names; do not assume imports, `require`, `process`, `services`, or other globals. v1.4.0 did not change this signature. Register through the kernel:

- `kernel.hooks.register(table, { initValue, validateWrite, validateDelete })`
- `kernel.events.on(table, eventName, handler)`
- `kernel.actions.set(name, handler)` (prefer a Function artifact for a public action)

Every registration is bound to the owning App. If a licensed Model's license is missing or expired, writes through its hooks and actions fail with `App '<app>' is read-only` even when another App triggers them.

Rules:

- Keep names unique and do not rely on registration order.
- Use the authenticated context supplied to handlers.
- Keep credentials and production data out of code.
- Treat Script changes as trusted administrative code. Submit them only when the live capabilities allow (`ai.scripts` on the Designer path, `executableArtifacts` on the AI REST path) and the task needs them.

## Functions

A Function is a named server operation invoked as `POST /api/action/:name`. Its body receives `(ctx, args, kernel, services)`. A Function Extension body receives `(ctx, args, next, kernel, services)` and must call `next()` exactly once, otherwise the call fails. Add every Function to a Privilege; a Form action alone does not grant access, and unauthorized actions are omitted from `/api/metadata` while direct calls still return `403`.

| Mode | Use | Boundary |
| --- | --- | --- |
| `transactional` | Synchronous validation and related writes | Whole handler is one transaction; throwing rolls back |
| `async` | Work requiring `await`, HTTP, or email | No automatic spanning transaction; use short synchronous `ctx.tts()` blocks |

Rules:

- Never return a Promise from a transactional Function.
- Never await HTTP, SMTP, or other network work inside `ctx.tts()`.
- Use the supplied authenticated `ctx` for reads and writes.
- Validate arguments, record existence and state, and target-record access.
- Return correctable problems as `ValidationError`.
- Do not expose secrets, SQL details, stacks, or remote sensitive headers and bodies.

### Image input

A Function may declare `imageInput: { "table": "SALES_Order", "recordIdArgument": "recordId", "multiple": true }`:

- `table` must be an existing business table, never `FW_*`. `recordIdArgument` is an identifier naming the argument that carries the record ID, and it cannot be `attachmentIds`, `__proto__`, `constructor`, or `prototype`.
- The Function dialog lets the user pick files or take photos (JPEG, PNG, or WebP). Images upload as attachments of the record with a client `uploadId` so retries do not duplicate them.
- Before the code runs, the server checks Function permission, update permission on the record, file signature, and that each attachment belongs to that record and the caller. The code receives `args.attachmentIds` (array of attachment ID strings; exactly one when `multiple` is not true) and a numeric record ID in `args[recordIdArgument]`.
- Uploaded images stay attached when the Function fails. Do not design the Function to assume rollback of uploads.
- `/api/metadata` publishes only `name`, `label`, and `imageInput` for permitted Functions in `functionInputs`; code is never sent to the client.
- Treat images as business data. Do not send them to external HTTP endpoints without explicit user approval, and do not log them.

## Bounded services in v1.4.0

`services` is passed to Functions and Function Extensions only. Hooks, data events, and Scripts do not receive it, so they cannot make HTTP or email calls. The v1.4.0 kernel changes did not alter the Script signature or the Function/service contract below.

`services.http.request(...)` supports HTTP/HTTPS, headers, JSON or text bodies, and a bounded timeout. The default is 15 seconds, the maximum is 60 seconds, and responses over 5 MB reject. A non-success HTTP status does not reject automatically; check `ok` or `status`.

`services.email.send(...)` uses the administrator-configured SMTP transport. It requires at least one recipient and text or HTML content; limits are 50 recipients and 1 MB of message content. Inspect `rejected` recipients. SMTP passwords and other secrets are encrypted at rest with the installation's `.emu-secret.key`.

Do not embed credentials in metadata. Use only integration mechanisms supported by the installed version. External side effects (email, HTTP) cannot be rolled back with SQLite, and an async action cannot undo effects performed before a license expires.

## Test logic

Cover success, invalid input, missing records, rollback after related-write failure, allowed and denied callers, update/delete/reference paths, duplicate action names, and registration interactions. Add draft cases: `initValue` output on a new draft, save, retry, expired token, and reopen after a metadata change.

For async Functions also cover connection errors, timeouts, non-success status, response limits, missing SMTP configuration, rejected recipients, and database state when a service fails before or after each explicit transaction. For image Functions cover wrong record, wrong user, unsupported type, and failure after upload.
