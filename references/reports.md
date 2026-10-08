# Reports

Use this reference when a task creates or changes a `report` artifact. There is no `reportExtension` kind in v1.4.0. Overriding an inherited report means a same-named replacement in a higher-Layer Model, which is effectively a copy. Prefer a new report and say so in the proposal.

Obtain the exact shape from capabilities. A report reads business tables at runtime under the caller's permissions, so grant the report name and source-table `read` in a Privilege.

## Page, units, and layout version

```json
{
  "kind": "report",
  "name": "SALES_OrderListReport",
  "app": "sales", "model": "Core", "layer": "ISV",
  "dataSource": "SALES_Order",
  "layoutVersion": 2,
  "designUnit": "cm",
  "defaultFont": "Noto Sans Thai",
  "page": { "size": "A4", "orientation": "portrait", "margins": [40, 40, 40, 40] },
  "bands": []
}
```

- `page.size`: `A3`, `A4`, `A5`, `Letter`, `Legal`, or `Custom`. `Custom` requires positive `page.width` and `page.height`. `landscape` swaps width and height.
- Stored geometry is always PostScript points: page width/height, margins `[top, right, bottom, left]`, band heights, element `x/y/width/height`, and column widths. A3 is 842 x 1191 pt, A4 595 x 842, A5 420 x 595, Letter 612 x 792, Legal 612 x 1008.
- `designUnit` (`cm`, `in`, `px`) is a Web Designer display/input preference only. Never convert stored geometry into it. For explanations: 1 in = 72 pt, 1 cm = 28.35 pt, 1 px = 0.75 pt.
- Emit `layoutVersion: 2` for every new or changed report. Version 2 turns printable-area findings into errors. Version 1, the default when omitted, only warns, so an overflowing legacy report still previews. A report saved from the Designer becomes version 2.

## Layout version 2 strict validation

Compute usable width (page width minus left and right margins) and usable height in points before placing elements. The validator reports:

- Always errors: `Custom` paper without positive width/height, negative margin, margins that leave no printable area, negative band height, negative or non-finite element geometry, and an image element without a valid source.
- Errors under version 2 (warnings under version 1): an element wider than the usable width or taller than its band, Tablix columns whose explicit widths sum past the usable width, and header plus footer band heights above the usable height.

Other structural rules: `tablix` is allowed only on `detail` bands and requires `elements: []`; `displayOn` applies only to header/footer; every `field`, Tablix column, parameter, and line-source relationship must exist. Encrypted fields cannot be rendered in a report or used as a report parameter.

## Borders and styles

Text, field, and rect element `style` accepts `fontSize`, `bold`, `italic`, `fontFamily`, `align`, `color`, `borderWidth` (non-negative), `borderColor`, and `borderStyle` (`solid`, `dashed`, `dotted`, `none`). A Tablix accepts `border: { width?, color? }`, `headerStyle`, and `rowStyle` (font properties, `backgroundColor`, `padding`). Unknown properties are rejected.

## Images and assets

An element with `type: "image"` needs an `image` object:

- `source: "asset"` with an `assetId` that matches an entry in the report's `assets` array.
- `source: "attachment"` with `attachmentIdField` (a field on the current record holding an attachment ID) or `attachmentName` (select a record attachment by display name).
- Optional `fit` (`stretch`, `contain`, `cover`, `original`), `horizontalAlign`, and `verticalAlign`.

`assets` embeds PNG or JPEG data as `{ id, name, mimeType, dataBase64 }`, keeping packages self-contained. Never fabricate or fetch image data. Reuse an existing asset ID from the workspace, use an attachment source, or ask the user to add the image through Web Designer. Do not read attachment contents or business data to test an image.

## Validate a report

On the Designer path, `POST /api/designer/reports/validate` with `{ "artifact": <full report> }` returns `valid`, `diagnostics` (layout findings with `severity`, plus missing-font warnings), and a `summary`. Run it before ChangeSet validation and resolve every `error`. A font that is not installed only warns; PDF falls back to Roboto. The AI REST path has no report endpoint; rely on ChangeSet validation there.

A human verifies PDF output, pagination, and mixed Thai/Latin rendering through the generated App.
