# `.apiext` — what's done

What the format expresses today, and the milestones behind it. For remaining work see
[`ROADMAP.md`](./ROADMAP.md); for the format itself see [`FORMAT.md`](./FORMAT.md) and
[`apiext.dtd`](./apiext.dtd).

---

## Shipped

The `.apiext` extended element-API format is live and consumed end-to-end.

- **DTD** — [`apiext.dtd`](./apiext.dtd): a **superset** of the legacy `.api` grammar (see
  [`legacy-api-format/`](./legacy-api-format)). Validates existing `.api` files and the `.apiext` files.
- **Spec doc** — [`FORMAT.md`](./FORMAT.md): field-by-field semantics for tooling.
- **Reference corpus** — all 14 AjaxSlim elements have authored `.apiext` files (in the
  [wonder-slim](https://github.com/undur/wonder-slim) repository).
- **Consumers** — AjaxSlim's element reference renders entirely from `.apiext`; the WOLips/Eclipse fork
  reads `.apiext` and renders a tag's API preview inline in the template editor.
- **Markdown in `<doc>`** — an injection-safe subset (inline code, bold, italic, links, fenced code
  blocks).

## What the format expresses today

- **Element** (`<wo>`): `class`, `wocomponentcontent`, `passthrough`, `<doc>` (role, Markdown).
- **Per binding** (`<binding>`): `name`, `required`, `settable`, `defaults`, and `<doc>` (Markdown).
- **Types**: one or more `<type>` per binding (a fully-qualified Java class or a value-set name), inside a
  direction block.
- **Directionality**: `<pull>` / `<push>` blocks — presence declares the direction; the type can differ
  per direction (e.g. a checkbox pulls a truthy `Object`, pushes a `Boolean`).
- **Interpretation**: `<type interpretation="truthy">` — names a coercion rule (read-as-boolean) alongside
  the type without changing it; validation uses the type, the interpretation is a display qualifier.
- **Cross-binding rules**: `<validation>` with the full `and`/`or`/`count`/`bound`/`unbound`/`ungettable`/
  `unsettable` vocabulary, carried verbatim from `.api`.
