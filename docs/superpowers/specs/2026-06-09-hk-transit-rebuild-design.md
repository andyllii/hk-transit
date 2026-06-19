# HK Transit — Next-Gen Rebuild Design Spec

**Date:** 2026-06-09
**Status:** Approved
**Author:** Sisyphus

## Overview

Rebuild the Hong Kong real-time transit guide as a modern Next.js application with PWA + Capacitor cross-platform support. Same features as the current SPA, redesigned with a refined dark aesthetic, mobile-app-like UX, and a Vercel-native tech stack.

## Tech Stack

| Layer | Choice | Rationale |
|---|---|---|
| Framework | Next.js 16 App Router | SSR/SSG, RSC, Server Actions, Vercel-native |
| UI Library | React 19 | Latest, View Transitions, Server Components |
| Styling | Tailwind CSS v4 | Utility-first, perf, dark mode built-in |
| Animation | CSS View Transitions + Framer Motion | Native route anims + micro-interactions |
| Data Fetching | SWR | Dedup, caching, revalidation for real-time APIs |
| Icons | lucide-react | Consistent, tree-shakeable |
| Map | Leaflet (react-leaflet) | Free, no API key, OSM tiles |
| Native Wrapper | Capacitor | iOS/Android from same codebase |
| PWA | next-pwa or serwist | Service worker, manifest, offline shell |
| Hosting | Vercel | Zero-config, edge functions, preview deploys |
| Package Manager | pnpm | Fast, disk-efficient |

## Project Structure

```
hk-transit/
├── app/
│   ├── layout.tsx              # Root layout: meta, fonts, AppShell
│   ├── page.tsx                # Home: hero + mode cards + weather
│   ├── mtr/
│   │   ├── page.tsx            # MTR lines grid
│   │   ├── [line]/page.tsx     # Line detail + station list
│   │   └── arrival/
│   │       └── [station]/page.tsx  # Real-time arrival board
│   ├── bus/
│   │   ├── page.tsx            # Bus search + route list
│   │   └── [route]/page.tsx    # Route detail + stops + ETA
│   ├── minibus/
│   │   └── page.tsx            # GMB route search
│   ├── ferry/
│   │   └── page.tsx            # Ferry route list
│   ├── map/
│   │   └── page.tsx            # Leaflet nearby map
│   ├── traffic/
│   │   └── page.tsx            # Road incident notices
│   └── weather/
│       └── page.tsx            # Weather detail panel
├── components/
│   ├── layout/
│   │   ├── app-shell.tsx       # Orchestrates header + content + nav
│   │   ├── bottom-nav.tsx      # Tab bar (Home, MTR, Bus, More)
│   │   ├── header-strip.tsx    # Brand, clock, theme, weather mini
│   │   └── more-modal.tsx      # Overflow modes grid
│   ├── home/
│   │   ├── hero.tsx            # Title + tagline
│   │   ├── mode-cards.tsx      # Transport mode card grid
│   │   └── weather-card.tsx    # Live weather card
│   ├── mtr/
│   │   ├── line-chips.tsx      # MTR line color chips
│   │   ├── station-list.tsx    # Stations for a line
│   │   └── arrival-board.tsx   # Real-time ETA table
│   ├── bus/
│   │   ├── route-search.tsx    # Search by route number/stop
│   │   ├── stop-list.tsx       # Stops for a route
│   │   └── eta-display.tsx     # Live arrival times
│   ├── ferry/
│   │   └── ferry-route-card.tsx
│   ├── map/
│   │   └── transit-map.tsx     # Leaflet with cluster layer
│   ├── traffic/
│   │   └── notice-list.tsx     # Road incident cards
│   ├── disruption/
│   │   ├── alert-banner.tsx    # Top-of-app disruption banner
│   │   └── disruption-list.tsx # Full disruption feed
│   ├── planner/
│   │   ├── journey-form.tsx    # A→B search form
│   │   └── route-result.tsx    # Journey result with transfers
│   ├── weather/
│   │   ├── header-weather.tsx  # Mini strip in header
│   │   ├── detail-card.tsx     # Full home card
│   │   └── weather-panel.tsx   # Slide-up detail panel
│   └── shared/
│       ├── glass-card.tsx      # Reusable glass-morphism card
│       ├── loading.tsx         # Skeleton loader
│       ├── error-boundary.tsx  # Error fallback
│       ├── empty-state.tsx     # No data state
│       └── icon-button.tsx     # Touch-friendly icon btn
├── lib/
│   ├── api/
│   │   ├── mtr.ts              # MTR ETA API client
│   │   ├── bus.ts              # KMB / Citybus API client
│   │   ├── ferry.ts            # HKKF API client
│   │   ├── traffic.ts          # Traffic notice parser
│   │   ├── weather.ts          # HKO API client
│   │   └── disruption.ts       # MTR disruption feed client
│   ├── routing/
│   │   ├── graph.ts            # Transit graph builder
│   │   ├── raptor.ts           # RAPTOR routing algorithm
│   │   └── transfers.ts        # Walking transfer computation
│   ├── hooks/
│   │   ├── use-eta.ts          # SWR ETA hook with 30s refresh
│   │   ├── use-weather.ts      # SWR weather with 10min refresh
│   │   ├── use-traffic.ts      # SWR traffic with 5min refresh
│   │   ├── use-disruption.ts   # SWR disruption with 2min refresh
│   │   ├── use-geolocation.ts  # One-time location fetch
│   │   ├── use-theme.ts        # Dark/light/system theme toggle
│   │   ├── use-notifications.ts # Push/local notification hooks
│   │   └── use-planner.ts      # Journey planner hook
│   ├── types.ts                # Shared TypeScript interfaces
│   ├── constants.ts            # Colors, URLs, config values
│   └── utils.ts                # Formatters, helpers
├── data/                       # Static route/stop JSON files
├── public/
│   ├── icons/                  # PWA icons
│   ├── manifest.json           # Web app manifest
│   └── sw.js                   # Service worker
├── capacitor/                  # Capacitor native projects
├── next.config.ts
├── tailwind.config.ts
├── tsconfig.json
└── package.json
```

