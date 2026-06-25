# apiext-format

`.apiext` is an extended element-API format for WebObjects (and ng-objects) dynamic elements — a strict
superset of WO's `.api` format that adds the documentation an element's bindings deserve: per-element and
per-binding descriptions, binding types, directionality, and value interpretation. Enough to render an
element's API in an editor.

This repository is the **format specification**: the grammar, the field-by-field spec, illustrative
examples, and status. It is implementation-agnostic — a reference parser may live here later, but the
format is defined independently of any consumer.

## Contents

| File | What it is |
|---|---|
| [`apiext.dtd`](./apiext.dtd) | The grammar. A superset of the legacy `.api` grammar — existing `.api` files validate against it. |
| [`FORMAT.md`](./FORMAT.md) | Field-by-field specification with examples. |
| [`DONE.md`](./DONE.md) | What the format expresses today. |
| [`ROADMAP.md`](./ROADMAP.md) | Remaining work, each item linked to its issue. |
| [`legacy-api-format/`](./legacy-api-format) | A precise record of the classic WebObjects `.api` format `.apiext` extends — the `.api` DTD and a detailed, corpus-verified description. |
| [`examples/`](./examples) | Illustrative `.apiext` files (not shipping elements) demonstrating specific features. |

## In brief

`.api` declares an element's bindings — names, a few flags, cross-binding validation — and nothing about
what a binding means, its type, or its direction. `.apiext` adds that as a strict superset: existing
`.api` files validate unchanged, and the new information is additive. The WO runtime does not read `.api`
(only tooling does — WOBuilder, WOLips, Parsley), so a successor format is constrained only by "don't
break existing files".

```xml
<wodefinitions>
  <wo class="WOCheckBox" wocomponentcontent="false" passthrough="true">

    <doc><![CDATA[A checkbox. Use **either** `checked` or `value`+`selection`.]]></doc>

    <binding name="checked">
      <pull><type interpretation="truthy">java.lang.Object</type></pull>
      <push><type>java.lang.Boolean</type></push>
      <doc>The checked state.</doc>
    </binding>

  </wo>
</wodefinitions>
```

See [`FORMAT.md`](./FORMAT.md) for the full walkthrough — documentation, types, directionality
(`<pull>`/`<push>`), and interpretation (`interpretation="truthy"`).

## Status

Proposal stage; in use. The format drives the AjaxSlim element reference (in the
[wonder-slim](https://github.com/undur/wonder-slim) repository) and a WOLips plugin preview. The
vocabulary is still settling — see [`DONE.md`](./DONE.md) for what's present and [`ROADMAP.md`](./ROADMAP.md)
for what's open.
