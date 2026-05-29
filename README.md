# Yet Another Weather Card

[![hacs_badge](https://img.shields.io/badge/HACS-Custom-orange.svg)](https://github.com/hacs/integration)
[![GitHub Release](https://img.shields.io/github/release/YOUR_GITHUB_USERNAME/yet-another-weather-card.svg)](https://github.com/YOUR_GITHUB_USERNAME/yet-another-weather-card/releases)
[![License](https://img.shields.io/github/license/YOUR_GITHUB_USERNAME/yet-another-weather-card.svg)](LICENSE)

Yes, the world needed another one. A custom weather card for Home Assistant with animated SVG icons, toggleable hourly/daily forecast, support for custom sensor entities, and location-based weather via GPS coordinates or a device tracker.

<img width="521" height="528" alt="chrome_Bbgutx0MyQ" src="https://github.com/user-attachments/assets/45689732-3bb1-45f2-b567-8542cbb633fe" />

 

## Features

- **Visual editor** — full GUI configuration, no YAML required
- **Animated SVG icons** — rotating sun, drifting clouds, falling rain, tumbling snow, flickering lightning, shifting fog; optionally disabled
- **Live forecast** — uses the modern `weather/subscribe_forecast` WebSocket API (HA 2024+), no polling
- **Hourly ⇄ Daily toggle** — automatically appears when both forecast types are available
- **Custom sensor entities** — override temperature, humidity, and pressure readings with your own sensors (e.g. an Ecowitt GW3000)
- **Location-based weather** — fetch current conditions and forecast from [Open-Meteo](https://open-meteo.com/) (free, no API key) for any GPS coordinate; location can be a fixed lat/lon or tracked via a `device_tracker` / `person` entity
- **Multi-language** — `en`, `de`, `fr`, `hu`; auto-detects your Home Assistant UI language
- **Theme-aware** — uses Home Assistant CSS variables, looks correct in both light and dark mode
- **Reduced-motion friendly** — respects `prefers-reduced-motion: reduce`; can also be forced off via config

## Installation via HACS

### 1. Add this repository as a custom repository

1. In Home Assistant, open **HACS**
2. Click the three-dots menu in the top right → **Custom repositories**
3. Add:
   - **Repository**: `https://github.com/balazsskorka/yet-another-weather-card`
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
| `default_mode` | `hourly` \| `daily` | `hourly` | Which forecast tab is active initially. |
| `forecast_items` | number | `7` | Number of forecast cells to show (1–24). |
| `forecast_style` | `strip` \| `graph` \| `both` | `strip` | Forecast display — icon cells, temperature graph, or both. |
| `language` | `en` \| `de` \| `fr` \| `hu` | HA locale | UI language for condition names and labels. |
| `show_current` | boolean | `true` | Show top block (temperature + icon + condition name). |
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

## Example configurations

**Minimal — entity mode**

```yaml
type: custom:yet-another-weather-card
entity: weather.home
```

**With custom sensors**

```yaml
type: custom:yet-another-weather-card
entity: weather.met_no
name: Home
temperature_entity: sensor.outdoor_temperature
humidity_entity: sensor.outdoor_humidity
pressure_entity: sensor.outdoor_pressure
default_mode: hourly
forecast_items: 7
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