## Routing & Navigation

### Tab-Based Layout

The app uses a bottom tab bar (native-app paradigm) rather than a top nav. Four tabs:

| Tab | Route | Content |
|---|---|---|
| Home | `/` | Hero, mode cards, weather card |
| MTR | `/mtr` | MTR lines grid → line detail → station arrivals (3-level drill-down) |
| Bus | `/bus` | Search bar + route list → route detail with stops + ETA |
| More | modal overlay | Grid of overflow modes: Minibus, Ferry, Map, Traffic, Weather |

### Tab persistence

- Tab state is preserved (SWR cache keeps data)
- Switching tabs is instant — no re-fetch unless stale
- Each tab maintains its scroll position
- View Transitions animate tab switches (slide horizontally)

### Deep navigation (within a tab)

- MTR tab supports 3 levels: lines → line detail → station arrivals
- Back navigation uses Next.js `router.back()` with View Transition
- Each level has its own URL for deep linking and shareability
- URL params persist tab context: `/mtr?line=tung-chung&station=tung-chung`

## Visual Design

### Color System

**Surfaces (dark theme):**
- `--bg-base: #0a0a14` — deepest background
- `--bg-raised: #12121e` — cards, panels
- `--bg-overlay: #1a1a2e` — modals, dropdowns
- `--glass-bg: rgba(255,255,255,0.03)` — glass card surface
- `--glass-border: rgba(255,255,255,0.06)` — glass card border

**Mode accent colors:**
- MTR: `#22d3ee` (cyan)
- Bus: `#ef4444` (red)
- Minibus: `#22c55e` (green)
- Ferry: `#06b6d4` (teal)
- Map: `#a855f7` (violet)
- Traffic: `#f59e0b` (amber)
- Weather: `#22d3ee` (cyan)

