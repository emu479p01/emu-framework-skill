# Views and Charts

Use this reference when a feature needs a declarative query, a reusable visualization, Form embedding, or a Power BI/service-token integration boundary.

## Build a View without SQL

A View is a virtual query executed at request time. Use only the schema returned by `/api/designer/capabilities`; never submit raw SQL, a subquery, window function, or computed expression.

Supported v0.1.1.0 concepts are:

- one source `{ table, alias }`
- `inner` or `left` joins with field-equality `on` pairs
- output columns from an aliased field or `count`, `sum`, `avg`, `min`, or `max`
- typed `string`, `int`, `real`, `boolean`, `date`, or `datetime` parameters
- filters whose value is a literal, literal array, or named parameter
- `groupBy` field references and `orderBy` output-column names

Use aliased field references such as `o.customerId`. Define parameters before filters that reference them. Group every selected non-aggregate field when aggregate columns are present. The validator rejects unknown/protected tables, unknown fields, incompatible types, duplicate aliases/columns, invalid grouping, and undeclared cross-App dependencies.

## Secure a View

An interactive session needs every gate:

1. `FW_AppAccess.canOpen=true` for the View's owning App.
2. A Role/Duty/Privilege whose `views` includes the View name.
3. Table `read` permission for every source and joined table.
4. All row scopes applicable to those tables.

Do not infer View access from table read permission or vice versa. Metadata filtering is not the security boundary; direct schema, data, and export requests must also deny unauthorized callers.

The runtime endpoints are:

```text
GET /api/views/:name/schema
GET /api/views/:name/data?param.<name>=...&limit=...&offset=...
GET /api/views/:name/export?format=csv&param.<name>=...
```

JSON defaults to 1,000 rows and caps one page at 10,000. CSV uses the configured server cap, 100,000 by default.

## Build a Chart

A Chart references one View and maps its output columns:

- `type`: `bar`, `line`, `pie`, `donut`, or `kpi`
- `dimension`: required for non-KPI Charts and must name a View output column
- `measures`: output fields with optional labels/colors
- optional `legend` and `stacked`

A KPI must have exactly one measure. Chart permission is inherited from its View; there is no separate Chart list in a Privilege.

To embed the Chart, add `FormMeta.charts` or a Form Extension contribution. Choose `half` or `full` width and bind each required View parameter from a current-record field or literal. Validate field/parameter type compatibility, duplicate bindings, required bindings, and App dependencies. A missing required record value on an unsaved record should produce the save-first state, not a malformed request.

## Keep service tokens human-administered

Power BI tokens are created/revoked only by a System Administrator through **Users & Security**. They are high-entropy, stored as hashes, shown once, scoped to named Views, optionally expiring, and accepted only in an HTTPS `Authorization: Bearer ...` header.

The App-building skill may design and validate a View but must not create, capture, print, persist, or place a service token in a URL. A service token can call only scoped `/api/views/*` endpoints and cannot call Chart, Data, Designer, or administration APIs.

## Verification

Test valid joins, parameters, filters, grouping/aggregates, invalid references/types, injection attempts, row limits, dependencies, source-table denial, row scopes, and unauthorized direct calls. When a human separately tests service tokens, cover valid, wrong-scope, expired, and revoked tokens. For Charts, cover each type, missing bindings, unauthorized Views, responsive resize, and component cleanup.
