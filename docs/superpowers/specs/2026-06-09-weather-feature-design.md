# Weather Feature — HK Transit App

**Date**: 2026-06-09  
**Status**: Approved for implementation

## Overview

Add real-time Hong Kong weather information to the hk-transit app using the Hong Kong Observatory (HKO) Open Data API. Weather data is displayed at three levels: a compact header strip (always visible), a home page card, and a slide-up detail panel.

## Data Sources

All endpoints are from HKO Open Data API (`https://data.weather.gov.hk/weatherAPI/opendata/weather.php`). No API key required.

| Endpoint | Data | Refresh interval |
|---|---|---|
| `dataType=rhrread&lang=tc` | Current temp, humidity, rainfall, UV index, weather warnings | 10 min |
| `dataType=flw&lang=tc` | Local weather forecast (3-day overview) | 60 min |

### rhrread Response Fields Used

- `temperature.data[].value` — current temp per station (use HKO station)
- `humidity.data[].value` — current humidity %
- `rainfall.data[].max` / `rainfall.data[].msg` — rainfall info
- `uvindex.data[].value` / `uvindex.data[].desc` — UV index
- `rainstorm` / `tcInfo` — active weather warnings
- `icon` — current weather icon type (53 = overcast, 60 = rain, etc.)

### flw Response Fields Used

- `forecastDesc` — general forecast text (e.g. "Mainly cloudy with showers")
- `forecastIcon` — forecast icon type
- `outlook` — 3-day outlook text

## Architecture

All data fetching and rendering lives in `index.html` (single-file app). No new dependencies.

### State (`STATE_WEATHER`)

```js
const STATE_WEATHER = {
  current: null,      // { temp, humidity, icon, uv, rainfall, warnings }
  forecast: null,     // { desc, icon, outlook }
  lastFetch: 0,       // timestamp
  interval: null,     // refresh interval ID
  loading: false,
  error: null
};
```

### Functions

| Function | Purpose |
|---|---|
| `fetchWeatherData()` | Fetch rhrread + flw in parallel, update state |
| `renderHeaderWeather()` | Populate header weather strip |
| `renderHomeWeather()` | Populate home pane weather card |
| `renderWeatherPanel()` | Build slide-up panel HTML |
| `openWeatherPanel()` | Show slide-up panel (tapped from header) |
| `closeWeatherPanel()` | Dismiss slide-up panel |

## UI Components

### 1. Header Weather Strip

```
[Brand] [Clock] [🌡️ 28° 🌧] [Theme toggle]
```

- Placed between the clock and theme toggle button
- Shows temperature (°C) + condition icon (emoji or inline SVG)
- Font: `JetBrains Mono` (matches clock styling)
- Color: `var(--neon-cyan)` for temperature value
- Tappable → opens slide-up detail panel
- If fetch fails, strip hides gracefully (no fallback toast spam)

### 2. Home Page Weather Card

Inserted in the home pane hero area, below the subtitle and above the quick mode tiles.

```
┌──────────────────────────────┐
│ 天氣 · WEATHER               │
│                               │
│  🌤️  28°                      │
│  體感 30° · 濕度 73%          │
│  UV 6 (高) · 今日最高 31°     │
│                               │
│  ⚠️ 黃色暴雨警告信號生效       │
│                               │
│  📋 大致多雲，有幾陣驟雨。    │
│  未來兩日持續不穩定。          │
└──────────────────────────────┘
```

- Styled as a glass card matching the app aesthetic
- Active warnings get amber/red coloring (`--neon-amber` / `--neon-red`)
- Sections: current conditions, warnings (if any), forecast summary

### 3. Slide-Up Detail Panel

Reuses the `map-sheet` CSS pattern (bottom sheet with handle, blur backdrop).

- **Current conditions**: temperature, feels-like, humidity, rainfall, UV
- **Weather warnings**: listed with severity color
- **Forecast**: 2-3 day outlook text
- Close: tap handle area or background overlay

## Error Handling

- Network failures: silently retry at next interval, no user-facing error
- Stale data: `STATE_WEATHER.current` preserved until fresh data arrives
- First load spinner: show loading state until both rhrread + flw resolve
- CORS: HKO API supports direct browser fetch (no proxy needed)

## Refresh Strategy

- Initial fetch: immediately on page load (in parallel with transit data)
- Interval: every 10 minutes (`setInterval`)
- Manual refresh: pull-to-refresh gesture on the slide-up panel (optional stretch goal)
- Tab visibility: pause refresh when tab is hidden (`document.hidden`), resume on visibility

## Non-Goals

- No geolocation-based weather (HK-wide data only)
- No animated weather backgrounds
- No 9-day forecast (3-day outlook sufficient)
- No separate weather nav tab

## Design Self-Review

✅ No placeholders or TODOs  
✅ No internal contradictions  
✅ Scope is focused on a single implementable unit  
✅ Requirements unambiguous