**Ink:**
- `--ink-0: #f0f0f4` — primary text
- `--ink-1: #b0b0c0` — secondary text
- `--ink-2: #707080` — tertiary/labels
- `--ink-3: #505060` — disabled

**Light theme:** All surfaces invert. Same accent colors with adjusted contrast.

### Typography

- **Display:** `"Plus Jakarta Sans"` for headings (modern, geometric, good CJK support)
- **Data:** `"JetBrains Mono"` for times, counts, ETA (tabular numbers)
- **Body:** System font stack (San Francisco on iOS, Roboto on Android)

### Component Styles

**Glass Card** (`components/shared/glass-card.tsx`):
- Semi-transparent background with backdrop blur
- 1px border with subtle glow on active
- Rounded corners (12-16px)
- Optional top gradient accent bar
- On press: scale(0.98) + brighter glow

**Bottom Nav:**
- Fixed at bottom, safe-area aware
- 4 tabs with icon + label
- Active tab has accent color + indicator dot
- Haptic feedback on selection (navigator.vibrate(8))

**Skeleton Loading:**
- Shimmer animation matching card shapes
- Content pushes in below as data arrives
- Never shows raw empty state during fetch

## Data Flow

### Real-Time Transit Data

All transit ETAs come from external HK government/open APIs:

```
Client Page → useEta(stopId, route) → SWR → fetch → API Gateway → External API
                                               ↓
                                         SWR Cache (keyed by stopId+route)
                                               ↓
                                         30s revalidate
```

- `useEta` returns `{ data, error, isLoading, isValidating }`
- Data is never stale for more than 30s
- SWR dedup ensures multiple components fetching same stop share one request
- On error: show last known data (stale-while-revalidate) + subtle error indicator
- On offline: show cached data if available

### Weather Data

```
useWeather() → SWR → fetchWeatherData() → HKO rhrread + flw (parallel)
                         ↓
                   STATE_WEATHER (SWR cache)
                         ↓
                   10min revalidate
```

- Uses `Promise.allSettled` for parallel rhrread + flw fetches
- Returns weather header data + forecast separately
- Error state hides weather UI components gracefully

### Static Data

Route lists, stop lists, ferry schedules → imported as JSON modules at build time. No runtime fetch needed.

- `data/mtr_lines.csv` → parsed build-time, typed as `MtrLine[]`
- `data/kmb_stops.json` → used for KMB stop search
- `data/gmb_routes_*.json` → region-split GMB data
- `data/citybus_routes.json` → Citybus route list

## Features (Expanded Scope)

### Push Notifications (ETA Alerts)

**Capability:** Users set alerts for specific routes/stops — "Notify me when the next bus to Central is within 5 minutes."

**Architecture (Progressive):**

| Tier | Approach | Platforms |
|---|---|---|
| Foreground | Capacitor Local Notifications — app polls ETAs and triggers local notification when threshold met | iOS, Android |
| Background | Service Worker Periodic Sync (PWA) + Vercel Cron Job or Edge Function to check ETAs and send Web Push | Web PWA |
| Full | Firebase Cloud Messaging (Capacitor) for reliable background pushes across all platforms | All |

**v1 scope:** Foreground local notifications + Web Push for PWA. When the app is open (or in background via service worker), it monitors subscribed ETAs and triggers notifications when conditions match.

**UI:**
- Bell icon on route/stop cards to subscribe
- "Manage Alerts" page listing active subscriptions
- Each alert: route + stop + direction + time threshold

**Data:**
- Alerts stored in `localStorage` (no backend required for v1)
- Service worker listens for push events, checks cached ETA data

### Offline Route Planning

**Capability:** Browse routes, stops, and schedules without network connection. Journey planning (A → B) using bundled transit graph.

**Offline data bundle:**
- All MTR lines + stations + interchange data
- All bus routes + stops (KMB, Citybus)
- All GMB minibus routes
- All ferry routes + schedules
- Walking transfer distances between nearby stops

**Implementation:**
- Transit graph built at build time from static data files
- Serialized to a compact JSON/web-friendly format
- Bundled with the app as a static import
- Client-side routing algorithm (simplified RAPTOR or connection-scan)
- Web Worker for compute-heavy routing without blocking UI

