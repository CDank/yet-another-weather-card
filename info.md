## Yet Another Weather Card

Yes, the world needed another one. A custom weather card with animated SVG icons, toggleable hourly/daily forecast, and support for custom temperature/humidity/pressure sensors.

**Features**
- Animated SVG icons (sun, clouds, rain, snow, lightning, fog, wind, hail)
- Hourly ⇄ Daily forecast toggle (live via WebSocket)
- Custom sensor entities override weather entity attributes
- Theme-aware (light & dark mode)
- Respects `prefers-reduced-motion`

**Minimal config**

```yaml
type: custom:yet-another-weather-card
entity: weather.your_weather_entity
```

See the README for full configuration options.
