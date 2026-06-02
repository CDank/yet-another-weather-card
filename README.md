# Yet Another Weather Card

[![hacs_badge](https://img.shields.io/badge/HACS-Custom-orange.svg)](https://github.com/hacs/integration)
[![GitHub Release](https://img.shields.io/github/release/CDank/yet-another-weather-card.svg)](https://github.com/CDank/yet-another-weather-card/releases)
[![License](https://img.shields.io/github/license/CDank/yet-another-weather-card.svg)](LICENSE)

Yes, the world needed another one. A custom weather card for Home Assistant with animated SVG icons, toggleable hourly/daily forecast, support for custom sensor entities, and location-based weather via GPS coordinates or a device tracker.

<img width="624" height="810" alt="Image" src="https://github.com/user-attachments/assets/b4d82580-3028-45e3-aadc-87abf6af6549" />

 

## Features

- **Visual editor** — full GUI configuration, no YAML required
- **Animated SVG icons** — rotating sun, drifting clouds, falling rain, tumbling snow, flickering lightning, shifting fog; optionally disabled
- **Live forecast** — uses the modern `weather/subscribe_forecast` WebSocket API (HA 2024+), no polling
- **Hourly ⇄ Daily toggle** — automatically appears when both forecast types are available
- **Today's high / low** — forecasted daily high (↑) and low (↓) shown below the current temperature in the header
- **Expandable daily view** — tap a day in the daily strip to open its hourly (or day/night) detail, when the integration provides it
- **Custom sensor entities** — override temperature, humidity, pressure, **and wind speed / direction** readings with your own sensors (e.g. an Ecowitt GW3000); wind direction is shown as a compass label (e.g. `W 12 km/h`)
- **Per-mode forecast sizing & layout** — separate item counts for the hourly and daily views, each laid out as a horizontal scroll strip or a wrapping grid
- **Temperature graph** — optional filled-area chart (`graph` / `both`) with adaptive label thinning so dense hourly forecasts stay readable
- **Optional fixed height** — keep the card at its tallest rendered height so switching views or opening a day doesn't shift the dashboard
- **Location-based weather** — fetch current conditions and forecast from [Open-Meteo](https://open-meteo.com/) (free, no API key) for any GPS coordinate; location can be a fixed lat/lon or tracked via a `device_tracker` / `person` entity
- **Multi-language** — `en`, `de`, `fr`, `hu`; auto-detects your Home Assistant UI language
- **Theme-aware** — uses Home Assistant CSS variables, looks correct in both light and dark mode
- **Reduced-motion friendly** — respects `prefers-reduced-motion: reduce`; can also be forced off via config

## Installation via HACS

### 1. Add this repository as a custom repository

1. In Home Assistant, open **HACS**
2. Click the three-dots menu in the top right → **Custom repositories**
3. Add:
   - **Repository**: `https://github.com/CDank/yet-another-weather-card`
   - **Type**: `Dashboard`
4. Click **Add**

### 2. Install

1. Search for **Yet Another Weather Card** in HACS
2. Click **Download**, then **Download** again to confirm
3. **Hard-refresh** your browser (Ctrl/Cmd + Shift + R)

HACS adds the resource automatically. If for some reason it doesn't, add it manually under **Settings → Dashboards → Resources**:

- URL: `/hacsfiles/yet-another-weather-card/yet-another-weather-card.js`
- Type: **JavaScript Module**

### 3. Add the card to your dashboard

```yaml
type: custom:yet-another-weather-card
entity: weather.your_weather_entity
```

## Configuration

### Data source

The card operates in one of two modes depending on what you configure:

| Mode | When | Data source |
|---|---|---|
| **Entity mode** | `entity` is set | HA `weather.*` entity via WebSocket — real-time updates |
| **Location mode** | No `entity`; `location_entity` or `latitude` + `longitude` set | [Open-Meteo](https://open-meteo.com/) — fetched on load and refreshed every 10 minutes or when coordinates change |

### Full option reference

| Option | Type | Default | Description |
|---|---|---|---|
| `entity` | string | — | A `weather.*` entity. Required unless location options are set. |
| `location_entity` | string | — | A `device_tracker.*` or `person.*` entity. Uses its `latitude` / `longitude` attributes as the forecast location. |
| `latitude` | number | — | Fixed latitude coordinate (used together with `longitude`). |
| `longitude` | number | — | Fixed longitude coordinate (used together with `latitude`). |
| `name` | string | friendly name | Title shown above the temperature. |
| `temperature_entity` | string | — | Custom sensor for current temperature (falls back to weather/Open-Meteo attribute). |
| `humidity_entity` | string | — | Custom sensor for humidity. |
| `pressure_entity` | string | — | Custom sensor for pressure. |
| `wind_speed_entity` | string | — | Custom sensor for wind speed (overrides the weather/Open-Meteo value). |
| `wind_bearing_entity` | string | — | Custom sensor for wind direction, value in degrees (0–360); rendered as a compass label. |
| `default_mode` | `hourly` \| `daily` | `hourly` | Which forecast tab is active initially. |
| `forecast_items` | number | `7` | Legacy fallback item count, used only when the per-mode options below are unset. |
| `forecast_items_hourly` | number | `7` | Number of hours shown in the hourly view (1–48). |
| `forecast_items_daily` | number | `7` | Number of days shown in the daily view (1–14). |
| `hourly_layout` | `scroll` \| `wrap` | `scroll` | Hourly strip layout — one horizontal scroll row, or a wrapping grid. |
| `daily_layout` | `scroll` \| `wrap` | `wrap` | Daily strip layout — one horizontal scroll row, or a wrapping grid. |
| `forecast_style` | `strip` \| `graph` \| `both` | `strip` | Forecast display — icon cells, temperature graph, or both. |
| `expandable_days` | boolean | `true` | In the daily view, tap a day to open its hourly (or day/night) detail. |
| `lock_height` | boolean | `false` | Keep the card at its tallest rendered height so switching views or opening a day doesn't shift the dashboard. |
| `language` | `en` \| `de` \| `fr` \| `hu` | HA locale | UI language for condition names and labels. |
| `show_current` | boolean | `true` | Show top block (temperature + icon + condition name). |
| `show_today_minmax` | boolean | `true` | Show today's forecasted high (↑) and low (↓) below the current temperature. Hidden automatically if the integration doesn't include today in the daily forecast. |
| `show_stats` | boolean | `true` | Show humidity / pressure / wind row. |
| `show_forecast` | boolean | `true` | Show forecast section. |
| `disable_animations` | boolean | `false` | Freeze all icon animations (icons still display, just static). |

### Location mode notes

- **`location_entity` takes priority over `latitude` / `longitude`** when both are set.
- Weather data is fetched from [Open-Meteo](https://open-meteo.com/) — free, no account or API key required.
- Temperature unit follows your Home Assistant unit system (°C or °F).
- `temperature_entity`, `humidity_entity`, and `pressure_entity` still work as overrides on top of Open-Meteo data.
- Coordinates are re-checked on every HA state update. If they change (e.g. a moving device tracker), a new fetch fires immediately.
- Data is refreshed at most once every 10 minutes when coordinates are unchanged.

### Forecast sizing, layout & detail notes

- `forecast_items_hourly` / `forecast_items_daily` take precedence over the legacy `forecast_items`. If only `forecast_items` is set, it applies to both views.
- `hourly_layout` / `daily_layout` control whether each strip is a single horizontal **scroll** row or a **wrap**ping grid.
- With `expandable_days: true`, days that have a more detailed forecast become tappable in the daily view. The detail view prefers the hourly forecast for that day and falls back to twice-daily (day/night) segments; it honours `forecast_style`, so `graph` / `both` draws an hourly graph for the selected day.
- In location mode the hourly forecast covers the next 48 hours, so `forecast_items_hourly` can go up to 48.

## Example configurations

**Minimal — entity mode**

```yaml
type: custom:yet-another-weather-card
entity: weather.home
```

**With custom sensors (incl. wind)**

```yaml
type: custom:yet-another-weather-card
entity: weather.met_no
name: Home
temperature_entity: sensor.outdoor_temperature
humidity_entity: sensor.outdoor_humidity
pressure_entity: sensor.outdoor_pressure
wind_speed_entity: sensor.outdoor_wind_speed
wind_bearing_entity: sensor.outdoor_wind_direction
default_mode: hourly
forecast_items_hourly: 12
forecast_items_daily: 7
```

**Per-mode layout, graph & expandable days**

```yaml
type: custom:yet-another-weather-card
entity: weather.home
forecast_style: both
hourly_layout: scroll
daily_layout: wrap
forecast_items_hourly: 24
forecast_items_daily: 7
expandable_days: true
lock_height: true
```

**Location mode — fixed coordinates**

```yaml
type: custom:yet-another-weather-card
latitude: 48.8566
longitude: 2.3522
name: Paris
forecast_style: both
```

**Location mode — device tracker (follows a person)**

```yaml
type: custom:yet-another-weather-card
location_entity: person.john
name: John's Weather
default_mode: daily
```

**Static icons, French language**

```yaml
type: custom:yet-another-weather-card
entity: weather.home
language: fr
disable_animations: true
```

## Supported weather conditions

`sunny`, `clear`, `clear-night`, `partlycloudy`, `cloudy`, `fog`, `rainy`, `pouring`, `snowy`, `snowy-rainy`, `lightning`, `lightning-rainy`, `windy`, `windy-variant`, `hail`, `exceptional`. Unknown conditions render a generic cloud.

## License

MIT