**v1 scope:** 
- Full offline browsing of all routes, stops, and schedules
- Simple journey planner (MTR + bus transfers) using bundled data
- Real-time ETA data still requires network (falls back to schedule times offline)

### Real-Time MTR Disruption Feed

**Capability:** Live MTR service disruption notices displayed across the app.

**Data source:** MTR's real-time disruption feed (available via MTR data portal).

**Display:**
- **Alert banner** at top of app when disruption active (color-coded by severity)
- **Disruption list** on home page showing all active incidents
- **Line-specific banners** on MTR page showing disruptions on that line
- **Auto-refresh** every 2 minutes

**UX:**
- Non-blocking banner (tappable for details)
- Icons indicating disruption type (delay, closed section, reduced service)
- Estimated resolution time when available
- Affected stations highlighted on line diagram

**v1 scope:** Fetch and display MTR disruption feed. Banner alerts + dedicated disruption list page.

## Capacitor Integration

### Strategy: URL-Based WebView

Rather than bundling the built app into native binaries, the Capacitor WebView points to the live Vercel URL. This means:

- No app store submission needed for most updates
- Same codebase, same deployment flow
- Capacitor plugins available for native features (haptics, geolocation, BLE)

### Build Flow

```bash
# Web deployment
npm run build          # next build
vercel deploy          # → production URL

# Native build (ci/release only)
npm run build          # next build + export
npx cap copy           # Copy to native project
npx cap sync           # Sync plugins
npx cap open ios       # Open Xcode for archiving
```

### Native Features

| Feature | Capacitor Plugin |
|---|---|
| Haptics | `@capacitor/haptics` — tap feedback |
| Geolocation | `@capacitor/geolocation` — user location |
| Status Bar | `@capacitor/status-bar` — dark content overlay |
| Splash Screen | `@capacitor/splash-screen` — branded launch |
| Local Notifications | `@capacitor/local-notifications` — ETA alerts |
| Push Notifications | Firebase Cloud Messaging via `@capacitor/push-notifications` — background pushes |

## Performance Targets

| Metric | Target |
|---|---|
| Lighthouse Performance | ≥95 |
| First Contentful Paint | ≤1.5s |
| Largest Contentful Paint | ≤2.5s |
| Time to Interactive | ≤2.0s |
| Lighthouse Accessibility | ≥95 |
| Bundle size (initial) | ≤120KB JS gzipped |
| Route prefetch | On hover (Next.js default) |

## Accessibility Requirements

- All interactive elements have visible focus rings (keyboard nav)
- Touch targets minimum 44×44px (Apple HIG / Material Design)
- `aria-live="polite"` on ETA displays that auto-update
- Screen reader announcements for: route changes, ETA updates, errors
- `prefers-reduced-motion` disables all animations, shows static transitions
- `prefers-contrast: more` increases border contrast, removes transparency
- Color usage never relies solely on color — always paired with text/labels
- Chinese + English bilingual UI (all labels in both languages)

## Migration Strategy

The rebuild is a **new project** in a new folder. Existing SPA is untouched.

1. Scaffold Next.js project with Tailwind + Capacitor
2. Copy static data files
3. Build shared components (GlassCard, Loading, ErrorBoundary, etc.)
4. Build layout (AppShell, BottomNav, Header)
5. Build Home page (hero, mode cards, weather)
6. Build MTR page (lines → stations → arrivals)
7. Build Bus page (search → routes → ETA)
8. Build Minibus, Ferry pages
9. Build Map page (Leaflet)
10. Build Traffic page
11. Build Weather page
12. Build disruption feed (MTR incidents banner + list)
13. Build journey planner (transit graph + routing algorithm)
14. Build notification system (local + push)
15. PWA setup (manifest, service worker, push)
16. Capacitor setup (native projects, splash screen)
17. Polish, animations, accessibility audit

## Out of Scope (v1)

- User accounts / favorites
- BLE proximity (adventure app feature — not transit)
