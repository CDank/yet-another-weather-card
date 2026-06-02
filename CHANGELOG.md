# Changelog

## 1.7.0 — 2026-06-02

Customized fork of upstream v1.6.0 ([CDank/yet-another-weather-card](https://github.com/CDank/yet-another-weather-card)).

### Added
- **Wind sensor overrides** — `wind_speed_entity` and `wind_bearing_entity` let custom sensors override the weather/Open-Meteo wind values (in both entity and location mode). Wind direction is rendered as a compass label (e.g. `W 12 km/h`).
- **Per-mode forecast item counts** — `forecast_items_hourly` (1–48) and `forecast_items_daily` (1–14) replace the single `forecast_items`, which is kept as a legacy fallback.
- **Per-mode layout** — `hourly_layout` and `daily_layout` (`scroll` / `wrap`) control whether each strip is a horizontal scroll row or a wrapping grid (defaults: hourly scrolls, daily wraps).
- **Expandable daily view** — `expandable_days` (default on); tap a day in the daily strip to open its hourly (or twice-daily day/night) detail. The detail view honours `forecast_style`, so `graph` / `both` draws an hourly graph for the selected day.
- **Optional fixed card height** — `lock_height` keeps the card at its tallest rendered height so switching views or opening a day doesn't shift the dashboard.
- **Today's high / low in the header** — `show_today_minmax` (default `true`) displays the forecasted daily high (↑) and low (↓) below the current temperature. Hidden automatically if the integration doesn't include today in its daily data.

### Changed
- Open-Meteo hourly forecast extended from 24 h to 48 h, so `forecast_items_hourly` can use the full range.
- Graph labels are now adaptively thinned (max ~7) so times and temperatures don't squish on dense hourly forecasts.
- Restored the upstream explanatory code comments and the bundled-Lit license notice that had been stripped from the working copy; added inline rationale comments for the fork's changes.

### Fixed
- **Strip icons** — fixed clipping, jitter, and vertical/horizontal centering. Icon size is pinned in CSS so Home Assistant's global styles can't collapse the SVG, and each strip icon is re-centred once per motif by reframing its viewBox.
- **Icon animation positioning** — animations now use `transform-box: fill-box` with an own-box `transform-origin`, so rotating/drifting elements spin in place instead of orbiting the icon centre (root cause of clouds and other motifs being mispositioned).
- Guarded `_resolveValue` against `null` weather-attribute lookups in location mode.

---

## 1.6.0 — 2026-05-29

### Added
- **Location-based weather** — new alternative to specifying a `weather.*` entity. When no `entity` is set, the card fetches current conditions and forecast from [Open-Meteo](https://open-meteo.com/) (free, no API key) using GPS coordinates. Two ways to provide the location:
  - `location_entity` — a `device_tracker.*` or `person.*` entity; the card reads its `latitude` / `longitude` attributes and re-fetches whenever they change (e.g. a moving phone).
  - `latitude` + `longitude` — fixed numeric coordinates entered directly in the visual editor or YAML.
- **`disable_animations`** — new boolean option that freezes all icon CSS animations while keeping the full animated SVG icons visible. Useful for low-power displays or personal preference. Mirrors the existing `prefers-reduced-motion` behaviour.
- **French language (`fr`)** — full translation for all condition names and UI labels.

### Changed
- `entity` is now optional when location options are configured.
- Temperature unit in location mode follows the Home Assistant unit system (°C / °F).
- `temperature_entity`, `humidity_entity`, `pressure_entity` still work as sensor overrides on top of Open-Meteo data in location mode.
- Hourly graph in location mode now shows a filled temperature band (current-hour temp → day's forecasted low), matching the entity-mode graph behaviour.

### Fixed
- Concurrent Open-Meteo fetch requests (triggered by rapid HA state updates after a config save) are now serialised with an `_omFetching` guard, preventing current conditions and forecast from being drawn from different responses.
- All three Open-Meteo payloads (`_omCurrent`, `_omHourly`, `_omDaily`) are built in local variables before any reactive assignment, so Lit always renders them in the same cycle — no partial-update flicker.
- Saving new coordinates now immediately resets and re-fetches instead of waiting for the 10-minute stale threshold.

---

## 1.5.0 — 2026-05-20

### Fixed
- **Works in Fully Kiosk Browser and other restricted WebViews now.** Previously the card imported Lit from `cdn.jsdelivr.net`, which Fully Kiosk's WebView blocks or fails to load on many devices, leading to a "Custom element doesn't exist" error even after refresh/restart. The card now has Lit inlined directly — fully self-contained, no external runtime dependencies. The single JS file (~57 KB) is fetched once from your HA host and that's it.

### Changed
- Inlined Lit 3 (~17 KB minified) is targeted at ES2019 (Chrome 71+, Android WebView 71+), broadening compatibility with older Fire tablets and kiosk devices.

## 1.4.2 — 2026-05-20

### Changed
- Graph labels actually visible now. The root cause was that SVG text scales with the viewBox, so the previous "13 px" font-size was being rendered at ~9-10 px once the SVG was scaled to fit the card. Fixed by (a) shrinking the viewBox width (600 → 400) so it more closely matches the rendered card width, and (b) bumping the SVG font-sizes to 16/14/13 so they actually read at strip-equivalent sizes. Graph height grew to 150 to make room.
- Data point circles slightly larger (r=3 → r=3.5) to remain visually balanced with the bigger labels.

## 1.4.1 — 2026-05-20

### Changed
- Graph label sizes now match the strip cells: high temperatures 13 px (was 11), low temperatures and time/day labels 11 px (was 10). Padding around the chart grew slightly to keep the bigger labels from clipping or colliding with the curves.

## 1.4.0 — 2026-05-20

### Added
- `forecast_style` config option with three values:
  - `strip` (default, existing behavior) — cells with icons and temperatures
  - `graph` — filled-area temperature chart with smooth curves between high (solid amber line) and low (dashed blue line). Each data point labeled with the high value above and low value below; time/day labels along the bottom
  - `both` — graph rendered above the cell strip
- When the forecast data has no `templow` (typically hourly forecasts), the graph degrades gracefully to a single high line with no area fill.

## 1.3.0 — 2026-05-20

### Added
- `language` option supporting `en`, `de`, `hu`. Defaults to your Home Assistant UI language; falls back to English. All condition names and UI labels (Hourly/Daily/Now/Today/Humidity/Pressure/Wind) are translated. Weekday and time formatting also follows the chosen language.

### Fixed
- Condition text no longer renders as one-word jumbles like "Partlycloudy" or "Snowyrainy" — now properly displayed as "Partly cloudy", "Snowy, rainy", etc.

## 1.2.0 — 2026-05-20

### Changed
- Icons enlarged ~50%: header icon 88 → 132 px, forecast icons 36 → 54 px.

## 1.1.0 — 2026-05-20

### Fixed
- **Icons now render correctly.** SVG-child content is now tagged with lit-html's `svg` template literal instead of `html`, which the previous version incorrectly used. The HTML parser was discarding `<circle>`, `<rect>`, `<g>` and similar SVG elements because they weren't being placed in the SVG namespace, leaving icons invisible.
- Lit is now loaded from `cdn.jsdelivr.net/gh/lit/dist@3/all/lit-all.min.js` (Lit 3, the same major version Home Assistant uses internally) instead of `unpkg.com/lit-element@2.5.1`. This is more reliable and includes the `svg` export needed for the fix above.

### Added
- Visual editor (`getConfigElement`) — full GUI configuration via `ha-form`. Entity pickers, dropdowns, toggles, all themed correctly.
- Card picker preview thumbnail (`preview: true`).

## 1.0.0
Initial release.
