# astro-ephemeris-model — Solar System (2D heliocentric) minimal viewer

A minimal Jupyter notebook that plots heliocentric planet positions in the ecliptic plane for a chosen UTC date/time.

This is meant to be simple and hackable:
- **2D** only (uses ecliptic plane; plots **x/y**)
- **Heliocentric** (positions are computed as *(body − Sun)*)
- Default timestamp is **now (UTC)**
- Controlled by a small YAML file instead of editing the notebook

## What the “model” does

Given a UTC timestamp, the notebook uses **Skyfield** with the **JPL DE421** ephemeris to compute each selected body’s heliocentric ecliptic position:

- Load ephemeris: `de421.bsp`
- Choose bodies (Mercury…Neptune; outer planets via barycenters)
- Compute heliocentric position: `(body - sun).at(t).ecliptic_position().au`
- Plot the **(x, y)** components in **AU**

Optional: it can draw a “circular orbit” for each planet as a circle with radius equal to the planet’s **current heliocentric distance** (purely visual; not a physical orbit fit).

## Files

- `solar_system_viewer.ipynb` — the viewer notebook
- `viewer_config.yaml` — runtime configuration (date/time, planets, plot limits)
- `de421.bsp` — JPL ephemeris kernel used by Skyfield

Note: `.gitignore` ignores `*.bsp`, so if you clone this repo elsewhere you may need to supply `de421.bsp` again.

## Requirements

- Python 3.x
- Packages:
  - `numpy`
  - `matplotlib`
  - `skyfield`
  - `pyyaml`

The notebook’s first code cell uses `%pip install ...` to install these into the active kernel environment.

## Configuration (`viewer_config.yaml`)

Keys:

- `target_date`: `"YYYY-MM-DD"` or `""` (empty) to use current time
- `target_time`: `"HH:MM"` or `""` (empty) to use current time
- `planets_to_show`: list of planet names (strings)
- `lim_au`: plot limits in AU (sets x/y to `[-lim_au, +lim_au]`)
- `show_circular_orbits`: `true/false` (draws the distance circles)

### Minor bodies (dwarf planets + clouds)

The notebook can optionally plot:
- **Named dwarf planets** (select individually; includes Pluto)
- **Clouds** of many minor bodies (main belt, Jupiter Trojans, Centaurs, Kuiper Belt)

This uses **Minor Planet Center (MPC)** orbital elements through Skyfield.

Notes:
- On the first run with minor bodies enabled, Skyfield may download the MPCORB catalog (tens of MB) and cache it.
- The notebook then builds a **small local excerpt** (default `mpcorb.sample.DAT`) so subsequent runs remain fast.
- Requires `pandas` (already included in the notebook install cell).
- A few MPCORB rows can contain elements that are numerically problematic for orbit reconstruction; the notebook skips those rows instead of crashing.

Keys:
- `dwarf_planets_to_show`: list of dwarf planet names to plot (strings)
- `show_main_belt`: `true/false`
- `show_jupiter_trojans`: `true/false`
- `show_centaurs`: `true/false`
- `show_kuiper_belt`: `true/false`
- `minor_bodies_total_sample`: how many MPCORB lines to sample into the local excerpt
- `main_belt_max`, `trojans_max`, `centaurs_max`, `kuiper_belt_max`: caps for how many dots to plot per group
- `minor_bodies_seed`: makes sampling deterministic/reproducible
- `minor_bodies_size`, `minor_bodies_alpha`: dot styling
- `minor_bodies_cache_path`: excerpt filename (relative to the notebook folder by default)
- `mpcorb_reload`: force re-download + rebuild the excerpt

Kuiper Belt definition used by this notebook:
- **Kuiper Belt cloud** is defined as minor bodies with semimajor axis **strictly** between **30 and 50 AU**: $30 < a < 50$ (where $a$ is MPCORB `semimajor_axis_au`).

Centaurs definition used by this notebook:
- **Centaurs cloud** is defined as minor bodies with semimajor axis and perihelion strictly inside the giant-planet region: $5.5 < a < 30$ and $5.5 < q < 30$ AU, where $q = a(1-e)$ and $e$ is MPCORB `eccentricity`.

Note: these are orbital-element filters from MPCORB, not full dynamical classifications.

Migration note:
- Old keys `show_tno` / `tno_max` are accepted as aliases, but are deprecated.

Allowed names:

- Planets: `Mercury`, `Venus`, `Earth`, `Mars`, `Jupiter`, `Saturn`, `Uranus`, `Neptune`
- Dwarf planets: `Pluto`, `Ceres`, `Eris`, `Haumea`, `Makemake`

Example:

```yaml
target_date: "2026-04-05"
target_time: "12:00"

planets_to_show:
  - Mercury
  - Venus
  - Earth
  - Mars
  - Jupiter

lim_au: 8
show_circular_orbits: true
```

## Run

1. Ensure `de421.bsp` is present next to the notebook (same folder).
2. Open `solar_system_viewer.ipynb` in VS Code (or Jupyter).
3. Edit `viewer_config.yaml` as needed.
4. Run the notebook cells top-to-bottom.

## Notes / limitations

- The plot is **2D** (ecliptic plane) and does not show inclination/out-of-plane motion.
- Outer planets are queried using **barycenters** (e.g., `"jupiter barycenter"`) which is standard for many ephemerides.
- DE421 has a limited time span (roughly 1900–2050). Dates outside that may fail.
