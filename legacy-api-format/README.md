# The legacy WebObjects `.api` format

A precise record of the classic WebObjects `.api` binding-declaration format, as it **actually exists** —
drawn from the element-definitions catalog in the open-source WOLips IDE and the `.api` files distributed
with WebObjects frameworks. The grammar is captured in [`legacy-api.dtd`](./legacy-api.dtd).

This is the format `.apiext` extends. Understanding the past precisely — including its relics and the
gaps between what the grammar *declares* and what the files *do* — anchors every decision about the
future. This document is **descriptive, not prescriptive**: it records what is, not what should be.

## What `.api` is

One XML file per element/component, named `<ClassName>.api`, declaring a dynamic element's binding
interface for IDE tooling (WOBuilder, WOLips). The WO runtime does not consume them — they ship inside
framework jars purely as resources so tooling can discover them. That gives a successor format real
freedom: the only hard constraint is not breaking the files that exist.

## Grammar

The grammar is [`legacy-api.dtd`](./legacy-api.dtd). The vocabulary:

```
wodefinitions → wo → (binding | validation | documentation)*
validation    → predicate tree of and / or / count over bound / unbound / ungettable / unsettable
```

All attributes are `CDATA #IMPLIED` (untyped, optional strings). The DTD constrains structure, never
value types — booleans are strings, and which strings is not enforced (see the casing quirk below).

---

## 1. Declared and used — the live core

### `<wodefinitions>` (root)

Contains one or more `<wo>`. In practice always exactly one per file.

### `<wo>` — the element definition

| Attribute | Notes |
|---|---|
| `class` | The **simple class name** (`WOString`), never fully-qualified — 222 distinct in the 5.4 corpus. A relic from the Objective-C era, before Java packages, when a simple name was a globally-unique class reference. (Tooling resolves it by classpath scan.) |
| `wocomponentcontent` | Whether the element wraps child content. A boolean — but **inconsistently spelled**: all four of `YES`, `NO`, `true`, `false` occur in the wild. |

Children: `binding`, `validation`, `documentation`.

### `<binding>` — one binding

`<binding>` is `#PCDATA` only — **it carries no child elements** in legacy `.api`. Everything an element
needs beyond a name and a few flags (types, documentation, directionality) is simply absent. This empty
slot is precisely what `.apiext` extends.

| Attribute | Notes |
|---|---|
| `name` | The binding name. Present on essentially every `<binding>` (158 distinct names in the 5.4 corpus). |
| `defaults` | Autocomplete value-source hint — see §3. A closed set of named sources. |
| `required` | Boolean. *Unused in the WebObjects-shipped `.api` files*; live in the wider Wonder corpus, inconsistently cased (`YES` / `yes` / `no`). |
| `settable` | Boolean — "push-capable" (the element writes to this binding). Same pattern: WO-unused, Wonder-used (`YES`). |
| `passthrough` | In the DTD; effectively unused. |

### `<validation>` — a cross-binding rule

A `message` attribute plus a predicate tree. **The semantics are inverted and implicit**, which is the
single most confusing thing about the format:

- A validation's message **fires when its predicates *hold*** — so the rule describes the **invalid**
  state, not the valid one. `<validation message="'a' and 'b' cannot both be bound"><bound name="a"/><bound name="b"/></validation>`
  reads "error *if* both a and b are bound".
- Bare predicates directly under `<validation>` are **implicitly AND-ed**.

The overwhelming majority of validations are simple "*'x' is a required binding*" forms (a single
`<unbound name="x"/>` predicate).

### Predicates — `<bound>` / `<unbound>` / `<ungettable>` / `<unsettable>`

Each takes a `name` attribute. The two obscure ones are worth spelling out, because they are widely
misunderstood:

| Predicate | True when binding `name` is… |
|---|---|
| `<bound name="x"/>` | bound to something |
| `<unbound name="x"/>` | not bound |
| `<unsettable name="x"/>` | bound, **but to a value the element cannot push back to** (a constant or non-settable keypath) — used to require that, e.g., `item` in a repetition is settable. |
| `<ungettable name="x"/>` | bound, **but to a value the element cannot pull from** — the read-side mirror of `unsettable`. **Declared but never used** in the corpus (the format reserved a capability nobody exercised). |

### Combinators — `<and>` / `<or>` / `<count test="…">`

`<and>` and `<or>` nest predicates. `<count test="…">` is the most obscure construct:

