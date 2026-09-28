# Style catalog

Six built-ins ship with the package. Every colormap below is in
survey-viz's curated set (`viz.render.CURATED_CMAPS`); survey-viz
re-validates each name against its matplotlib registry at render time.
Variables not listed under a style fall back to that style's
`default_cmap`.

## reel-dark — "Reel Dark" (default)

Base: `reel-dark` · default cmap: `turbo` · underlay: on ·
title `{title}` · footer: none · tags: `dark`, `default`

| Variable | Colormap |
|---|---|
| sst, t2m | turbo |
| wind, currents | viridis |
| ocean-color, chlorophyll | YlGnBu |
| tp | Blues |
| msl | RdBu_r |
| sea-ice | Blues |
| night-lights | inferno |
| topography | terrain |
| water-storage | BrBG |
| fires | hot |

## reel-light — "Reel Light"

Base: `light` · default cmap: `viridis` · underlay: on ·
title `{title}` · footer: none · tags: `light`, `default`

| Variable | Colormap |
|---|---|
| sst, t2m | YlOrRd |
| wind, currents | viridis |
| ocean-color, chlorophyll | YlGnBu |
| tp | Blues |
| msl | RdBu_r |
| sea-ice | Blues |
| night-lights | magma |
| topography | gist_earth |
| water-storage | BrBG |
| fires | YlOrRd |

## midnight-ocean — "Midnight Ocean"

Base: `reel-dark` · default cmap: `inferno` · underlay: on ·
title `{title}` · footer `{variable_label} · {region_label}` ·
tags: `dark`, `ocean`, `cinematic`

| Variable | Colormap |
|---|---|
| sst, t2m | inferno |
| currents | plasma |
| ocean-color, chlorophyll | viridis |
| wind | magma |
| night-lights | inferno |
| sea-ice | cool |
| tp | Blues |
| msl | RdBu_r |
| topography | terrain |
| water-storage | BrBG |
| fires | hot |

## field-notes — "Field Notes"

Base: `light` · default cmap: `terrain` · underlay: on ·
title `{title}` · footer `Field notes · {date}` ·
tags: `light`, `land`, `print`

| Variable | Colormap |
|---|---|
| topography | gist_earth |
| sst, t2m | YlOrRd |
| tp | Blues |
| msl | RdBu_r |
| ocean-color, chlorophyll | YlGnBu |
| wind | Greens |
| currents | ocean |
| sea-ice | Blues |
| water-storage | BrBG |
| fires | YlOrRd |
| night-lights | magma |

## storm-chaser — "Storm Chaser"

Base: `reel-dark` · default cmap: `turbo` · underlay: on ·
title `{title}` · footer `{variable_label} · {start} → {end}` ·
tags: `dark`, `atmosphere`, `hazards`

| Variable | Colormap |
|---|---|
| wind | turbo |
| msl | seismic |
| tp | Blues |
| fires | hot |
| sst, t2m | coolwarm |
| currents | viridis |
| ocean-color, chlorophyll | YlGnBu |
| sea-ice | Blues |
| night-lights | inferno |
| topography | terrain |
| water-storage | BrBG |

## creator — "Creator Brand Kit"

Base: `reel-dark` · default cmap: `turbo` · underlay: on ·
title `{title}` · footer `© {channel}` ·
tags: `dark`, `brand`

Same colormap table as `reel-dark`. The `{channel}` field must be
supplied when applying (`apply_style(spec, "creator",
channel="My Channel")`); without it you get an error naming the fix,
not a broken footer.

## Suggestions

`suggest_style(variable)` maps variables to the built-in that fits
best; anything unmapped (including future variables) suggests
`reel-dark`, the neutral default:

sst / currents / ocean-color / chlorophyll / night-lights → `midnight-ocean`;
wind / msl / tp / fires / storm-tracks → `storm-chaser`;
topography / water-storage → `field-notes`;
t2m / sea-ice / earthquakes / streamflow → `reel-dark`.

A suggestion is a starting point, not a rule — pass any style name you
like to `apply_style`.

## Writing your own

A style is one JSON object; every key below is optional except `name`
(defaults shown). Unknown keys are rejected with the file named.

```json
{
  "name": "arctic-dawn",
  "version": "1.0",
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

Template fields: `{title}` `{variable}` `{variable_label}` `{region}`
`{region_label}` `{start}` `{end}` `{date}` `{year}` `{channel}`.
`{channel}` is caller-supplied (`apply_style(..., channel="...")`).
Literal braces: `{{` and `}}`.
