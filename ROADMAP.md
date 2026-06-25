# `.apiext` — roadmap

Remaining work on the format. Each item has (or will have) a GitHub issue for the discussion and the
change. For what's already shipped see [`DONE.md`](./DONE.md).

The guiding stance: the format *describes* an element's binding contract; enforcement and presentation are
the consuming tool's concern. New vocabulary is added deliberately, not speculatively.

---

## Format vocabulary

Features not yet in the format.

| Item | Notes | Issue |
|---|---|---|
| Unknown-attribute policy | `unknownAttributes="forbidden\|allowed\|passthrough"` on `<wo>`; folds in / replaces the `passthrough` boolean. | [#1](https://github.com/undur/apiext-format/issues/1) |
| `settable` vs. `<push>` redundancy | `settable` duplicates `<push>` (without the type); deprecate it for `.apiext` authoring. | [#2](https://github.com/undur/apiext-format/issues/2) |
| Default values | `<default>` — the value a binding takes when unbound. Distinct from `defaults` (the autocomplete preset). | [#3](https://github.com/undur/apiext-format/issues/3) |
| Value sets | A binding's allowed values — literal enums *and* dynamic, tool-resolved sources (e.g. `wo:img filename` → existing resources). Generalizes WO's `defaults` presets. | [#4](https://github.com/undur/apiext-format/issues/4) |
| Deprecation | Mark a binding deprecated, with migration docs. | [#5](https://github.com/undur/apiext-format/issues/5) |
| Component reusability | Is a component usable as a tag inside other templates? Page-level components are one example of the non-reusable case. | [#7](https://github.com/undur/apiext-format/issues/7) |
| Use scope (access control) | Who may use an element/component — public / framework-internal / project-internal — like Java's access modifiers. Distinct from reusability: *who may use it* vs. *can it be a tag*. | [#8](https://github.com/undur/apiext-format/issues/8) |
| Binding-validation syntax | The `<validation>`/`<and>`/`<count>`/`<bound>`… predicate language, inherited verbatim from `.api` and never designed/documented for `.apiext`. Review, document, possibly redesign. | [#9](https://github.com/undur/apiext-format/issues/9) |
| Primary rendered HTML element | Declare the HTML element an element renders as (`WOImage` → `<img>`). Reference value + makes passthrough checkable (cf. #1). | [#10](https://github.com/undur/apiext-format/issues/10) |
| `<wo class>` simple-name relic | The root `class` attribute carries the simple class name (Obj-C-era, ambiguous with packages). Reconcile with "class references are FQN; the simple name is the element name". | [#11](https://github.com/undur/apiext-format/issues/11) |

## Current-format plumbing

Work on what the format already declares, not new vocabulary.

| Item | Notes | Issue |
|---|---|---|
| Tag library (shortcuts / aliases) | A consumer-owned `name → element` map (e.g. `if` → `WOConditional`), with framework defaults a project can override (`link` → a custom subclass). Not on the element — so not format vocabulary; an ng-objects implementation + a WO bridge. | [#6](https://github.com/undur/apiext-format/issues/6) |
| `.apiext` as single source of truth | Today `.api` and `.apiext` are parallel files. Endgame: `.apiext` canonical, `.api` regenerated or retired. | _TBD_ |

## Structural decisions (gating, not yet made)

Open design questions rather than implementable features:

| Decision | Issue |
|---|---|
| Where element metadata lives — per-file / project-override / framework-bundled, and precedence | _TBD_ |
| `.api` v1 compatibility — parallel / auto-migrate / drop at a major version | _TBD_ |
| What a runtime enforces vs. IDE-only warnings, and at what severity | _TBD_ |
| Where the shared parser/model library lives | _TBD_ |

---

## Ownership

The format is a shared contract. AjaxSlim (in wonder-slim) is the proving ground and reference corpus; the
larger vocabulary and the structural decisions are owned more by the runtime and IDE consumers
(ng-objects, Parsley/WOLips) than by any single framework.
