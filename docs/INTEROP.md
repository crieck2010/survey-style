# Interop

How survey-style talks to the rest of the earthwatch-suite. The rule,
shared with survey-layout: **peers exchange plain data, never imports.**
survey-style never imports survey-viz, survey-layout, or reel-studio;
consumers never need to import survey-style's internals, only its
top-level API.

## survey-viz (renders styles)

survey-style writes exactly the `VizSpec` fields of the same names and
one `render_viz` kwarg:

| survey-style output | survey-viz consumer |
|---|---|
| `spec_dict["style"]` | `VizSpec.style` — `"reel-dark"` / `"light"` (`KNOWN_STYLES`) |
| `spec_dict["title"]` | `VizSpec.title` — frame header |
| `spec_dict["caption"]` | `VizSpec.caption` — footer caption (`None` = default footer) |
| `spec_dict["underlay"]` | `VizSpec.underlay` — basemap underlay toggle |
| `render_kwargs["cmap"]` | `render_viz(..., cmap=...)` — any registered matplotlib colormap |

The base-style tuple `KNOWN_BASE_STYLES` mirrors
`viz.spec.KNOWN_STYLES`; the test suite re-verifies the mirror against
the installed survey-viz so drift fails loudly. Colormap *names* are
validated by survey-viz against its matplotlib registry at render
time — survey-style stays stdlib-only and treats them as opaque
strings. Every built-in cmap is drawn from viz's curated
recommendations (`CURATED_CMAPS`); a test enforces that too.

Categorical renderers (storm tracks, earthquakes, streamgages) ignore
`cmap`; the style's title/footer/underlay still apply to them.

## survey-layout (platform geometry)

Orthogonal by design: survey-layout decides *where* things sit on the
platform canvas (regions, safe zones); survey-style decides *how* they
look (colors, titles, footers). Apply both to one spec dict — neither
knows about the other, and the order doesn't matter:

```python
spec, render_kw = style.apply_style(spec, "midnight-ocean")
canvas = layout.to_viz_canvas("tiktok")
viz.render_viz(data, spec, canvas=canvas, **render_kw)
```

## reel-studio (UI)

The Aesthetics step offers `style.list_styles()` as preset choices.
Picking one calls `apply_style` against the current spec and fills the
existing style / colormap / title / caption / underlay controls, which
the pipeline then consumes unchanged. If the `style` peer is missing,
reel-studio hides the preset picker and the manual controls keep
working — the same optional-peer pattern as survey-layout.

## Adding a variable

1. Add its colormap to the built-ins' `colormaps` tables (or rely on
   `default_cmap`).
2. Add a label to `VARIABLE_LABELS` for `{variable_label}` templates.
3. Optionally add a `SUGGESTIONS` entry so `suggest_style` finds it.
4. Unknown variables already work everywhere: prettified-slug label,
   `default_cmap`, `"reel-dark"` suggestion. Nothing crashes on a
   variable the style author never saw.

## Versioning across peers

- Package version: semver (`__version__`).
- Style *schema* version: `STYLE_SCHEMA_VERSION` (`"1.0"`), recorded on
  every `Style`. If fields are ever added/renamed, loaders can migrate
  or refuse based on this — user JSON styles keep working.
