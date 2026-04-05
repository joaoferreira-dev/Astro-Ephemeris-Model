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
- Choose bodies (Mercury…Pluto; outer planets via barycenters)
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

Allowed names:

- `Mercury`, `Venus`, `Earth`, `Mars`, `Jupiter`, `Saturn`, `Uranus`, `Neptune`, `Pluto`

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
