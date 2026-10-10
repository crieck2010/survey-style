> [!IMPORTANT]
> **Consolidated into [EarthWatch-Suite](https://github.com/crieck2010/earthwatch-suite)**  
> This engine has been consolidated into the unified [EarthWatch-Suite monorepo](https://github.com/crieck2010/earthwatch-suite). Active development, bug fixes, releases, and unified testing now live in earthwatch-suite. This repository is preserved as an archive.

# survey-style

Reusable visual styles/templates for the earthwatch-suite render stack.

A **style** is a named, versioned bundle of the aesthetic choices
survey-viz supports — base style (`"reel-dark"` / `"light"`),
per-variable colormaps, title/footer templates, and the basemap-underlay
toggle. Apply one named look across many reels instead of re-picking
widgets every time.

```python
import style

style.list_styles()
# ['creator', 'field-notes', 'midnight-ocean', 'reel-dark', 'reel-light', 'storm-chaser']

spec_dict, render_kwargs = style.apply_style(
    viz_spec, "midnight-ocean")          # spec dict + {"cmap": "inferno"}
```

## Installation

```bash
pip install git+https://github.com/crieck2010/survey-style.git
```

Requires Python 3.10+. The core is **stdlib-only** — no numpy, no
matplotlib, no survey-viz import. (The test suite reads the installed
survey-viz only to verify the two peers agree on style names and the
curated colormap set; the package itself never imports it.)

## Quickstart

```python
import style

# 1. Pick a style (or get a suggestion for your variable)
name = style.suggest_style("sst")          # "midnight-ocean"

# 2. Apply it to a VizSpec / spec dict / to_dict() object
spec_dict, render_kwargs = style.apply_style(
    spec, name, channel="My Channel")      # channel only needed for {channel} templates

# 3. Hand the pieces to survey-viz exactly as before
viz.render_viz(data, spec_dict, **render_kwargs)
```

`apply_style` writes the `VizSpec` fields `style`, `title`, `caption`,
and `underlay`, and returns `{"cmap": ...}` for `render_viz`'s `cmap`
parameter. The input spec is never mutated. Per-call tweaks without
touching the registry:

```python
spec_dict, kw = style.apply_style(
    spec, "reel-dark",
    overrides={"footer_template": "© {channel}", "underlay": False},
    channel="My Channel")
```

## Built-in styles

| Name | Base | For | Footer |
|---|---|---|---|
| `reel-dark` | reel-dark | Everything (default) | none |
| `reel-light` | light | Daytime / print-friendly | none |
| `midnight-ocean` | reel-dark | Ocean variables | `{variable_label} · {region_label}` |
| `field-notes` | light | Terrain / land | `Field notes · {date}` |
| `storm-chaser` | reel-dark | Atmosphere / hazards | `{variable_label} · {start} → {end}` |
| `creator` | reel-dark | Channel brand kit | `© {channel}` (needs `channel=...`) |

Full per-variable colormap tables: [docs/STYLES.md](docs/STYLES.md).

## Your own styles

Styles are data. Drop JSON files in a directory and load them:

```json
{
  "name": "arctic-dawn",
  "label": "Arctic Dawn",
  "description": "Pale style for sea-ice reels.",
  "base_style": "light",
  "colormaps": {"sea-ice": "Blues", "sst": "cool"},
  "default_cmap": "viridis",
  "title_template": "{title}",
  "footer_template": "{variable_label} · {region_label}",
  "underlay": true,
  "tags": ["light", "polar"]
}
```

```python
style.load_styles_from_dir("~/my-styles")   # *.json, validated on load
```

Templates may use `{title} {variable} {variable_label} {region}
{region_label} {start} {end} {date} {year} {channel}` — any other field
fails at definition time, not mid-render.

## API

| Function | Does |
|---|---|
| `list_styles()` | sorted names of built-ins + loaded user styles |
| `get_style(name)` | the `Style`; unknown names raise with a did-you-mean hint |
| `apply_style(spec, name, *, overrides=None, channel=None)` | `(spec_dict, render_kwargs)` |
| `render_kwargs_for(style, variable)` | `{"cmap": ...}` for a variable |
| `suggest_style(variable)` | suggested built-in for a variable (`"reel-dark"` fallback) |
| `validate_style(style)` | list of problems (empty = valid) |
| `register_style(style, replace=False)` | add a programmatic style |
| `load_styles_from_dir(path)` | load `*.json` user styles |
| `render_template(t, ctx)` / `variable_label(v)` / `region_label(r)` | template + label helpers |

`StyleError` (a `ValueError`) is the single error type for unknown
names, invalid definitions, bad templates, and missing template values.

## Interop

- **survey-viz** — consumes the spec dict (`style`/`title`/`caption`/`underlay`
  are `VizSpec` fields) and `render_viz(..., cmap=...)`. survey-style never
  imports survey-viz; the contract is duck-typed dicts.
- **survey-layout** — orthogonal: styles color *what* is drawn, layouts
  decide *where* on the platform canvas. Apply both to one spec.
- **reel-studio** — the Aesthetics step offers the style catalog as
  presets that fill the existing style/colormap/title/caption/underlay
  controls.

See [docs/INTEROP.md](docs/INTEROP.md) and
[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Versioning

Semantic versioning. The style *schema* is versioned separately
(`STYLE_SCHEMA_VERSION = "1.0"`, recorded on every `Style`) so loaders
can migrate or refuse cleanly when fields change.

## License

MIT — see [LICENSE](LICENSE).
