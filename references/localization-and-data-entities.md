# Localization and Data Entities

Use this reference for App `defaultLocale`, `translation` artifacts, `dataEntity`, and `dataEntityExtension`. All three are metadata kinds in v1.4.0 (translation and Data Entity since 1.1.0; `defaultLocale` and `dataEntityExtension` since 1.2.0). Confirm the live schema through capabilities before use.

## App default locale

`defaultLocale` is an optional App manifest property: a BCP-47 tag canonicalized with `Intl.getCanonicalLocales` (`th-th` becomes `th-TH`). Apps without it use `en`.

- It is an App-level setting, not a Model/Layer artifact. Do not upsert the App manifest to set it; ask the user to set **Default language** in the Web Designer App editor.
- Read it from the App manifest in the snapshot/workspace before writing a translation, so you know which locale is the owner fallback.
- The stored `label` stays the last-resort text, even when written in another language.

## Translation artifacts

A `translation` is a base artifact (not an Extension) with common placement plus:

```json
{
  "kind": "translation",
  "name": "SALES_Thai",
  "app": "sales",
  "model": "AICustomizations",
  "layer": "CUS",
  "locale": "th",
  "resources": {
    "form.SALES_OrderForm.label": "ใบสั่งซื้อ",
    "table.SALES_Order.field.deliveryNote.label": "หมายเหตุการจัดส่ง"
  }
}
```

- `locale` is a BCP-47 tag (`^[A-Za-z]{2,3}(-[A-Za-z0-9]{2,8})*$`) and is canonicalized on load. An invalid tag is rejected.
- `resources` maps resource keys to text. Omit a key to leave the stored label; an empty cell in the Designer means "do not override". Never submit empty strings as placeholders.
- Several Translations per locale are allowed (per Model or Layer). Name them uniquely with the App prefix, for example `SALES_<Model>_<Locale>`.
- Put the Translation in the fixed Model/Layer. Never edit a lower-layer Translation or the framework `FW_UiEn`/`FW_UiTh` Translations.

### Resource keys

Generate keys exactly as below. A key whose target does not exist is not a load failure; it surfaces as a stale-key warning.

| Target | Key |
| --- | --- |
| App / Model | `app.<app>.label` / `model.<app>.<model>.label` |
| Table / Field | `table.<table>.label` / `table.<table>.field.<field>.label` |
| Enum / Value | `enum.<enum>.label` / `enum.<enum>.value.<valueName>.label` |
| Form | `form.<form>.label` |
| Form group / action / line | `form.<form>.group.<id>.label` / `form.<form>.action.<id>.label` / `form.<form>.line.<id>.label` |
| Line action | `form.<form>.line.<lineId>.action.<actionId>.label` |
| Menu / item | `menu.<menu>.label` / `menu.<menu>.item.<itemId>.label` |
| Report | `report.<report>.label` |
| Report parameter / text element | `report.<report>.parameter.<field>.label` / `report.<report>.element.<elementId>.text` |
| View, Chart, Privilege, Duty, Role, Data Entity | `<kind>.<name>.label` |

Group, action, line, and menu-item keys use the stable `id` values, so give those elements stable IDs. System field labels (`table.<table>.field.createdAt.label`, `sys_*`) are valid field keys.

### Reserved keys and dependencies

- Keys starting with `ui.` are reserved for the framework. Only system-app Translations may define them. Never write `ui.*` keys.
- A Translation may override another App's artifact only when its own App declares a direct or transitive dependency on that App. Otherwise the resource is silently ignored. Verify `dependsOn` in the manifest; do not edit it.

### Layer resolution and fallback

For one key and locale, the highest Layer wins (`SYS < ISV < LOC < DEV < CUS`); equal Layers are decided by Translation name, so load order never matters. Per key the resolver tries: exact user locale, its base language, owner App `defaultLocale`, its base language, the stored label, then the component name. A partial Translation affects only the keys it covers.

### Verification

`GET /api/designer/translations/diagnostics` returns duplicate-key and missing-target warnings (Designer session only). Resolve missing targets; treat duplicates as intentional only when a higher Layer overrides a lower one on purpose. `GET /api/metadata` returns localized labels for the signed-in user, so use the Designer snapshot or workspace for stored labels, never `/api/metadata`.

## Data Entities

A `dataEntity` describes a header/lines document for XLSX or CSV-ZIP exchange:

```json
{
  "kind": "dataEntity",
  "name": "SALES_OrderEntity",
  "app": "sales",
  "model": "Core",
  "layer": "ISV",
  "rootTable": "SALES_Order",
  "businessKey": ["orderNo"],
  "fields": ["orderNo", "customerId", "amount"],
  "lines": [
    { "name": "Lines", "table": "SALES_OrderLine", "parentReference": "orderId",
      "fields": ["lineNo", "itemId", "quantity"], "lineKeys": ["lineNo"] }
  ]
}
```

Required: `rootTable`, non-empty `businessKey`, non-empty `fields`. Optional: `lines`, `archiveEligible`, `businessDateField`, `label`. The registry enforces:

- `rootTable`, every root field, and every line table/field must exist; system fields and `id` are allowed.
- Every `businessKey` field must also be in `fields`.
- Each line needs `name`, `table`, `parentReference`, non-empty `fields`, and non-empty `lineKeys`; every key must also be in that line's `fields`.
- `parentReference` must be a `reference` field on the line table that targets `rootTable`.
- `archiveEligible: true` requires `businessDateField`, which must be a `date` or `datetime` field.
- Encrypted fields are silently excluded from exchange. Do not list them.

Rules for AI:

- Define the tables and reference fields first; the Data Entity cannot create them.
- Leave `archiveEligible` unset unless the user explicitly asks. Archive policies are administrator-configured and move business data.
- Data Entity runtime endpoints (`/api/data-entities/*`) read and write business data. Never call them; `ai.businessData=false` stays in force. Access follows table permissions: export needs `read` on the root and line tables, import needs `create` and `update`.
- Secure the underlying tables through Privileges. A Data Entity has no Privilege list of its own.

## Data Entity Extensions

A `dataEntityExtension` appends to a Data Entity from a strictly higher Layer:

```json
{
  "kind": "dataEntityExtension",
  "name": "SALES_AICustomizations_SALES_OrderEntity_Extension",
  "app": "sales",
  "model": "AICustomizations",
  "layer": "CUS",
  "dataEntity": "SALES_OrderEntity",
  "fields": ["deliveryNote"],
  "lineExtensions": [{ "name": "Lines", "fields": ["remark"] }]
}
```

Append-only merge rules:

- `fields` appends root fields. Each must not already be in the entity and must exist on the root table.
- `lines` appends whole new lines. The line `name` must not collide with an existing line.
- `lineExtensions` appends fields to existing lines, matched by line `name`. Each field must not already be on that line; an unknown line is an error.
- The schema rejects any other property. `rootTable`, `businessKey`, existing line tables, `parentReference`, `lineKeys`, and archive settings can never change.
- The merged entity is validated as a whole after every Extension applies.
- Add the table field first with a `tableExtension` in the same ChangeSet. A new mandatory table field without a default makes restoring older archives fail clearly while the archive is preserved, so prefer optional fields or defaults.
- Follow the normal Extension rules: name ends `_Extension` (canonical `<AppPrefix>_<Model>_<Target>_Extension`), Layer strictly higher than the target, one per `(app, model, kind, target)`, declared dependency for cross-App targets.