- It fires when the **number of true child predicates** satisfies the `test` comparison — e.g. a `test`
  meaning "more than one" expresses mutual exclusion across a set ("no more than one of a, b, c may be
  bound"), which a flat pairwise `<bound>`/`<bound>` AND cannot.
- The `test` mini-syntax is **undocumented**, and `<count>` is barely used in the corpus (a handful of
  files), so the exact grammar `test` accepts is unverified from the public files.

---

## 2. Declared but unused — reserved, never exercised

These are in the DTD but appear **nowhere** in the WebObjects-shipped `.api` corpus (some are used in the
wider Wonder ecosystem, noted above):

- **`<documentation directory|domain|path>`** — a pointer to external documentation resources. Never used
  in the corpus. The format reserved a documentation mechanism, but it was a *resource pointer*, not
  inline prose — and nobody populated it. (`.apiext`'s `<doc>` is the inline answer to the same need.)
- **`<ungettable>`** — the read-side validation predicate (above). Declared, never used.
- **`binding passthrough`** — declared, effectively unused.

The lesson for the successor: a declared capability that no file uses is weak evidence it was the right
shape. `.apiext` re-derives the documentation need (inline `<doc>` with Markdown) rather than inheriting
the unused `<documentation>` resource-pointer.

---

## 3. The `defaults` value-source vocabulary

`defaults` is the one place legacy `.api` types binding *values* (not just binding names). It is **not** a
default value — it names a **source the IDE draws autocomplete suggestions from**. The 5.4 corpus uses a
closed, hardcoded set of sources:

| `defaults=` | The IDE offers… |
|---|---|
| `Boolean` | `true` / `false` |
| `Actions` | action methods on the component |
| `Page Names` | WOComponent subclasses in the project |
| `Resources` | existing project resources (images, files) |
| `Frameworks` | framework names |
| `Date Format Strings` | date-format patterns |
| `Number Format Strings` | number-format patterns |

This is dynamic, project-aware value resolution — but **closed and hardcoded into WOLips**, not
user-extensible. It is the direct ancestor of [apiext-format#4](https://github.com/undur/apiext-format/issues/4)
(value sets, literal *and* dynamic). Note the name collides with a *default value* concept (which `.api`
has no way to express); `.apiext` keeps `defaults` (the source hint) and adds `<default>` (the value) as
separate things — see [apiext-format#3](https://github.com/undur/apiext-format/issues/3).

---

## 4. Used in practice but **not** in the DTD

> The grammar a format *declares* and the grammar its files *use* are not always the same. The item below
> was used across the corpus without ever being added to the DTD — so those files do not validate against
> the grammar. It documents a real need the format met *outside* its declared vocabulary, and may well
> have been a de-facto standard the DTD simply never ratified.

### `<image path="…">` — a palette/preview icon

```xml
<wo class="D2WInspect" wocomponentcontent="NO">
  <image path="D2WInspect.tiff"/>
  ...
</wo>
```

A child of `<wo>` naming the icon WebObjects Builder showed for the element in its component palette.
Used across the corpus (concentrated in DirectToWeb), every one a `.tiff`. It is **not part of the
declared grammar** — so those `.api` files are technically invalid against the DTD, yet shipped and
worked. (`legacy-api.dtd` deliberately leaves it out, to represent the declared grammar; this section
records what the files actually use.)

This is the cleanest example of declared-vs-practiced drift. The need (an element knows what icon
represents it in a palette) was real and met; the grammar just never caught up. Whether the successor
wants an equivalent is open — note `.apiext`'s richer preview arguably supersedes a static palette icon,
and "which HTML element does this render as" ([apiext-format#10](https://github.com/undur/apiext-format/issues/10))
is a more modern take on "what is this element, visually".

---

## Relics & quirks — the "understand the past" summary

1. **Simple class names** in `class` (Obj-C era, pre-packages — ambiguous now; see apiext-format#11).
2. **`<binding>` carries no structure** — name + flag attributes only. The empty slot `.apiext` fills.
3. **Inverted, implicit validation logic** — rules describe the *invalid* state, predicates implicitly
   AND-ed. Confusing; see apiext-format#9.
4. **DTD vs. reality drift** — `<image>` used but undeclared; predicates nested more loosely than the DTD
   permits (`<validation>` officially allows only `<and>`/`<count>`, yet bare predicates appear directly
   inside it); `<documentation>` and `<ungettable>` declared but dead.
5. **Inconsistent booleans** — `YES` / `NO` / `yes` / `no` / `true` / `false` all in the wild, untyped by
   the DTD.
6. **`defaults` is a closed, hardcoded value-source set** — the ancestor of `.apiext`'s dynamic value
   sets (apiext-format#4).
