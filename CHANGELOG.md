# Changelog

All notable changes to this project. Format follows
[Keep a Changelog](https://keepachangelog.com/en/1.0.0/); versioning
follows [Semantic Versioning](https://semver.org/).

## [0.1.0] - 2026-09-28

Initial release.

### Added
- `Style` dataclass: name, version, label, description, `base_style`
  (`"reel-dark"`/`"light"`, mirroring `viz.spec.KNOWN_STYLES`),
  per-variable `colormaps`, `default_cmap`, `title_template`,
  `footer_template`, `underlay` toggle, `tags`.
- Six built-in presets: `reel-dark`, `reel-light`, `midnight-ocean`,
  `field-notes`, `storm-chaser`, `creator` (channel brand kit).
- `apply_style(spec, name, *, overrides=None, channel=None)` returning
  `(spec_dict, render_kwargs)`; accepts a VizSpec, a `to_dict()`
  object, or a plain dict; never mutates the input.
- Fixed template field set (`{title} {variable} {variable_label}
  {region} {region_label} {start} {end} {date} {year} {channel}`)
  with definition-time validation and helpful missing-value errors.
- `suggest_style(variable)` with per-variable suggestions and a
  `"reel-dark"` fallback.
- User styles: `load_styles_from_dir(path)` (`*.json`, validated on
  load) and `register_style(style, replace=False)`.
- `validate_style(style)` non-raising validator; `StyleError` as the
  single error type; did-you-mean hints on unknown style names.
- `STYLE_SCHEMA_VERSION = "1.0"` recorded on every style.
- 37 tests, stdlib-only core (the package never imports survey-viz;
  two tests read the installed viz to verify peer agreement).
- Docs: README, docs/STYLES.md (full catalog), docs/INTEROP.md,
  docs/ARCHITECTURE.md.
