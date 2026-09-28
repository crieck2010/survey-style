# Architecture

## Module layout

```
src/style/
├── __init__.py     # public API surface (the only import consumers need)
├── styles.py       # Style dataclass, validation, registry
├── presets.py      # built-in styles (data) + JSON user-style loading
├── templates.py    # template field set, checking, rendering
├── labels.py       # variable/region human labels
└── apply.py        # apply_style / render_kwargs_for / suggest_style
tests/
└── test_style.py   # 37 tests, stdlib-only (no network, no render)
docs/
├── STYLES.md       # built-in catalog + authoring guide
├── INTEROP.md      # peer contracts
└── ARCHITECTURE.md # this file
```

## Design principles

**Styles are data, not code.** A `Style` is a frozen dataclass of
strings, a dict, and a bool. Built-ins are module-level data in
`presets.py`; user styles are JSON. No behavior hides inside a style,
so styles are trivially serializable, diffable, and shareable.

**One error type.** `StyleError` (a `ValueError`) covers unknown names,
invalid definitions, bad templates, and missing template values. Every
message names the field and the fix; unknown style names get
did-you-mean hints.

**Validate at the boundary.** The `Style` constructor validates, so an
invalid style cannot exist in-process. `validate_style` is the
non-raising twin for UIs and loaders that want the full problem list.
JSON loading validates per file and names the file — nothing is
half-registered.

**Fixed template vocabulary.** Templates draw from exactly ten fields.
The set is fixed so a style renders identically in every consumer and
a typo'd field fails at definition time, not mid-render.

**No viz import.** The package is stdlib-only. The contract with
survey-viz is duck-typed: spec dicts in, `VizSpec`-named fields out.
Two tests read the installed survey-viz to verify the peers agree on
style names and the curated colormap set — the package itself never
imports it. This keeps survey-style usable in contexts where viz isn't
installed (config tools, style editors) and avoids version pinning.

**Registry, not singletons-with-state.** Built-ins register at import;
`register_style` / `load_styles_from_dir` add user styles.
`apply_style` never mutates the registry or its input — overrides are
merged into a fresh `Style` (re-validated) for that call only.

**Scaling.** The registry pattern means the catalog grows without
code changes: more built-ins are more data entries; user styles are
files in a directory. Tags exist so UIs can group/filter a large
catalog later. The schema version on every style is the migration hook
if fields ever change.

## What's deliberately out of scope

- Font files, sizes, and pixel geometry — survey-viz owns rendering.
- Image-based style transfer or palette extraction from a reference
  image — deterministic named styles only (ownership: everything the
  creator ships must be reproducible and locally owned).
- Per-frame animation of style parameters — motion stays in
  survey-animate's domain.
