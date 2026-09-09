# Business logic

Use this reference for hooks, events, Scripts, Functions/actions, transactions, and integrations.

## Select the smallest mechanism

| Requirement | Mechanism |
| --- | --- |
| Field default | Field `default` or `initValue` hook |
| Write/delete validation | Hook or cancellable pre-event |
| Lifecycle reaction | Data event |
| One explicit user operation | Function/action |
| Several related registrations | Script |
| Bounded HTTP or email integration | Async Function |
| Complex reusable/native integration | Reviewed TypeScript |

Place validation at the server-side data boundary so API, Script, Function, tests, and internal services share the rule.

## Write lifecycle

```text
permission -> field validation -> onValidating -> validateWrite
           -> onInserting/onUpdating -> SQLite write -> onInserted/onUpdated
```

Pre-events can cancel. Post-events cannot cancel a completed operation. Reference deletion can be `restrict`, `setNull`, or `cascade`; child writes still follow their normal lifecycle.

## Scripts

Use a Script as a short deterministic registration unit. In v0.1.4.0, dynamic Script code receives only `kernel`, `ValidationError`, and `DataEventCancelled`; do not assume arbitrary imports or globals. Register hooks/events/actions through the kernel.

- Keep names unique and do not rely on incidental registration order.
- Use the authenticated context supplied to handlers.
- Keep credentials and production data out of code.
- Prefer a Function for a public explicit action.
- Treat Script changes as trusted administrative code.

## Functions

A Function is a named server operation invoked through the action registry/API. Its body receives `(ctx, args, kernel, services)`. Add every Function to a privilege; a form action alone does not grant access.

| Mode | Use | Boundary |
| --- | --- | --- |
| `transactional` | Synchronous validation and related writes | Whole handler is one transaction; throwing rolls back |
| `async` | Work requiring `await`, HTTP, or email | No automatic spanning transaction; use short `ctx.tts()` blocks |

Rules:

- Never return a Promise from a transactional Function.
- Never await HTTP, SMTP, or other network work inside `ctx.tts()`.
- Use the supplied authenticated `ctx` for reads and writes.
- Validate arguments, record existence/state, and target-record access.
- Return correctable problems as `ValidationError` when supported.
- Do not expose secrets, SQL details, internal stacks, remote sensitive headers, or bodies.

## Bounded services in v0.1.4.0

`services.http.request(...)` supports HTTP/HTTPS, headers, JSON or text bodies, and a bounded timeout. The documented default timeout is 15 seconds, the maximum is 60 seconds, and responses over 5 MB reject. Non-success HTTP status does not reject automatically; check `ok` or `status`.

`services.email.send(...)` uses the administrator-configured SMTP transport. It requires at least one recipient and text or HTML content; the documented limits are 50 recipients and 1 MB of message content. Inspect rejected recipients.

Do not embed credentials in metadata. Use only the integration mechanisms supported by the installed version.

## Test logic

Cover success, invalid input, missing records, rollback after related-write failure, allowed and denied callers, update/delete/reference paths, duplicate action names, and registration interactions.

For async Functions also cover connection errors, timeouts, non-success status, response limits, missing SMTP configuration, rejected recipients, and database state when a service fails before or after each explicit transaction.
