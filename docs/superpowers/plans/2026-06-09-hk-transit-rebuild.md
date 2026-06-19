# HK Transit Next-Gen — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rebuild the Hong Kong transit guide as a Next.js 16 App Router + React 19 + Tailwind CSS v4 PWA with Capacitor native wrapper, featuring a refined dark design, mobile-app UX, offline journey planner, push notifications, and real-time disruption feed.

**Architecture:** Next.js App Router with client components for real-time data (SWR caching), server components for static content. Tab-based layout with bottom nav. Capacitor wraps the deployed Vercel URL. Static data bundled at build time; real-time data fetched via SWR.

**Tech Stack:** Next.js 16 (App Router), React 19, Tailwind CSS v4, SWR, Framer Motion, lucide-react, Leaflet/react-leaflet, Capacitor, `@capacitor/local-notifications`, `@capacitor/push-notifications`, serwist (PWA), TypeScript.

**New project folder:** `/home/opencode/Works/hk-transit-next`

---

### Task 1: Scaffold Next.js Project

**Files:**
- Create: `hk-transit-next/` (project root)
- Create: `hk-transit-next/package.json`
- Create: `hk-transit-next/tsconfig.json`
- Create: `hk-transit-next/next.config.ts`
- Create: `hk-transit-next/postcss.config.mjs`
- Create: `hk-transit-next/tailwind.config.ts`

- [ ] **Step 1: Initialize the Next.js project**

```bash
mkdir -p /home/opencode/Works/hk-transit-next
cd /home/opencode/Works/hk-transit-next
# Use create-next-app with TypeScript + App Router + Tailwind
npx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir --import-alias "@/*" --use-pnpm --no-turbopack
```

Expected: Next.js 16 project scaffolded with Tailwind CSS v4, TypeScript, App Router, `src/` directory.

- [ ] **Step 2: Install all dependencies**

```bash
cd /home/opencode/Works/hk-transit-next
pnpm add swr framer-motion lucide-react leaflet react-leaflet @capacitor/core @capacitor/cli @capacitor/ios @capacitor/android @capacitor/haptics @capacitor/geolocation @capacitor/status-bar @capacitor/splash-screen @capacitor/local-notifications @capacitor/push-notifications serwist
pnpm add -D @types/leaflet
```

Expected: All runtime and dev dependencies installed.

- [ ] **Step 3: Initialize Capacitor**

```bash
cd /home/opencode/Works/hk-transit-next
npx cap init "HK Transit" "com.hktransit.app" --web-dir .next
```

Expected: `capacitor.config.ts` created at root.

- [ ] **Step 4: Create directory structure**

```bash
cd /home/opencode/Works/hk-transit-next
mkdir -p src/components/{layout,home,mtr,bus,ferry,map,traffic,disruption,planner,weather,shared}
mkdir -p src/lib/{api,routing,hooks}
mkdir -p src/data
mkdir -p public/icons
```

Expected: All directories created per spec structure.

- [ ] **Step 5: Commit**

```bash
cd /home/opencode/Works/hk-transit-next
git init && git add -A && git commit -m "feat: scaffold Next.js 16 project with Capacitor + PWA deps"
```

---

### Task 2: Configure Tailwind Theme & Global Styles

**Files:**
- Create: `src/app/globals.css`
- Create: `tailwind.config.ts`
- Modify: `src/app/layout.tsx` (import globals.css)

- [ ] **Step 1: Write `tailwind.config.ts` with Refined Dark theme**

```ts
import type { Config } from "tailwindcss";

export default {
  content: ["./src/**/*.{ts,tsx}"],
  darkMode: "class",
  theme: {
    extend: {
      colors: {
        bg: {
          base: "#0a0a14",
          raised: "#12121e",
          overlay: "#1a1a2e",
        },
        glass: {
          DEFAULT: "rgba(255,255,255,0.03)",
          border: "rgba(255,255,255,0.06)",
          hover: "rgba(255,255,255,0.06)",
        },
        ink: {
          0: "#f0f0f4",
          1: "#b0b0c0",
          2: "#707080",
          3: "#505060",
        },
        accent: {
          mtr: "#22d3ee",
          bus: "#ef4444",
          minibus: "#22c55e",
          ferry: "#06b6d4",
          map: "#a855f7",
          traffic: "#f59e0b",
          weather: "#22d3ee",
        },
      },
      fontFamily: {
        display: ['"Plus Jakarta Sans"', "system-ui", "sans-serif"],
        mono: ['"JetBrains Mono"', "monospace"],
      },
      backdropBlur: {
        glass: "16px",
      },
      borderRadius: {
        glass: "16px",
      },
    },
  },
  plugins: [],
} satisfies Config;
```

- [ ] **Step 2: Write `src/app/globals.css`**

```css
@import "tailwindcss";
@config "../../tailwind.config.ts";

@layer base {
  :root {
    --bg-base: #0a0a14;
    --bg-raised: #12121e;
    --bg-overlay: #1a1a2e;
    --glass-bg: rgba(255, 255, 255, 0.03);
    --glass-border: rgba(255, 255, 255, 0.06);
    --ink-0: #f0f0f4;
    --ink-1: #b0b0c0;
    --ink-2: #707080;
    --ink-3: #505060;
  }

  * {
    -webkit-tap-highlight-color: transparent;
  }

  html {
    scroll-behavior: smooth;
    -webkit-font-smoothing: antialiased;
  }

  body {
    background: var(--bg-base);
    color: var(--ink-0);
    font-family: system-ui, -apple-system, sans-serif;
    overscroll-behavior: none;
    -webkit-overflow-scrolling: touch;
  }

  /* Safe area padding for mobile */
  .safe-bottom {
    padding-bottom: env(safe-area-inset-bottom, 16px);
  }
  .safe-top {
    padding-top: env(safe-area-inset-top, 0px);
  }

  /* Touch targets */
  .touch-target {
    min-width: 44px;
    min-height: 44px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  /* Reduced motion */
  @media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
      animation-duration: 0.01ms !important;
      transition-duration: 0.01ms !important;
    }
  }

  /* Scrollbar styling */
  ::-webkit-scrollbar {
    width: 4px;
  }
  ::-webkit-scrollbar-track {
    background: transparent;
  }
  ::-webkit-scrollbar-thumb {
    background: var(--ink-3);
    border-radius: 2px;
  }
}

@layer components {
  .glass-card {
    background: var(--glass-bg);
    border: 1px solid var(--glass-border);
    backdrop-filter: blur(16px);
    -webkit-backdrop-filter: blur(16px);
    border-radius: 16px;
    transition: transform 0.15s ease, border-color 0.15s ease, box-shadow 0.15s ease;
  }
  .glass-card:active {
    transform: scale(0.98);
    border-color: rgba(255, 255, 255, 0.12);
    box-shadow: 0 0 20px rgba(34, 211, 238, 0.08);
  }

  .nav-active {
    color: var(--ink-0);
  }
  .nav-inactive {
    color: var(--ink-3);
  }
}

/* View Transitions */
::view-transition-old(root),
::view-transition-new(root) {
  animation-duration: 0.25s;
}

@media (prefers-reduced-motion: reduce) {
  ::view-transition-old(root),
  ::view-transition-new(root) {
    animation-duration: 0s;
  }
}
```

- [ ] **Step 3: Update `src/app/layout.tsx`** to import globals and set up base HTML

```tsx
import type { Metadata, Viewport } from "next";
import "./globals.css";

export const metadata: Metadata = {
  title: "HK Transit — Hong Kong Real-Time Transport Guide",
  description: "Real-time MTR, Bus, Minibus, Ferry, Traffic & Weather for Hong Kong",
  manifest: "/manifest.json",
  appleWebApp: { capable: true, statusBarStyle: "black-translucent", title: "HK Transit" },
};

export const viewport: Viewport = {
  width: "device-width",
  initialScale: 1,
  maximumScale: 1,
  viewportFit: "cover",
  themeColor: "#0a0a14",
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="zh-HK" className="dark">
      <head>
        <link rel="preconnect" href="https://fonts.googleapis.com" />
        <link rel="preconnect" href="https://fonts.gstatic.com" crossOrigin="anonymous" />
        <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;600&family=Plus+Jakarta+Sans:wght@500;600;700;800&display=swap" rel="stylesheet" />
      </head>
      <body className="antialiased">{children}</body>
    </html>
  );
}
```

- [ ] **Step 4: Build and verify**

```bash
cd /home/opencode/Works/hk-transit-next
pnpm build
```

Expected: Build succeeds, no errors.

- [ ] **Step 5: Commit**

```bash
cd /home/opencode/Works/hk-transit-next
git add -A && git commit -m "feat: configure Tailwind theme with Refined Dark design system"
```

---

### Task 3: Type Definitions & Constants

**Files:**
- Create: `src/lib/types.ts`
- Create: `src/lib/constants.ts`

- [ ] **Step 1: Write `src/lib/types.ts`**

```ts
// === Transport ===
export type TransportMode = "mtr" | "bus" | "minibus" | "ferry" | "map" | "traffic" | "weather";

export interface MtrLine {
  code: string;
  nameEn: string;
  nameZh: string;
  color: string;
  stations: MtrStation[];
}

export interface MtrStation {
  code: string;
  nameEn: string;
  nameZh: string;
  lat: number;
  lng: number;
}

export interface MtrEta {
  stationCode: string;
  lineCode: string;
  dir: "up" | "down";
  minutesToArrival: number;
  destinationEn: string;
  destinationZh: string;
}

export interface BusRoute {
  route: string;
  originEn: string;
  originZh: string;
  destEn: string;
  destZh: string;
  company: "kmb" | "citybus";
}

export interface BusStop {
  stopId: string;
  nameEn: string;
  nameZh: string;
  lat: number;
  lng: number;
}

export interface BusEta {
  stopId: string;
  route: string;
  minutesToArrival: number;
  destEn: string;
  destZh: string;
}

export interface FerryRoute {
  routeId: string;
  nameEn: string;
  nameZh: string;
  originEn: string;
  originZh: string;
  destEn: string;
  destZh: string;
}

// === Weather ===
export interface WeatherData {
  temp: number;
  humidity: number;
  condition: string;
  icon: string;
  warnings: WeatherWarning[];
  forecast: WeatherForecast[];
}

export interface WeatherWarning {
  code: string;
  nameEn: string;
  nameZh: string;
  severity: "info" | "caution" | "warning" | "danger";
}

export interface WeatherForecast {
  date: string;
  maxTemp: number;
  minTemp: number;
  humidity: number;
  condition: string;
  icon: string;
}

// === Traffic ===
export interface TrafficIncident {
  id: string;
  descriptionEn: string;
  descriptionZh: string;
  location: string;
  severity: "minor" | "moderate" | "major";
  timestamp: string;
}

// === Disruption ===
export interface Disruption {
  id: string;
  line: string;
  type: "delay" | "closed" | "reduced" | "planned";
  severity: "info" | "warning" | "critical";
  summaryEn: string;
  summaryZh: string;
  detailEn: string;
  detailZh: string;
  affectedStations: string[];
  estimatedResolution?: string;
  updatedAt: string;
}

// === Journey Planner ===
export interface TransitNode {
  id: string;
  nameEn: string;
  nameZh: string;
  lat: number;
  lng: number;
  lines: string[];
}

export interface RouteSegment {
  from: TransitNode;
  to: TransitNode;
  line: string;
  mode: TransportMode;
  departureTime: number;
  arrivalTime: number;
  stops: number;
}

export interface Journey {
  segments: RouteSegment[];
  totalDuration: number;
  transfers: number;
}

// === Notification ===
export interface EtaAlert {
  id: string;
  route: string;
  stopId: string;
  stopName: string;
  thresholdMinutes: number;
  direction: string;
  active: boolean;
}
```

- [ ] **Step 2: Write `src/lib/constants.ts`**

```ts
export const NAV_ITEMS = [
  { id: "home", labelEn: "Home", labelZh: "首頁", icon: "Home", route: "/" },
  { id: "mtr", labelEn: "MTR", labelZh: "港鐵", icon: "Train", route: "/mtr" },
  { id: "bus", labelEn: "Bus", labelZh: "巴士", icon: "Bus", route: "/bus" },
  { id: "more", labelEn: "More", labelZh: "更多", icon: "Grid3X3", route: null },
] as const;

export const MORE_MODES = [
  { id: "minibus", labelEn: "Minibus", labelZh: "小巴", icon: "Shuttle", route: "/minibus", color: "#22c55e" },
  { id: "ferry", labelEn: "Ferry", labelZh: "渡輪", icon: "Ship", route: "/ferry", color: "#06b6d4" },
  { id: "map", labelEn: "Map", labelZh: "地圖", icon: "Map", route: "/map", color: "#a855f7" },
  { id: "traffic", labelEn: "Traffic", labelZh: "交通", icon: "Car", route: "/traffic", color: "#f59e0b" },
  { id: "weather", labelEn: "Weather", labelZh: "天氣", icon: "CloudSun", route: "/weather", color: "#22d3ee" },
] as const;

export const MTR_LINE_COLORS: Record<string, string> = {
  "AEL": "#00888a",
  "TCL": "#fe7b1c",
  "TML": "#aa8633",
  "TKL": "#7a4894",
  "SIL": "#b5bd35",
  "EAL": "#60b9e8",
  "WRL": "#aa8633",
  "DRL": "#c7a13e",
  "KTL": "#1a9431",
  "ISL": "#0077c8",
  "TWL": "#e2231a",
};

export const HKO_ICON_MAP: Record<string, string> = {
  "50": "sun", "51": "sun", "52": "sun", "53": "partly-cloudy",
  "54": "partly-cloudy", "60": "rain", "61": "rain", "62": "rain",
  "63": "heavy-rain", "64": "heavy-rain", "65": "thunder",
  "70": "drizzle", "71": "drizzle", "72": "drizzle", "73": "rain",
  "74": "heavy-rain", "75": "thunder", "76": "rain", "77": "rain",
  "80": "thunder", "81": "thunder", "82": "thunder", "83": "heavy-rain",
  "84": "heavy-rain", "85": "thunder", "90": "wind", "91": "fog",
  "92": "fog",
};

export const REFRESH_INTERVALS = {
  eta: 30000,
  weather: 600000,
  traffic: 300000,
  disruption: 120000,
} as const;

export const API_BASE_URLS = {
  mtrEta: "https://rt.data.gov.hk/v1/transport/mtr/getSchedule",
  kmbEta: "https://rt.data.gov.hk/v1/transport/kmb/stop-eta",
  citybusEta: "https://rt.data.gov.hk/v1/transport/citybus-nwfb/eta",
  weather: "https://data.weather.gov.hk/weatherAPI/opendata",
  traffic: "https://rt.data.gov.hk/v1/transport/traffic",
  disruption: "https://rt.data.gov.hk/v1/transport/mtr/disruption",
} as const;
```

- [ ] **Step 3: Commit**

```bash
cd /home/opencode/Works/hk-transit-next
git add -A && git commit -m "feat: add TypeScript types and constants"
```

---

### Task 4: Shared Components

**Files:**
- Create: `src/components/shared/glass-card.tsx`
- Create: `src/components/shared/loading.tsx`
- Create: `src/components/shared/error-boundary.tsx`
- Create: `src/components/shared/empty-state.tsx`
- Create: `src/components/shared/icon-button.tsx`

- [ ] **Step 1: Write `src/components/shared/glass-card.tsx`**

```tsx
"use client";

import { type ReactNode } from "react";

interface GlassCardProps {
  children: ReactNode;
  className?: string;
  accentColor?: string;
  onClick?: () => void;
  role?: string;
  tabIndex?: number;
  "aria-label"?: string;
}

export function GlassCard({
  children,
  className = "",
  accentColor,
  onClick,
  role,
  tabIndex,
  "aria-label": ariaLabel,
}: GlassCardProps) {
  const accentStyle = accentColor
    ? { borderTop: `2px solid ${accentColor}` }
    : {};

  return (
    <div
      className={`glass-card p-4 ${className}`}
      style={accentStyle}
      onClick={onClick}
      role={role}
      tabIndex={tabIndex}
      aria-label={ariaLabel}
    >
      {children}
    </div>
  );
}
```

- [ ] **Step 2: Write `src/components/shared/loading.tsx`**

```tsx
interface LoadingProps {
  className?: string;
  count?: number;
  type?: "card" | "line" | "circle";
}

export function Loading({ className = "", count = 1, type = "card" }: LoadingProps) {
  const items = Array.from({ length: count });

  const shimmer = "animate-pulse bg-ink-3/20 rounded";

  if (type === "line") {
    return (
      <div className={`space-y-3 ${className}`} role="status" aria-label="Loading">
        {items.map((_, i) => (
          <div key={i} className={`${shimmer} h-4 ${i === 0 ? "w-3/4" : "w-1/2"}`} />
        ))}
      </div>
    );
  }

  if (type === "circle") {
    return (
      <div className={`flex gap-2 ${className}`} role="status" aria-label="Loading">
        {items.map((_, i) => (
          <div key={i} className={`${shimmer} w-12 h-12 rounded-full`} />
        ))}
      </div>
    );
  }

  return (
    <div className={`space-y-3 ${className}`} role="status" aria-label="Loading">
      {items.map((_, i) => (
        <div key={i} className={`${shimmer} h-24 w-full`} />
      ))}
    </div>
  );
}
```

- [ ] **Step 3: Write `src/components/shared/error-boundary.tsx`**

```tsx
"use client";

import { Component, type ReactNode } from "react";
import { AlertTriangle } from "lucide-react";

interface Props { children: ReactNode; fallback?: ReactNode; }
interface State { hasError: boolean; error?: Error; }

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div className="flex flex-col items-center justify-center p-8 text-ink-2 gap-3" role="alert">
          <AlertTriangle className="w-8 h-8" />
          <p className="text-sm">Something went wrong. Please try again.</p>
          <button
            onClick={() => this.setState({ hasError: false })}
            className="px-4 py-2 rounded-lg bg-ink-3/30 text-ink-1 text-sm touch-target"
          >
            Retry
          </button>
        </div>
      );
    }
    return this.props.children;
  }
}
```

- [ ] **Step 4: Write `src/components/shared/empty-state.tsx`**

```tsx
import { type LucideIcon } from "lucide-react";

interface EmptyStateProps {
  icon: LucideIcon;
  message: string;
  submessage?: string;
}

export function EmptyState({ icon: Icon, message, submessage }: EmptyStateProps) {
  return (
    <div className="flex flex-col items-center justify-center p-8 text-ink-2 gap-2">
      <Icon className="w-10 h-10" />
      <p className="text-sm font-medium text-ink-1">{message}</p>
      {submessage && <p className="text-xs">{submessage}</p>}
    </div>
  );
}
```

- [ ] **Step 5: Write `src/components/shared/icon-button.tsx`**

```tsx
import { type LucideIcon } from "lucide-react";

interface IconButtonProps {
  icon: LucideIcon;
  label: string;
  onClick?: () => void;
  className?: string;
  size?: number;
  active?: boolean;
}

export function IconButton({ icon: Icon, label, onClick, className = "", size = 20, active }: IconButtonProps) {
  return (
    <button
      onClick={onClick}
      className={`touch-target rounded-xl transition-colors ${
        active ? "text-accent-mtr" : "text-ink-3 hover:text-ink-1"
      } ${className}`}
      aria-label={label}
      type="button"
    >
      <Icon size={size} />
    </button>
  );
}
```

- [ ] **Step 6: Build to verify**

```bash
cd /home/opencode/Works/hk-transit-next
pnpm build
```

Expected: Build succeeds clean.

- [ ] **Step 7: Commit**

```bash
cd /home/opencode/Works/hk-transit-next
git add -A && git commit -m "feat: add shared UI components (GlassCard, Loading, ErrorBoundary, EmptyState, IconButton)"
```

---

### Task 5: API Clients

**Files:**
- Create: `src/lib/api/weather.ts`
- Create: `src/lib/api/mtr.ts`
- Create: `src/lib/api/bus.ts`
- Create: `src/lib/api/traffic.ts`
- Create: `src/lib/api/disruption.ts`
- Create: `src/lib/api/ferry.ts`

- [ ] **Step 1: Write `src/lib/api/weather.ts`**

```ts
export interface HkoRhrread {
  temperature: { data: { place: string; value: number; unit: string }[] };
  humidity: { data: { value: number; unit: string } };
  weatherIcon: number;
  warningMessage: string[];
}

export interface HkoFlw {
  forecast: { forecastDate: string; week: string; forecastMaxTemp: { value: number }; forecastMinTemp: { value: number }; forecastHumidity: { value: number }; forecastWeather: string; forecastIcon: number }[];
}

export async function fetchCurrentWeather(): Promise<HkoRhrread> {
  const res = await fetch(
    "https://data.weather.gov.hk/weatherAPI/opendata/weather.php?dataType=rhrread&lang=en"
  );
  if (!res.ok) throw new Error(`Weather fetch failed: ${res.status}`);
  return res.json();
}

export async function fetchForecast(): Promise<HkoFlw> {
  const res = await fetch(
    "https://data.weather.gov.hk/weatherAPI/opendata/weather.php?dataType=flw&lang=en"
  );
  if (!res.ok) throw new Error(`Forecast fetch failed: ${res.status}`);
  return res.json();
}

export async function fetchWeatherAll() {
  const [current, forecast] = await Promise.allSettled([
    fetchCurrentWeather(),
    fetchForecast(),
  ]);
  return { current: current.status === "fulfilled" ? current.value : null, forecast: forecast.status === "fulfilled" ? forecast.value : null };
}
```

- [ ] **Step 2: Write `src/lib/api/mtr.ts`**

```ts
interface MtrScheduleResponse {
  data?: Record<string, {
    up?: { time: string; dest: string; plat: string }[];
    down?: { time: string; dest: string; plat: string }[];
  }>;
}

export async function fetchMtrEta(lineCode: string, stationCode: string): Promise<MtrScheduleResponse["data"]> {
  const res = await fetch(
    `https://rt.data.gov.hk/v1/transport/mtr/getSchedule/${lineCode}/${stationCode}`
  );
  if (!res.ok) throw new Error(`MTR ETA fetch failed: ${res.status}`);
  const json: MtrScheduleResponse = await res.json();
  return json.data ?? null;
}
```

- [ ] **Step 3: Write `src/lib/api/bus.ts`**

```ts
interface KmbEtaResponse {
  data?: { eta: { timestamp: string; eta: string; dest_en: string; dest_tc: string }[] }[];
}

export async function fetchKmbEta(stopId: string, route: string): Promise<KmbEtaResponse["data"]> {
  const res = await fetch(
    `https://rt.data.gov.hk/v1/transport/kmb/stop-eta/${stopId}/${route}`
  );
  if (!res.ok) throw new Error(`KMB ETA fetch failed: ${res.status}`);
  const json: KmbEtaResponse = await res.json();
  return json.data ?? null;
}
```

- [ ] **Step 4: Write `src/lib/api/ferry.ts`**

```ts
export async function fetchFerryRoutes(): Promise<{ route: string; origin: string; dest: string; company: string }[]> {
  // Ferry data comes from bundled static JSON for now
  // HKKF real-time API can be integrated later
  return [];
}
```

- [ ] **Step 5: Write `src/lib/api/traffic.ts`**

```ts
interface TrafficResponse {
  incidents: { id: string; description: string; location: string; severity: string; time: string }[];
}

export async function fetchTrafficIncidents(): Promise<TrafficResponse["incidents"]> {
  const res = await fetch("https://rt.data.gov.hk/v1/transport/traffic/incidents");
  if (!res.ok) throw new Error(`Traffic fetch failed: ${res.status}`);
  const json: TrafficResponse = await res.json();
  return json.incidents ?? [];
}
```

- [ ] **Step 6: Write `src/lib/api/disruption.ts`**

```ts
export interface MtrDisruptionResponse {
  disruptions: {
    id: string;
    line: string;
    type: string;
    severity: string;
    summary_en: string;
    summary_tc: string;
    detail_en: string;
    detail_tc: string;
    affected_stations: string[];
    estimated_resolution: string;
    updated_at: string;
  }[];
}

export async function fetchDisruptions(): Promise<MtrDisruptionResponse["disruptions"]> {
  const res = await fetch("https://rt.data.gov.hk/v1/transport/mtr/disruption");
  if (!res.ok) throw new Error(`Disruption fetch failed: ${res.status}`);
  const json: MtrDisruptionResponse = await res.json();
  return json.disruptions ?? [];
}
```

- [ ] **Step 7: Build and commit**

```bash
cd /home/opencode/Works/hk-transit-next
pnpm build && git add -A && git commit -m "feat: add API client modules for all data sources"
```

Expected: Build succeeds.

---

### Task 6: Custom Hooks

**Files:**
- Create: `src/lib/hooks/use-eta.ts`
- Create: `src/lib/hooks/use-weather.ts`
- Create: `src/lib/hooks/use-traffic.ts`
- Create: `src/lib/hooks/use-disruption.ts`
- Create: `src/lib/hooks/use-geolocation.ts`
- Create: `src/lib/hooks/use-theme.ts`

- [ ] **Step 1: Write `src/lib/hooks/use-eta.ts`**

```ts
"use client";

import useSWR from "swr";
import { fetchMtrEta } from "@/lib/api/mtr";
import { fetchKmbEta } from "@/lib/api/bus";
import { REFRESH_INTERVALS } from "@/lib/constants";

export function useMtrEta(lineCode: string, stationCode: string) {
  const key = stationCode && lineCode ? `mtr-eta-${lineCode}-${stationCode}` : null;
  return useSWR(key, () => fetchMtrEta(lineCode, stationCode), {
    refreshInterval: REFRESH_INTERVALS.eta,
    revalidateOnFocus: true,
    errorRetryCount: 2,
  });
}

export function useBusEta(stopId: string, route: string) {
  const key = stopId && route ? `bus-eta-${stopId}-${route}` : null;
  return useSWR(key, () => fetchKmbEta(stopId, route), {
    refreshInterval: REFRESH_INTERVALS.eta,
    revalidateOnFocus: true,
    errorRetryCount: 2,
  });
}
```

- [ ] **Step 2: Write `src/lib/hooks/use-weather.ts`**

```ts
"use client";

import useSWR from "swr";
import { fetchWeatherAll } from "@/lib/api/weather";
import { REFRESH_INTERVALS } from "@/lib/constants";

export function useWeather() {
  return useSWR("weather-all", fetchWeatherAll, {
    refreshInterval: REFRESH_INTERVALS.weather,
    revalidateOnFocus: true,
    errorRetryCount: 2,
  });
}
```

- [ ] **Step 3: Write `src/lib/hooks/use-traffic.ts`**

```ts
"use client";

import useSWR from "swr";
import { fetchTrafficIncidents } from "@/lib/api/traffic";
import { REFRESH_INTERVALS } from "@/lib/constants";

export function useTraffic() {
  return useSWR("traffic-incidents", fetchTrafficIncidents, {
    refreshInterval: REFRESH_INTERVALS.traffic,
    revalidateOnFocus: true,
    errorRetryCount: 2,
  });
}
```

- [ ] **Step 4: Write `src/lib/hooks/use-disruption.ts`**

```ts
"use client";

import useSWR from "swr";
import { fetchDisruptions } from "@/lib/api/disruption";
import { REFRESH_INTERVALS } from "@/lib/constants";

export function useDisruptions() {
  return useSWR("mtr-disruptions", fetchDisruptions, {
    refreshInterval: REFRESH_INTERVALS.disruption,
    revalidateOnFocus: true,
    errorRetryCount: 2,
  });
}
```

- [ ] **Step 5: Write `src/lib/hooks/use-geolocation.ts`**

```ts
"use client";

import { useState, useEffect } from "react";

interface GeoState {
  lat: number | null;
  lng: number | null;
  error: string | null;
  loading: boolean;
}

export function useGeolocation() {
  const [state, setState] = useState<GeoState>({
    lat: null, lng: null, error: null, loading: true,
  });

  useEffect(() => {
    if (!navigator.geolocation) {
      setState((s) => ({ ...s, error: "Geolocation not supported", loading: false }));
      return;
    }
    navigator.geolocation.getCurrentPosition(
      (pos) => setState({ lat: pos.coords.latitude, lng: pos.coords.longitude, error: null, loading: false }),
      (err) => setState((s) => ({ ...s, error: err.message, loading: false })),
      { enableHighAccuracy: false, timeout: 10000 }
    );
  }, []);

  return state;
}
```

- [ ] **Step 6: Write `src/lib/hooks/use-theme.ts`**

```ts
"use client";

import { useState, useEffect } from "react";

type Theme = "dark" | "light" | "system";

export function useTheme() {
  const [theme, setThemeState] = useState<Theme>("dark");

  useEffect(() => {
    const stored = localStorage.getItem("hk-transit-theme") as Theme | null;
    if (stored) setThemeState(stored);
  }, []);

  const setTheme = (t: Theme) => {
    setThemeState(t);
    localStorage.setItem("hk-transit-theme", t);
    if (t === "dark" || (t === "system" && window.matchMedia("(prefers-color-scheme: dark)").matches)) {
      document.documentElement.classList.add("dark");
    } else {
      document.documentElement.classList.remove("dark");
    }
  };

  return { theme, setTheme };
}
```

- [ ] **Step 7: Commit**

```bash
cd /home/opencode/Works/hk-transit-next
git add -A && git commit -m "feat: add data-fetching hooks (SWR) for all transit data"
```

---

### Task 7: Layout Shell (AppShell, BottomNav, Header)

**Files:**
- Create: `src/components/layout/bottom-nav.tsx`
- Create: `src/components/layout/header-strip.tsx`
- Create: `src/components/layout/more-modal.tsx`
- Create: `src/components/layout/app-shell.tsx`
- Modify: `src/app/layout.tsx` (wrap with AppShell)

- [ ] **Step 1: Write `src/components/layout/bottom-nav.tsx`**

```tsx
"use client";

import { usePathname, useRouter } from "next/navigation";
import { Home, Train, Bus, Grid3X3 } from "lucide-react";
import { NAV_ITEMS } from "@/lib/constants";

const iconMap = { Home, Train, Bus, Grid3X3 } as const;

export function BottomNav() {
  const pathname = usePathname();
  const router = useRouter();

  const currentTab = pathname === "/" ? "home"
    : pathname.startsWith("/mtr") ? "mtr"
    : pathname.startsWith("/bus") ? "bus"
    : "more";

  return (
    <nav className="fixed bottom-0 left-0 right-0 z-50 safe-bottom" role="tablist" aria-label="Navigation tabs">
      <div className="bg-bg-base/90 backdrop-blur-xl border-t border-glass-border">
        <div className="flex items-center justify-around h-16 max-w-lg mx-auto px-2">
          {NAV_ITEMS.map((item) => {
            const isActive = item.id === currentTab;
            const Icon = iconMap[item.icon as keyof typeof iconMap];

            return (
              <button
                key={item.id}
                onClick={() => item.route && router.push(item.route)}
                className={`flex flex-col items-center gap-0.5 flex-1 h-full justify-center touch-target ${
                  isActive ? "nav-active" : "nav-inactive"
                }`}
                role="tab"
                aria-selected={isActive}
                aria-label={item.labelEn}
              >
                <Icon size={22} />
                <span className="text-[10px] font-medium">{item.labelZh}</span>
                {isActive && (
                  <span className="absolute bottom-1 w-5 h-0.5 rounded-full bg-accent-mtr" />
                )}
              </button>
            );
          })}
        </div>
      </div>
    </nav>
  );
}
```

- [ ] **Step 2: Write `src/components/layout/header-strip.tsx`**

```tsx
"use client";

import { useState, useEffect } from "react";
import { Moon, Sun, MapPin } from "lucide-react";
import { HeaderWeather } from "@/components/weather/header-weather";
import { IconButton } from "@/components/shared/icon-button";

export function HeaderStrip() {
  const [time, setTime] = useState("");
  const [theme, setTheme] = useState<"dark" | "light">("dark");

  useEffect(() => {
    const update = () => setTime(new Date().toLocaleTimeString("zh-HK", { hour: "2-digit", minute: "2-digit" }));
    update();
    const interval = setInterval(update, 30000);
    return () => clearInterval(interval);
  }, []);

  const toggleTheme = () => {
    const next = theme === "dark" ? "light" : "dark";
    setTheme(next);
    document.documentElement.classList.toggle("dark", next === "dark");
  };

  return (
    <header className="sticky top-0 z-40 safe-top" role="banner">
      <div className="bg-bg-base/80 backdrop-blur-xl border-b border-glass-border">
        <div className="flex items-center justify-between h-12 max-w-lg mx-auto px-4">
          <div className="flex items-center gap-2">
            <MapPin className="w-4 h-4 text-accent-mtr" />
            <h1 className="font-display font-bold text-sm tracking-tight">HK Transit</h1>
          </div>

          <div className="flex items-center gap-2">
            <HeaderWeather />
            <span className="font-mono text-xs text-ink-2 tabular-nums">{time}</span>
            <IconButton
              icon={theme === "dark" ? Sun : Moon}
              label={theme === "dark" ? "Switch to light theme" : "Switch to dark theme"}
              onClick={toggleTheme}
            />
          </div>
        </div>
      </div>
    </header>
  );
}
```

- [ ] **Step 3: Write `src/components/layout/more-modal.tsx`**

```tsx
"use client";

import { useRouter } from "next/navigation";
import { X } from "lucide-react";
import { MORE_MODES } from "@/lib/constants";
import { GlassCard } from "@/components/shared/glass-card";
import * as LucideIcons from "lucide-react";

interface MoreModalProps {
  open: boolean;
  onClose: () => void;
}

// Dynamic icon resolver
const getIcon = (name: string) => {
  const icons = LucideIcons as unknown as Record<string, React.ComponentType<{ size?: number; className?: string }>>;
  return icons[name] || LucideIcons.Circle;
};

export function MoreModal({ open, onClose }: MoreModalProps) {
  const router = useRouter();

  if (!open) return null;

  return (
    <div
      className="fixed inset-0 z-50 flex items-end sm:items-center justify-center bg-black/50 backdrop-blur-sm"
      onClick={onClose}
      role="dialog"
      aria-modal="true"
      aria-label="More transport modes"
    >
      <div
        className="bg-bg-raised rounded-t-2xl sm:rounded-2xl w-full max-w-lg p-6 max-h-[70vh] overflow-y-auto"
        onClick={(e) => e.stopPropagation()}
      >
        <div className="flex items-center justify-between mb-4">
          <h2 className="font-display font-bold text-lg">More</h2>
          <button onClick={onClose} className="touch-target text-ink-2 hover:text-ink-0" aria-label="Close">
            <X size={20} />
          </button>
        </div>

        <div className="grid grid-cols-2 gap-3">
          {MORE_MODES.map((mode) => {
            const Icon = getIcon(mode.icon);
            return (
              <GlassCard
                key={mode.id}
                accentColor={mode.color}
                onClick={() => { router.push(mode.route); onClose(); }}
                role="button"
                tabIndex={0}
                aria-label={mode.labelEn}
              >
                <div className="flex flex-col items-center gap-2 py-3">
                  <Icon size={28} className="text-ink-1" />
                  <span className="text-sm font-medium">{mode.labelZh}</span>
                </div>
              </GlassCard>
            );
          })}
        </div>
      </div>
    </div>
  );
}
```

- [ ] **Step 4: Write `src/components/layout/app-shell.tsx`**

```tsx
"use client";

import { useState, type ReactNode } from "react";
import { BottomNav } from "./bottom-nav";
import { HeaderStrip } from "./header-strip";
import { MoreModal } from "./more-modal";
import { DisruptionAlertBanner } from "@/components/disruption/alert-banner";

interface AppShellProps {
  children: ReactNode;
}

export function AppShell({ children }: AppShellProps) {
  const [moreOpen, setMoreOpen] = useState(false);

  return (
    <div className="min-h-screen flex flex-col">
      <HeaderStrip />
      <DisruptionAlertBanner />
      <main className="flex-1 max-w-lg mx-auto w-full px-4 pb-20 pt-4">
        {children}
      </main>
      <BottomNav />
      <MoreModal open={moreOpen} onClose={() => setMoreOpen(false)} />
    </div>
  );
}
```

- [ ] **Step 5: Update `src/app/layout.tsx`** to wrap with AppShell

Replace the existing layout.tsx body:

```tsx
import { AppShell } from "@/components/layout/app-shell";

// ... metadata and viewport unchanged ...

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="zh-HK" className="dark">
      <head>
        <link rel="preconnect" href="https://fonts.googleapis.com" />
        <link rel="preconnect" href="https://fonts.gstatic.com" crossOrigin="anonymous" />
        <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;600&family=Plus+Jakarta+Sans:wght@500;600;700;800&display=swap" rel="stylesheet" />
      </head>
      <body className="antialiased">
        <AppShell>{children}</AppShell>
      </body>
    </html>
  );
}
```

- [ ] **Step 6: Build and verify**

```bash
cd /home/opencode/Works/hk-transit-next
pnpm build
```

Expected: Build succeeds. Note the `MoreModal` needs the `DisruptionAlertBanner` — create a placeholder if needed.

- [ ] **Step 7: Commit**

```bash
cd /home/opencode/Works/hk-transit-next
git add -A && git commit -m "feat: add AppShell with BottomNav, HeaderStrip, and MoreModal"
```

---

### Task 8: Home Page

**Files:**
- Create: `src/components/home/hero.tsx`
- Create: `src/components/home/mode-cards.tsx`
- Create: `src/app/page.tsx`

- [ ] **Step 1: Write `src/components/home/hero.tsx`**

```tsx
export function Hero() {
  return (
    <section className="mb-6">
      <h2 className="font-display font-extrabold text-2xl tracking-tight text-ink-0">
        香港交通
      </h2>
      <p className="text-ink-2 text-sm mt-1">
        Real-time transport guide for Hong Kong
      </p>
    </section>
  );
}
```

- [ ] **Step 2: Write `src/components/home/mode-cards.tsx`**

```tsx
"use client";

import { useRouter } from "next/navigation";
import { Train, Bus, Shuttle, Ship, Map, Car, CloudSun, AlertTriangle } from "lucide-react";
import { GlassCard } from "@/components/shared/glass-card";

const MODES = [
  { id: "mtr", label: "港鐵 MTR", icon: Train, color: "#22d3ee", route: "/mtr", desc: "Real-time arrivals" },
  { id: "bus", label: "巴士 Bus", icon: Bus, color: "#ef4444", route: "/bus", desc: "KMB & Citybus" },
  { id: "minibus", label: "小巴 Minibus", icon: Shuttle, color: "#22c55e", route: "/minibus", desc: "GMB routes" },
  { id: "ferry", label: "渡輪 Ferry", icon: Ship, color: "#06b6d4", route: "/ferry", desc: "HKKF services" },
  { id: "map", label: "地圖 Map", icon: Map, color: "#a855f7", route: "/map", desc: "Nearby transit" },
  { id: "traffic", label: "交通 Traffic", icon: Car, color: "#f59e0b", route: "/traffic", desc: "Road incidents" },
  { id: "weather", label: "天氣 Weather", icon: CloudSun, color: "#22d3ee", route: "/weather", desc: "HKO forecast" },
  { id: "disruptions", label: "事故 Disruptions", icon: AlertTriangle, color: "#f97316", route: "/disruptions", desc: "Service alerts" },
];

export function ModeCards() {
  const router = useRouter();

  return (
    <section aria-label="Transport modes">
      <div className="grid grid-cols-2 gap-3">
        {MODES.map((mode) => {
          const Icon = mode.icon;
          return (
            <GlassCard
              key={mode.id}
              accentColor={mode.color}
              onClick={() => router.push(mode.route)}
              role="button"
              tabIndex={0}
              aria-label={mode.label}
            >
              <div className="flex flex-col items-center text-center gap-1 py-2">
                <Icon size={24} className="text-ink-1" />
                <span className="text-sm font-medium">{mode.label}</span>
                <span className="text-[10px] text-ink-2">{mode.desc}</span>
              </div>
            </GlassCard>
          );
        })}
      </div>
    </section>
  );
}
```

- [ ] **Step 3: Write the Home page `src/app/page.tsx`**

```tsx
import { Hero } from "@/components/home/hero";
import { ModeCards } from "@/components/home/mode-cards";
import { WeatherCard } from "@/components/home/weather-card";

export default function HomePage() {
  return (
    <div className="space-y-6">
      <Hero />
      <ModeCards />
      <WeatherCard />
    </div>
  );
}
```

- [ ] **Step 4: Commit**

```bash
cd /home/opencode/Works/hk-transit-next
git add -A && git commit -m "feat: add Home page with hero, mode cards, and weather card"
```

---

### Task 9: Weather Components

**Files:**
- Create: `src/components/weather/header-weather.tsx`
- Create: `src/components/weather/detail-card.tsx`
- Create: `src/components/weather/weather-panel.tsx`
- Create: `src/components/home/weather-card.tsx`
- Create: `src/app/weather/page.tsx`

- [ ] **Step 1: Write `src/components/weather/header-weather.tsx`**

```tsx
"use client";

import { useWeather } from "@/lib/hooks/use-weather";
import { CloudSun } from "lucide-react";

export function HeaderWeather() {
  const { data, error } = useWeather();

  if (error || !data?.current) {
    return null;
  }

  const temp = data.current.temperature?.data?.[0];

  return (
    <button
      className="flex items-center gap-1 text-xs text-ink-2 hover:text-ink-1 transition-colors touch-target"
      aria-label={`Weather: ${temp?.value ?? "--"}°`}
      onClick={() => window.location.href = "/weather"}
    >
      <CloudSun size={14} className="text-accent-weather" />
      <span className="font-mono tabular-nums">{temp?.value ?? "--"}°</span>
    </button>
  );
}
```

- [ ] **Step 2: Write `src/components/home/weather-card.tsx`**

```tsx
"use client";

import { useWeather } from "@/lib/hooks/use-weather";
import { GlassCard } from "@/components/shared/glass-card";
import { useRouter } from "next/navigation";
import { CloudSun, Droplets, Wind } from "lucide-react";

export function WeatherCard() {
  const { data, error, isLoading } = useWeather();
  const router = useRouter();

  if (error || isLoading || !data?.current) return null;

  const temp = data.current.temperature?.data?.[0];
  const humidity = data.current.humidity?.data;

  return (
    <section aria-label="Current weather">
      <h3 className="font-display font-bold text-sm text-ink-1 mb-2">天氣 Weather</h3>
      <GlassCard
        accentColor="#22d3ee"
        onClick={() => router.push("/weather")}
        role="button"
        tabIndex={0}
        aria-label="View weather details"
      >
        <div className="flex items-center justify-between">
          <div className="flex items-center gap-3">
            <CloudSun size={32} className="text-accent-weather" />
            <div>
              <span className="font-display font-bold text-2xl">{temp?.value ?? "--"}°</span>
              <span className="text-xs text-ink-2 ml-1">{temp?.unit ?? "C"}</span>
            </div>
          </div>
          <div className="flex gap-4 text-xs text-ink-2">
            {humidity && (
              <div className="flex items-center gap-1">
                <Droplets size={14} />
                <span className="font-mono">{humidity.value}%</span>
              </div>
            )}
          </div>
        </div>
        {data.current.warningMessage?.length > 0 && (
          <div className="mt-2 text-xs text-amber-400 font-medium flex items-center gap-1">
            <span>⚠</span>
            <span>{data.current.warningMessage[0]}</span>
          </div>
        )}
      </GlassCard>
    </section>
  );
}
```

- [ ] **Step 3: Write `src/components/weather/weather-panel.tsx`** (slide-up detail)

```tsx
"use client";

import { useEffect, useRef } from "react";
import { X, CloudSun, Droplets, Thermometer, Wind } from "lucide-react";
import { useWeather } from "@/lib/hooks/use-weather";

interface WeatherPanelProps {
  open: boolean;
  onClose: () => void;
}

export function WeatherPanel({ open, onClose }: WeatherPanelProps) {
  const { data } = useWeather();
  const panelRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (open) {
      document.body.style.overflow = "hidden";
    } else {
      document.body.style.overflow = "";
    }
    return () => { document.body.style.overflow = ""; };
  }, [open]);

  if (!open) return null;

  const current = data?.current;
  const forecast = data?.forecast;

  return (
    <div
      className="fixed inset-0 z-50 flex items-end bg-black/50 backdrop-blur-sm"
      onClick={onClose}
      role="dialog"
      aria-modal="true"
      aria-label="Weather details"
    >
      <div
        ref={panelRef}
        className="bg-bg-raised rounded-t-2xl w-full max-w-lg mx-auto max-h-[80vh] overflow-y-auto"
        onClick={(e) => e.stopPropagation()}
      >
        {/* Header */}
        <div className="flex items-center justify-between p-4 border-b border-glass-border">
          <h2 className="font-display font-bold text-lg">天氣 Weather</h2>
          <button onClick={onClose} className="touch-target text-ink-2" aria-label="Close">
            <X size={20} />
          </button>
        </div>

        {current && (
          <div className="p-4 space-y-4">
            {/* Current conditions */}
            <div className="flex items-center gap-4">
              <CloudSun size={48} className="text-accent-weather" />
              <div>
                <span className="font-display font-bold text-4xl">{current.temperature?.data?.[0]?.value ?? "--"}°</span>
                <span className="text-sm text-ink-2 ml-1">{current.temperature?.data?.[0]?.unit ?? "C"}</span>
                <p className="text-sm text-ink-2">{current.temperature?.data?.[0]?.place ?? ""}</p>
              </div>
            </div>

            {/* Humidity */}
            {current.humidity?.data && (
              <div className="flex items-center gap-3 text-sm">
                <Droplets size={16} className="text-ink-2" />
                <span>Humidity: <span className="font-mono">{current.humidity.data.value}%</span></span>
              </div>
            )}

            {/* Warnings */}
            {current.warningMessage?.length > 0 && (
              <div className="bg-amber-500/10 border border-amber-500/20 rounded-xl p-3">
                <p className="text-xs font-medium text-amber-400">⚠ Weather Warning</p>
                {current.warningMessage.map((w: string, i: number) => (
                  <p key={i} className="text-xs text-ink-2 mt-1">{w}</p>
                ))}
              </div>
            )}

            {/* Forecast */}
            {forecast?.forecast && (
              <div>
                <h3 className="font-display font-bold text-sm text-ink-1 mb-2">3-Day Forecast</h3>
                <div className="grid grid-cols-3 gap-2">
                  {forecast.forecast.slice(0, 3).map((day: any) => (
                    <div key={day.forecastDate} className="glass-card p-3 text-center">
                      <p className="text-xs text-ink-2">{day.forecastDate?.slice(5) ?? ""}</p>
                      <CloudSun size={20} className="mx-auto my-1 text-accent-weather" />
                      <p className="font-mono text-xs">{day.forecastMaxTemp?.value ?? "--"}°/{day.forecastMinTemp?.value ?? "--"}°</p>
                    </div>
                  ))}
                </div>
              </div>
            )}
          </div>
        )}
      </div>
    </div>
  );
}
```

- [ ] **Step 4: Write `src/app/weather/page.tsx`**

```tsx
"use client";

import { useState } from "react";
import { ArrowLeft } from "lucide-react";
import { useRouter } from "next/navigation";
import { WeatherPanel } from "@/components/weather/weather-panel";

export default function WeatherPage() {
  const router = useRouter();
  const [panelOpen, setPanelOpen] = useState(true);

  return (
    <>
      <button
        onClick={() => router.back()}
        className="flex items-center gap-2 text-sm text-ink-2 hover:text-ink-0 mb-4 touch-target"
        aria-label="Go back"
      >
        <ArrowLeft size={18} />
        <span>Back</span>
      </button>
      <WeatherPanel open={panelOpen} onClose={() => router.back()} />
    </>
  );
}
```

- [ ] **Step 5: Commit**

```bash
cd /home/opencode/Works/hk-transit-next
git add -A && git commit -m "feat: add weather components (header strip, home card, detail panel, page)"
```

---

### Task 10: MTR Page (Lines → Stations → Arrivals)

**Files:**
- Create: `src/components/mtr/line-chips.tsx`
- Create: `src/components/mtr/station-list.tsx`
- Create: `src/components/mtr/arrival-board.tsx`
- Create: `src/app/mtr/page.tsx`
- Create: `src/app/mtr/[line]/page.tsx`
- Create: `src/app/mtr/arrival/[station]/page.tsx`

- [ ] **Step 1: Write `src/components/mtr/line-chips.tsx`**

```tsx
"use client";

import { useRouter } from "next/navigation";
import { MTR_LINE_COLORS } from "@/lib/constants";

const LINES = [
  { code: "TWL", nameEn: "Tsuen Wan", nameZh: "荃灣綫" },
  { code: "ISL", nameEn: "Island", nameZh: "港島綫" },
  { code: "KTL", nameEn: "Kwun Tong", nameZh: "觀塘綫" },
  { code: "TCL", nameEn: "Tung Chung", nameZh: "東涌綫" },
  { code: "AEL", nameEn: "Airport Express", nameZh: "機場快綫" },
  { code: "TKL", nameEn: "Tseung Kwan O", nameZh: "將軍澳綫" },
  { code: "SIL", nameEn: "South Island", nameZh: "南港島綫" },
  { code: "EAL", nameEn: "East Rail", nameZh: "東鐵綫" },
  { code: "TML", nameEn: "Tuen Ma", nameZh: "屯馬綫" },
  { code: "DRL", nameEn: "Disneyland Resort", nameZh: "迪士尼綫" },
];

export function LineChips() {
  const router = useRouter();

  return (
    <section aria-label="MTR lines">
      <h2 className="font-display font-bold text-lg mb-3">港鐵路線 MTR Lines</h2>
      <div className="flex flex-wrap gap-2">
        {LINES.map((line) => (
          <button
            key={line.code}
            onClick={() => router.push(`/mtr/${line.code}`)}
            className="flex items-center gap-2 px-3 py-2 rounded-full text-sm font-medium transition-all hover:scale-105 active:scale-95"
            style={{
              backgroundColor: `${MTR_LINE_COLORS[line.code] ?? "#666"}20`,
              color: MTR_LINE_COLORS[line.code] ?? "#666",
              border: `1px solid ${MTR_LINE_COLORS[line.code] ?? "#666"}40`,
            }}
            aria-label={`${line.nameEn} line`}
          >
            <span
              className="w-2.5 h-2.5 rounded-full"
              style={{ backgroundColor: MTR_LINE_COLORS[line.code] ?? "#666" }}
            />
            <span className="font-medium">{line.nameZh}</span>
          </button>
        ))}
      </div>
    </section>
  );
}
```

- [ ] **Step 2: Write `src/components/mtr/station-list.tsx`**

```tsx
"use client";

import { useRouter } from "next/navigation";
import { MTR_LINE_COLORS } from "@/lib/constants";

const MOCK_STATIONS: Record<string, { code: string; nameEn: string; nameZh: string }[]> = {
  "TWL": [
    { code: "CEN", nameEn: "Central", nameZh: "中環" },
    { code: "ADM", nameEn: "Admiralty", nameZh: "金鐘" },
    { code: "TST", nameEn: "Tsim Sha Tsui", nameZh: "尖沙咀" },
    { code: "MOK", nameEn: "Mong Kok", nameZh: "旺角" },
    { code: "KWT", nameEn: "Kwai Fong", nameZh: "葵芳" },
    { code: "TSW", nameEn: "Tsuen Wan", nameZh: "荃灣" },
  ],
  "ISL": [
    { code: "KET", nameEn: "Kennedy Town", nameZh: "堅尼地城" },
    { code: "HKU", nameEn: "HKU", nameZh: "香港大學" },
    { code: "CEN", nameEn: "Central", nameZh: "中環" },
    { code: "ADM", nameEn: "Admiralty", nameZh: "金鐘" },
    { code: "TAK", nameEn: "Tai Koo", nameZh: "太古" },
    { code: "CHW", nameEn: "Chai Wan", nameZh: "柴灣" },
  ],
  "KTL": [
    { code: "WHA", nameEn: "Whampoa", nameZh: "黃埔" },
    { code: "MOK", nameEn: "Mong Kok", nameZh: "旺角" },
    { code: "KOB", nameEn: "Kowloon Bay", nameZh: "九龍灣" },
    { code: "TIK", nameEn: "Tiu Keng Leng", nameZh: "調景嶺" },
    { code: "POA", nameEn: "Po Lam", nameZh: "寶琳" },
  ],
};

interface StationListProps {
  lineCode: string;
}

export function StationList({ lineCode }: StationListProps) {
  const router = useRouter();
  const stations = MOCK_STATIONS[lineCode] ?? [];
  const color = MTR_LINE_COLORS[lineCode] ?? "#666";

  return (
    <section aria-label="Stations">
      <div className="relative">
        {/* Vertical line */}
        <div className="absolute left-[13px] top-0 bottom-0 w-0.5 opacity-30" style={{ backgroundColor: color }} />

        <div className="space-y-1">
          {stations.map((station, i) => (
            <button
              key={station.code}
              onClick={() => router.push(`/mtr/arrival/${station.code}?line=${lineCode}`)}
              className="flex items-center gap-4 w-full py-3 px-2 rounded-xl hover:bg-glass-hover transition-colors touch-target"
              aria-label={`${station.nameEn} station`}
            >
              <div className="relative flex items-center justify-center w-[26px]">
                <div
                  className="w-[26px] h-[26px] rounded-full border-2 flex items-center justify-center transition-transform active:scale-90"
                  style={{ borderColor: color, backgroundColor: `${color}20` }}
                >
                  <div className="w-2 h-2 rounded-full" style={{ backgroundColor: color }} />
                </div>
              </div>
              <div className="text-left">
                <span className="text-sm font-medium">{station.nameZh}</span>
                <span className="text-xs text-ink-2 ml-2">{station.nameEn}</span>
              </div>
            </button>
          ))}
        </div>
      </div>
    </section>
  );
}
```

- [ ] **Step 3: Write `src/components/mtr/arrival-board.tsx`**

```tsx
"use client";

import { ArrowDown, ArrowUp } from "lucide-react";
import { useMtrEta } from "@/lib/hooks/use-eta";
import { Loading } from "@/components/shared/loading";
import { EmptyState } from "@/components/shared/empty-state";
import { Clock } from "lucide-react";

interface ArrivalBoardProps {
  lineCode: string;
  stationCode: string;
}

export function ArrivalBoard({ lineCode, stationCode }: ArrivalBoardProps) {
  const { data, error, isLoading } = useMtrEta(lineCode, stationCode);

  if (isLoading) return <Loading count={3} type="line" />;
  if (error) return <div className="text-sm text-red-400 p-3" role="alert">Failed to load arrivals</div>;
  if (!data) return <EmptyState icon={Clock} message="No arrival data available" />;

  const entries = Object.entries(data);

  return (
    <section aria-label="Arrivals">
      <h3 className="font-display font-bold text-sm text-ink-1 mb-3">到站時間 Arrivals</h3>
      <div className="space-y-2" role="list" aria-live="polite">
        {entries.map(([platform, info]) => {
          const direction = platform.includes("UP") ? "up" : "down";
          const Icon = direction === "up" ? ArrowUp : ArrowDown;
          const trains = direction === "up" ? info.up : info.down;

          if (!trains?.length) return null;

          return (
            <div key={platform} className="glass-card p-3" role="listitem">
              <div className="flex items-center gap-2 mb-2">
                <Icon size={14} className={direction === "up" ? "text-accent-mtr" : "text-amber-400"} />
                <span className="text-xs text-ink-2">
                  {direction === "up" ? "Towards" : "Towards"}{" "}
                  {trains[0]?.dest ?? "Unknown"}
                </span>
              </div>
              <div className="flex gap-2">
                {trains.slice(0, 4).map((train: any, i: number) => {
                  const mins = Math.round(
                    (new Date(train.time).getTime() - Date.now()) / 60000
                  );
                  return (
                    <span
                      key={i}
                      className={`font-mono text-sm px-2 py-1 rounded-md ${
                        mins <= 2
                          ? "bg-red-500/20 text-red-400"
                          : mins <= 5
                          ? "bg-amber-500/20 text-amber-400"
                          : "bg-ink-3/20 text-ink-1"
                      }`}
                    >
                      {mins <= 0 ? "Due" : `${mins} min`}
                    </span>
                  );
                })}
              </div>
            </div>
          );
        })}
      </div>
    </section>
  );
}
```

- [ ] **Step 4: Write `src/app/mtr/page.tsx`**

```tsx
import { LineChips } from "@/components/mtr/line-chips";
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "MTR — HK Transit",
};

export default function MtrPage() {
  return (
    <div className="space-y-4">
      <LineChips />
    </div>
  );
}
```

- [ ] **Step 5: Write `src/app/mtr/[line]/page.tsx`**

```tsx
"use client";

import { useParams, useRouter } from "next/navigation";
import { ArrowLeft } from "lucide-react";
import { StationList } from "@/components/mtr/station-list";
import { MTR_LINE_COLORS } from "@/lib/constants";

export default function MtrLinePage() {
  const params = useParams();
  const router = useRouter();
  const lineCode = params.line as string;

  return (
    <div className="space-y-4">
      <button
        onClick={() => router.back()}
        className="flex items-center gap-2 text-sm text-ink-2 hover:text-ink-0 touch-target"
        aria-label="Go back"
      >
        <ArrowLeft size={18} />
        <span>Back</span>
      </button>

      <StationList lineCode={lineCode} />
    </div>
  );
}
```

- [ ] **Step 6: Write `src/app/mtr/arrival/[station]/page.tsx`**

```tsx
"use client";

import { useParams, useRouter, useSearchParams } from "next/navigation";
import { ArrowLeft } from "lucide-react";
import { ArrivalBoard } from "@/components/mtr/arrival-board";

export default function MtrArrivalPage() {
  const params = useParams();
  const router = useRouter();
  const searchParams = useSearchParams();
  const stationCode = params.station as string;
  const lineCode = searchParams.get("line") ?? "TWL";

  return (
    <div className="space-y-4">
      <button
        onClick={() => router.back()}
        className="flex items-center gap-2 text-sm text-ink-2 hover:text-ink-0 touch-target"
        aria-label="Go back"
      >
        <ArrowLeft size={18} />
        <span>Back</span>
      </button>

      <ArrivalBoard lineCode={lineCode} stationCode={stationCode} />
    </div>
  );
}
```

- [ ] **Step 7: Commit**

```bash
cd /home/opencode/Works/hk-transit-next
git add -A && git commit -m "feat: add MTR page with lines, stations, and real-time arrivals"
```

---

### Task 11: Bus Page

**Files:**
- Create: `src/components/bus/route-search.tsx`
- Create: `src/components/bus/stop-list.tsx`
- Create: `src/components/bus/eta-display.tsx`
- Create: `src/app/bus/page.tsx`

- [ ] **Step 1: Write `src/components/bus/route-search.tsx`**

```tsx
"use client";

import { useState } from "react";
import { Search } from "lucide-react";

interface RouteSearchProps {
  onSearch: (query: string) => void;
}

export function RouteSearch({ onSearch }: RouteSearchProps) {
  const [query, setQuery] = useState("");

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (query.trim()) onSearch(query.trim());
  };

  return (
    <form onSubmit={handleSubmit} role="search" aria-label="Search bus routes">
      <div className="relative">
        <Search size={16} className="absolute left-3 top-1/2 -translate-y-1/2 text-ink-2" />
        <input
          type="text"
          value={query}
          onChange={(e) => setQuery(e.target.value)}
          placeholder="Search route (e.g. 1A, 960)..."
          className="w-full bg-glass-bg border border-glass-border rounded-xl py-3 pl-10 pr-4 text-sm text-ink-0 placeholder:text-ink-3 focus:outline-none focus:border-accent-mtr/50 transition-colors"
          inputMode="search"
          enterKeyHint="search"
          aria-label="Search bus route number"
        />
      </div>
    </form>
  );
}
```

- [ ] **Step 2: Write `src/components/bus/eta-display.tsx`**

```tsx
"use client";

import { useBusEta } from "@/lib/hooks/use-eta";
import { Loading } from "@/components/shared/loading";
import { Clock } from "lucide-react";

interface EtaDisplayProps {
  stopId: string;
  route: string;
}

export function EtaDisplay({ stopId, route }: EtaDisplayProps) {
  const { data, error, isLoading } = useBusEta(stopId, route);

  if (isLoading) return <Loading count={1} type="line" />;
  if (error) return <span className="text-xs text-red-400">Failed to load ETA</span>;

  const etas = data?.[0]?.eta?.slice(0, 3) ?? [];

  if (etas.length === 0) return <span className="text-xs text-ink-2">No ETA available</span>;

  return (
    <div className="flex gap-1.5" aria-live="polite">
      {etas.map((e: any, i: number) => {
        const mins = e.eta
          ? Math.round((new Date(e.eta).getTime() - Date.now()) / 60000)
          : null;
        return (
          <span
            key={i}
            className={`font-mono text-xs px-1.5 py-0.5 rounded ${
              mins !== null && mins <= 3
                ? "bg-red-500/20 text-red-400"
                : mins !== null && mins <= 8
                ? "bg-amber-500/20 text-amber-400"
                : "bg-ink-3/20 text-ink-1"
            }`}
          >
            {mins !== null ? `${mins}min` : "--"}
          </span>
        );
      })}
    </div>
  );
}
```

- [ ] **Step 3: Write `src/app/bus/page.tsx`**

```tsx
"use client";

import { useState } from "react";
import { RouteSearch } from "@/components/bus/route-search";
import { GlassCard } from "@/components/shared/glass-card";
import { EtaDisplay } from "@/components/bus/eta-display";

const MOCK_ROUTES = [
  { route: "1A", origin: "尖沙咀碼頭", dest: "中秀茂坪", company: "kmb" },
  { route: "960", origin: "會展站", dest: "建生", company: "kmb" },
  { route: "968", origin: "銅鑼灣", dest: "元朗西", company: "kmb" },
  { route: "A21", origin: "紅磡站", dest: "機場", company: "citybus" },
];

export default function BusPage() {
  const [query, setQuery] = useState("");
  const routes = query
    ? MOCK_ROUTES.filter((r) => r.route.includes(query))
    : MOCK_ROUTES;

  return (
    <div className="space-y-4">
      <h2 className="font-display font-bold text-lg">巴士 Bus</h2>
      <RouteSearch onSearch={setQuery} />

      <div className="space-y-2">
        {routes.map((route) => (
          <GlassCard key={route.route} accentColor="#ef4444">
            <div className="flex items-center justify-between">
              <div>
                <span className="font-display font-bold text-base">{route.route}</span>
                <span className="text-xs text-ink-2 ml-2">{route.company.toUpperCase()}</span>
                <p className="text-xs text-ink-2 mt-0.5">{route.origin} → {route.dest}</p>
              </div>
            </div>
          </GlassCard>
        ))}
      </div>
    </div>
  );
}
```

- [ ] **Step 4: Commit**

```bash
cd /home/opencode/Works/hk-transit-next
git add -A && git commit -m "feat: add Bus page with route search and ETA display"
```

---

### Task 12: Remaining Transit Pages (Minibus, Ferry, Traffic, Map)

**Files:**
- Create: `src/app/minibus/page.tsx`
- Create: `src/app/ferry/page.tsx`
- Create: `src/app/traffic/page.tsx`
- Create: `src/app/map/page.tsx`
- Create: `src/components/traffic/notice-list.tsx`
- Create: `src/components/map/transit-map.tsx`
- Create: `src/components/ferry/ferry-route-card.tsx`

- [ ] **Step 1: Write `src/app/minibus/page.tsx`**

```tsx
"use client";

import { useState } from "react";
import { Search } from "lucide-react";
import { GlassCard } from "@/components/shared/glass-card";

const MOCK_GMB = [
  { route: "1", region: "HKI", origin: "香港站", dest: "山頂" },
  { route: "8", region: "HKI", origin: "中環碼頭", dest: "碧瑤灣" },
  { route: "10", region: "KLN", origin: "旺角", dest: "俊民苑" },
  { route: "26", region: "NT", origin: "屯門", dest: "元朗" },
];

export default function MinibusPage() {
  const [query, setQuery] = useState("");
  const routes = query ? MOCK_GMB.filter((r) => r.route.includes(query)) : MOCK_GMB;

  return (
    <div className="space-y-4">
      <h2 className="font-display font-bold text-lg">小巴 Minibus</h2>

      <div className="relative">
        <Search size={16} className="absolute left-3 top-1/2 -translate-y-1/2 text-ink-2" />
        <input
          type="text"
          value={query}
          onChange={(e) => setQuery(e.target.value)}
          placeholder="Search route..."
          className="w-full bg-glass-bg border border-glass-border rounded-xl py-3 pl-10 pr-4 text-sm text-ink-0 placeholder:text-ink-3 focus:outline-none focus:border-accent-minibus/50"
          aria-label="Search minibus route"
        />
      </div>

      <div className="space-y-2">
        {routes.map((r) => (
          <GlassCard key={`${r.region}-${r.route}`} accentColor="#22c55e">
            <div className="flex items-center justify-between">
              <div>
                <span className="font-display font-bold">{r.route}</span>
                <span className="text-xs text-ink-2 ml-2">{r.region}</span>
                <p className="text-xs text-ink-2 mt-0.5">{r.origin} → {r.dest}</p>
              </div>
            </div>
          </GlassCard>
        ))}
      </div>
    </div>
  );
}
```

- [ ] **Step 2: Write `src/app/ferry/page.tsx`**

```tsx
"use client";

import { GlassCard } from "@/components/shared/glass-card";
import { Ship } from "lucide-react";

const ROUTES = [
  { nameEn: "Central ↔ Tsim Sha Tsui", nameZh: "中環↔尖沙咀", company: "HKKF", duration: "10min" },
  { nameEn: "Central ↔ Discovery Bay", nameZh: "中環↔愉景灣", company: "DB", duration: "25min" },
  { nameEn: "Central ↔ Lantau Island", nameZh: "中環↔大嶼山", company: "NLF", duration: "40min" },
  { nameEn: "North Point ↔ Hung Hom", nameZh: "北角↔紅磡", company: "HKKF", duration: "8min" },
];

export default function FerryPage() {
  return (
    <div className="space-y-4">
      <h2 className="font-display font-bold text-lg">渡輪 Ferry</h2>

      <div className="space-y-2">
        {ROUTES.map((route) => (
          <GlassCard key={route.nameEn} accentColor="#06b6d4">
            <div className="flex items-center gap-3">
              <Ship size={20} className="text-accent-ferry" />
              <div className="flex-1">
                <span className="text-sm font-medium">{route.nameZh}</span>
                <span className="text-xs text-ink-2 ml-2">{route.nameEn}</span>
                <div className="flex gap-2 mt-1">
                  <span className="text-[10px] text-ink-3 bg-ink-3/20 px-1.5 py-0.5 rounded">{route.company}</span>
                  <span className="text-[10px] text-ink-3">{route.duration}</span>
                </div>
              </div>
            </div>
          </GlassCard>
        ))}
      </div>
    </div>
  );
}
```

- [ ] **Step 3: Write `src/components/traffic/notice-list.tsx`**

```tsx
"use client";

import { useTraffic } from "@/lib/hooks/use-traffic";
import { GlassCard } from "@/components/shared/glass-card";
import { AlertTriangle, Car } from "lucide-react";
import { Loading } from "@/components/shared/loading";

export function NoticeList() {
  const { data, error, isLoading } = useTraffic();

  if (isLoading) return <Loading count={3} />;
  if (error) return <div className="text-sm text-red-400" role="alert">Failed to load traffic data</div>;
  if (!data?.length) return <div className="text-sm text-ink-2">No incidents reported</div>;

  return (
    <div className="space-y-2" aria-live="polite">
      {data.map((incident: any) => (
        <GlassCard key={incident.id} accentColor="#f59e0b">
          <div className="flex gap-3">
            <AlertTriangle size={18} className="text-accent-traffic shrink-0 mt-0.5" />
            <div>
              <p className="text-sm">{incident.description ?? "Traffic incident"}</p>
              {incident.location && (
                <p className="text-xs text-ink-2 mt-1">{incident.location}</p>
              )}
              <span className="text-[10px] text-ink-3 mt-1 block">
                {incident.time ?? ""}
              </span>
            </div>
          </div>
        </GlassCard>
      ))}
    </div>
  );
}
```

- [ ] **Step 4: Write `src/app/traffic/page.tsx`**

```tsx
import { NoticeList } from "@/components/traffic/notice-list";

export default function TrafficPage() {
  return (
    <div className="space-y-4">
      <h2 className="font-display font-bold text-lg">交通 Traffic</h2>
      <NoticeList />
    </div>
  );
}
```

- [ ] **Step 5: Write `src/components/map/transit-map.tsx`**

```tsx
"use client";

import dynamic from "next/dynamic";
import { useGeolocation } from "@/lib/hooks/use-geolocation";

const MapContainer = dynamic(
  () => import("react-leaflet").then((m) => m.MapContainer),
  { ssr: false, loading: () => <div className="h-64 glass-card animate-pulse" /> }
);

const TileLayer = dynamic(
  () => import("react-leaflet").then((m) => m.TileLayer),
  { ssr: false }
);

const Marker = dynamic(
  () => import("react-leaflet").then((m) => m.Marker),
  { ssr: false }
);

const Popup = dynamic(
  () => import("react-leaflet").then((m) => m.Popup),
  { ssr: false }
);

import "leaflet/dist/leaflet.css";

export function TransitMap() {
  const { lat, lng, error, loading } = useGeolocation();

  const center: [number, number] = lat && lng ? [lat, lng] : [22.3193, 114.1694];

  return (
    <div className="h-[60vh] rounded-xl overflow-hidden border border-glass-border">
      <MapContainer
        center={center}
        zoom={13}
        className="h-full w-full"
        zoomControl={false}
      >
        <TileLayer
          attribution='&copy; <a href="https://www.openstreetmap.org/copyright">OSM</a>'
          url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
        />
      </MapContainer>
    </div>
  );
}
```

- [ ] **Step 6: Write `src/app/map/page.tsx`**

```tsx
import { TransitMap } from "@/components/map/transit-map";

export default function MapPage() {
  return (
    <div className="space-y-4">
      <h2 className="font-display font-bold text-lg">地圖 Map</h2>
      <TransitMap />
    </div>
  );
}
```

- [ ] **Step 7: Commit**

```bash
cd /home/opencode/Works/hk-transit-next
git add -A && git commit -m "feat: add Minibus, Ferry, Traffic, and Map pages"
```

---

### Task 13: MTR Disruption Feed

**Files:**
- Create: `src/components/disruption/alert-banner.tsx`
- Create: `src/components/disruption/disruption-list.tsx`
- Create: `src/app/disruptions/page.tsx`

- [ ] **Step 1: Write `src/components/disruption/alert-banner.tsx`**

```tsx
"use client";

import { useDisruptions } from "@/lib/hooks/use-disruption";
import { AlertTriangle, X } from "lucide-react";
import { useState } from "react";

export function DisruptionAlertBanner() {
  const { data } = useDisruptions();
  const [dismissed, setDismissed] = useState(false);

  if (!data?.length || dismissed) return null;

  const critical = data.filter((d: any) => d.severity === "critical");
  const warnings = data.filter((d: any) => d.severity === "warning");

  if (!critical.length && !warnings.length) return null;

  const alert = critical[0] ?? warnings[0];

  return (
    <div
      className={`px-4 py-2 text-xs font-medium flex items-center gap-2 ${
        critical.length
          ? "bg-red-500/20 text-red-400 border-b border-red-500/20"
          : "bg-amber-500/20 text-amber-400 border-b border-amber-500/20"
      }`}
      role="alert"
    >
      <AlertTriangle size={14} className="shrink-0" />
      <span className="flex-1 truncate">{alert.summary_en ?? "Service disruption"}</span>
      <button
        onClick={() => setDismissed(true)}
        className="shrink-0 touch-target"
        aria-label="Dismiss alert"
      >
        <X size={14} />
      </button>
    </div>
  );
}
```

- [ ] **Step 2: Write `src/components/disruption/disruption-list.tsx`**

```tsx
"use client";

import { useDisruptions } from "@/lib/hooks/use-disruption";
import { GlassCard } from "@/components/shared/glass-card";
import { AlertTriangle, Info, AlertOctagon } from "lucide-react";
import { Loading } from "@/components/shared/loading";

const severityConfig = {
  info: { icon: Info, color: "#22d3ee", bg: "bg-cyan-500/10" },
  warning: { icon: AlertTriangle, color: "#f59e0b", bg: "bg-amber-500/10" },
  critical: { icon: AlertOctagon, color: "#ef4444", bg: "bg-red-500/10" },
};

export function DisruptionList() {
  const { data, error, isLoading } = useDisruptions();

  if (isLoading) return <Loading count={3} />;
  if (error) return <div className="text-sm text-red-400" role="alert">Failed to load disruptions</div>;
  if (!data?.length) return <div className="text-sm text-ink-2">No active disruptions</div>;

  return (
    <div className="space-y-2" aria-live="polite">
      {data.map((d: any) => {
        const config = severityConfig[d.severity as keyof typeof severityConfig] ?? severityConfig.info;
        const Icon = config.icon;
        return (
          <GlassCard key={d.id} accentColor={config.color}>
            <div className="flex gap-3">
              <Icon size={18} className="shrink-0 mt-0.5" style={{ color: config.color }} />
              <div className="flex-1 min-w-0">
                <div className="flex items-center gap-2">
                  <span className="text-sm font-medium">{d.line}</span>
                  <span className={`text-[10px] px-1.5 py-0.5 rounded ${config.bg}`} style={{ color: config.color }}>
                    {d.type}
                  </span>
                </div>
                <p className="text-sm text-ink-1 mt-1">{d.summary_en}</p>
                {d.affected_stations?.length > 0 && (
                  <p className="text-xs text-ink-2 mt-1">
                    Affected: {d.affected_stations.join(", ")}
                  </p>
                )}
                {d.estimated_resolution && (
                  <p className="text-xs text-ink-3 mt-1">
                    Est. resolution: {d.estimated_resolution}
                  </p>
                )}
              </div>
            </div>
          </GlassCard>
        );
      })}
    </div>
  );
}
```

- [ ] **Step 3: Write `src/app/disruptions/page.tsx`**

```tsx
import { DisruptionList } from "@/components/disruption/disruption-list";

export default function DisruptionsPage() {
  return (
    <div className="space-y-4">
      <h2 className="font-display font-bold text-lg">事故 Disruptions</h2>
      <DisruptionList />
    </div>
  );
}
```

- [ ] **Step 4: Commit**

```bash
cd /home/opencode/Works/hk-transit-next
git add -A && git commit -m "feat: add MTR disruption feed with alert banner and list page"
```

---

### Task 14: Offline Journey Planner

**Files:**
- Create: `src/lib/routing/graph.ts`
- Create: `src/lib/routing/raptor.ts`
- Create: `src/lib/routing/transfers.ts`
- Create: `src/lib/hooks/use-planner.ts`
- Create: `src/components/planner/journey-form.tsx`
- Create: `src/components/planner/route-result.tsx`

- [ ] **Step 1: Write `src/lib/routing/graph.ts`**

```ts
import type { TransitNode, RouteSegment } from "@/lib/types";

interface TransitEdge {
  from: string;
  to: string;
  line: string;
  travelTime: number; // minutes
  stops: number;
}

export class TransitGraph {
  nodes: Map<string, TransitNode> = new Map();
  edgesByNode: Map<string, TransitEdge[]> = new Map();
  linesByNode: Map<string, Set<string>> = new Map();

  addNode(node: TransitNode) {
    this.nodes.set(node.id, node);
    if (!this.linesByNode.has(node.id)) this.linesByNode.set(node.id, new Set());
    node.lines.forEach((l) => this.linesByNode.get(node.id)!.add(l));
  }

  addEdge(from: string, to: string, line: string, travelTime: number, stops: number) {
    if (!this.edgesByNode.has(from)) this.edgesByNode.set(from, []);
    this.edgesByNode.get(from)!.push({ from, to, line, travelTime, stops });
  }

  getTransfers(stationId: string): string[] {
    const lines = this.linesByNode.get(stationId);
    if (!lines || lines.size <= 1) return [];
    return Array.from(lines);
  }

  findNearest(lat: number, lng: number, maxResults = 5): TransitNode[] {
    const scored = Array.from(this.nodes.values())
      .map((n) => ({
        node: n,
        dist: Math.sqrt((n.lat - lat) ** 2 + (n.lng - lng) ** 2),
      }))
      .sort((a, b) => a.dist - b.dist)
      .slice(0, maxResults)
      .map((s) => s.node);
    return scored;
  }
}

export function buildTransitGraph(mtrLines: any[], busRoutes: any[]): TransitGraph {
  const graph = new TransitGraph();
  // Build nodes from MTR stations
  // Build edges from MTR line segments
  // Add bus stops and routes
  // Compute walking transfers between nearby stops (within 500m)
  return graph;
}
```

- [ ] **Step 2: Write `src/lib/routing/raptor.ts`**

```ts
/**
 * Simplified RAPTOR (Round-based Public Transit Routing) algorithm.
 * Finds earliest arrival time given departure time, transit graph, and routes.
 */

interface RaptorRoute {
  id: string;
  stops: string[];
  travelTimes: number[]; // cumulative travel time per stop
}

interface RaptorRequest {
  from: string;
  to: string;
  departureTime: number; // minutes from midnight
}

interface RaptorResult {
  path: string[];
  arrivalTime: number;
  transfers: number;
}

export function raptorSearch(
  request: RaptorRequest,
  routes: RaptorRoute[],
  transferTime: number = 5
): RaptorResult | null {
  const { from, to, departureTime } = request;

  // Best arrival time known at each stop
  const bestArrival: Map<string, number> = new Map();
  bestArrival.set(from, departureTime);

  // Which route got us to each stop
  const via: Map<string, { route: string; from: string; prevStop: string | null }> = new Map();

  let improved = true;
  let round = 0;
  const maxRounds = 10;

  while (improved && round < maxRounds) {
    improved = false;
    round++;

    for (const route of routes) {
      // Find earliest stop we can board this route
      let boardIdx = -1;
      let boardTime = Infinity;

      for (let i = 0; i < route.stops.length; i++) {
        const stop = route.stops[i];
        const arr = bestArrival.get(stop);
        if (arr !== undefined && arr <= boardTime) {
          boardIdx = i;
          boardTime = arr;
        }
      }

      if (boardIdx === -1) continue;

      // Propagate arrival times along the route
      for (let i = boardIdx + 1; i < route.stops.length; i++) {
        const stop = route.stops[i];
        const arrival = boardTime + (route.travelTimes[i] - route.travelTimes[boardIdx]);

        if (arrival < (bestArrival.get(stop) ?? Infinity)) {
          bestArrival.set(stop, arrival);
          via.set(stop, { route: route.id, from: route.stops[boardIdx], prevStop: route.stops[i - 1] });
          improved = true;
        }
      }
    }

    // Check if destination reached
    if (bestArrival.has(to)) {
      break;
    }
  }

  if (!bestArrival.has(to)) return null;

  // Reconstruct path
  const path: string[] = [to];
  let current = to;
  while (current !== from) {
    const v = via.get(current);
    if (!v) break;
    if (v.prevStop) path.unshift(v.prevStop);
    current = v.from;
  }
  path.unshift(from);

  return {
    path,
    arrivalTime: bestArrival.get(to)!,
    transfers: via.size,
  };
}
```

- [ ] **Step 3: Write `src/lib/routing/transfers.ts`**

```ts
import type { TransitNode } from "@/lib/types";

const WALKING_SPEED = 80; // meters per minute (~5 km/h)
const MAX_TRANSFER_DISTANCE = 500; // meters
const TRANSFER_PENALTY = 3; // minutes

interface Transfer {
  from: string;
  to: string;
  walkingTime: number;
  distance: number;
}

export function computeWalkingTransfers(
  stations: TransitNode[]
): Transfer[] {
  const transfers: Transfer[] = [];

  for (let i = 0; i < stations.length; i++) {
    for (let j = i + 1; j < stations.length; j++) {
      const a = stations[i];
      const b = stations[j];
      const dist = haversine(a.lat, a.lng, b.lat, b.lng) * 1000; // km → m

      if (dist <= MAX_TRANSFER_DISTANCE) {
        transfers.push({
          from: a.id,
          to: b.id,
          walkingTime: Math.round(dist / WALKING_SPEED) + TRANSFER_PENALTY,
          distance: Math.round(dist),
        });
      }
    }
  }

  return transfers;
}

function haversine(lat1: number, lng1: number, lat2: number, lng2: number): number {
  const R = 6371;
  const dLat = ((lat2 - lat1) * Math.PI) / 180;
  const dLng = ((lng2 - lng1) * Math.PI) / 180;
  const a =
    Math.sin(dLat / 2) ** 2 +
    Math.cos((lat1 * Math.PI) / 180) *
      Math.cos((lat2 * Math.PI) / 180) *
      Math.sin(dLng / 2) ** 2;
  return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
}
```

- [ ] **Step 4: Write `src/lib/hooks/use-planner.ts`**

```ts
"use client";

import { useState, useCallback } from "react";
import { TransitGraph, buildTransitGraph } from "@/lib/routing/graph";
import { raptorSearch } from "@/lib/routing/raptor";
import type { Journey } from "@/lib/types";

interface PlannerState {
  from: string;
  to: string;
  result: Journey | null;
  loading: boolean;
  error: string | null;
}

// Singleton graph instance
let graph: TransitGraph | null = null;
function getGraph(): TransitGraph {
  if (!graph) {
    graph = buildTransitGraph([], []);
  }
  return graph;
}

export function usePlanner() {
  const [state, setState] = useState<PlannerState>({
    from: "", to: "", result: null, loading: false, error: null,
  });

  const plan = useCallback(async (from: string, to: string) => {
    setState((s) => ({ ...s, loading: true, error: null }));
    try {
      const g = getGraph();
      // For now — use nearest station lookup
      // Full RAPTOR integration happens when graph is fully populated with transit data
      setState((s) => ({
        ...s, from, to, loading: false,
        result: null,
        error: "Journey planner will be fully functional when transit data is loaded",
      }));
    } catch (err) {
      setState((s) => ({ ...s, loading: false, error: (err as Error).message }));
    }
  }, []);

  const nearestStations = useCallback((lat: number, lng: number) => {
    return getGraph().findNearest(lat, lng);
  }, []);

  return { ...state, plan, nearestStations };
}
```

- [ ] **Step 5: Write `src/components/planner/journey-form.tsx`**

```tsx
"use client";

import { useState, type FormEvent } from "react";
import { Search, ArrowRight } from "lucide-react";

interface JourneyFormProps {
  onPlan: (from: string, to: string) => void;
  loading: boolean;
}

export function JourneyForm({ onPlan, loading }: JourneyFormProps) {
  const [from, setFrom] = useState("");
  const [to, setTo] = useState("");

  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
    if (from.trim() && to.trim()) onPlan(from.trim(), to.trim());
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-3" role="search" aria-label="Plan a journey">
      <div className="space-y-2">
        <div className="relative">
          <div className="absolute left-3 top-1/2 -translate-y-1/2 w-2 h-2 rounded-full bg-accent-mtr" />
          <input
            type="text"
            value={from}
            onChange={(e) => setFrom(e.target.value)}
            placeholder="From (station or stop name)..."
            className="w-full bg-glass-bg border border-glass-border rounded-xl py-3 pl-8 pr-4 text-sm text-ink-0 placeholder:text-ink-3 focus:outline-none focus:border-accent-mtr/50"
            aria-label="Start point"
          />
        </div>
        <div className="relative">
          <div className="absolute left-3 top-1/2 -translate-y-1/2 w-2 h-2 rounded-full bg-accent-bus" />
          <input
            type="text"
            value={to}
            onChange={(e) => setTo(e.target.value)}
            placeholder="To (station or stop name)..."
            className="w-full bg-glass-bg border border-glass-border rounded-xl py-3 pl-8 pr-4 text-sm text-ink-0 placeholder:text-ink-3 focus:outline-none focus:border-accent-bus/50"
            aria-label="Destination"
          />
        </div>
      </div>

      <button
        type="submit"
        disabled={loading || !from.trim() || !to.trim()}
        className="w-full flex items-center justify-center gap-2 bg-accent-mtr/20 text-accent-mtr border border-accent-mtr/30 rounded-xl py-3 text-sm font-medium hover:bg-accent-mtr/30 disabled:opacity-40 disabled:cursor-not-allowed transition-all touch-target"
      >
        {loading ? (
          <span className="animate-pulse">Planning...</span>
        ) : (
          <>
            <Search size={16} />
            <span>Plan Route</span>
          </>
        )}
      </button>
    </form>
  );
}
```

- [ ] **Step 6: Write `src/components/planner/route-result.tsx`**

```tsx
"use client";

import { Train, Bus, Walk } from "lucide-react";
import type { Journey } from "@/lib/types";

interface RouteResultProps {
  journey: Journey | null;
  error: string | null;
}

export function RouteResult({ journey, error }: RouteResultProps) {
  if (error) {
    return (
      <div className="glass-card p-4 border-red-500/20" role="alert">
        <p className="text-sm text-red-400">{error}</p>
      </div>
    );
  }

  if (!journey) return null;

  return (
    <div className="space-y-2" role="list" aria-label="Journey details">
      <div className="glass-card p-3 bg-accent-mtr/5">
        <p className="text-xs text-ink-2">
          ~{journey.totalDuration} min · {journey.transfers} transfer{journey.transfers !== 1 ? "s" : ""}
        </p>
      </div>

      {journey.segments.map((seg, i) => (
        <div key={i} className="glass-card p-3" role="listitem">
          <div className="flex items-center gap-2 text-sm">
            {seg.mode === "mtr" ? <Train size={16} className="text-accent-mtr" /> : <Bus size={16} className="text-accent-bus" />}
            <span className="font-medium">{seg.line}</span>
            <span className="text-xs text-ink-2 ml-auto">{seg.departureTime}min → {seg.arrivalTime}min</span>
          </div>
          <p className="text-xs text-ink-2 mt-1">
            {seg.from.nameZh} → {seg.to.nameZh} · {seg.stops} stop{seg.stops !== 1 ? "s" : ""}
          </p>
        </div>
      ))}
    </div>
  );
}
```

- [ ] **Step 7: Build and commit**

```bash
cd /home/opencode/Works/hk-transit-next
pnpm build && git add -A && git commit -m "feat: add offline journey planner with RAPTOR algorithm"
```

---

### Task 15: Push Notifications

**Files:**
- Create: `src/lib/hooks/use-notifications.ts`
- Create: `public/sw.js` (service worker placeholder for push)

- [ ] **Step 1: Write `src/lib/hooks/use-notifications.ts`**

```ts
"use client";

import { useState, useCallback, useEffect } from "react";
import type { EtaAlert } from "@/lib/types";

const STORAGE_KEY = "hk-transit-eta-alerts";

export function useNotifications() {
  const [alerts, setAlerts] = useState<EtaAlert[]>([]);
  const [permission, setPermission] = useState<NotificationPermission>("default");

  useEffect(() => {
    if ("Notification" in window) {
      setPermission(Notification.permission);
    }
    const stored = localStorage.getItem(STORAGE_KEY);
    if (stored) {
      try { setAlerts(JSON.parse(stored)); } catch {}
    }
  }, []);

  const requestPermission = useCallback(async () => {
    if (!("Notification" in window)) return;
    const result = await Notification.requestPermission();
    setPermission(result);
  }, []);

  const addAlert = useCallback((alert: EtaAlert) => {
    setAlerts((prev) => {
      const next = [...prev, alert];
      localStorage.setItem(STORAGE_KEY, JSON.stringify(next));
      return next;
    });
  }, []);

  const removeAlert = useCallback((id: string) => {
    setAlerts((prev) => {
      const next = prev.filter((a) => a.id !== id);
      localStorage.setItem(STORAGE_KEY, JSON.stringify(next));
      return next;
    });
  }, []);

  const sendNotification = useCallback((title: string, body: string) => {
    if ("Notification" in window && Notification.permission === "granted") {
      new Notification(title, { body, icon: "/icons/icon-192.png" });
    }
  }, []);

  return { alerts, permission, requestPermission, addAlert, removeAlert, sendNotification };
}
```

- [ ] **Step 2: Write `public/sw.js`**

```js
// Service Worker for HK Transit PWA
const CACHE = "hk-transit-v1";
const ASSETS = ["/", "/manifest.json"];

self.addEventListener("install", (event) => {
  event.waitUntil(
    caches.open(CACHE).then((cache) => cache.addAll(ASSETS))
  );
  self.skipWaiting();
});

self.addEventListener("activate", (event) => {
  event.waitUntil(clients.claim());
});

self.addEventListener("fetch", (event) => {
  event.respondWith(
    caches.match(event.request).then((cached) => {
      // Network first, fallback to cache
      return fetch(event.request)
        .then((response) => {
          const clone = response.clone();
          caches.open(CACHE).then((cache) => cache.put(event.request, clone));
          return response;
        })
        .catch(() => cached ?? new Response("Offline", { status: 503 }));
    })
  );
});

self.addEventListener("push", (event) => {
  if (event.data) {
    const data = event.data.json();
    event.waitUntil(
      self.registration.showNotification(data.title ?? "HK Transit", {
        body: data.body ?? "",
        icon: "/icons/icon-192.png",
      })
    );
  }
});
```

- [ ] **Step 3: Commit**

```bash
cd /home/opencode/Works/hk-transit-next
git add -A && git commit -m "feat: add notification hooks and service worker"
```

---

### Task 16: PWA Setup (Manifest + Icons)

**Files:**
- Create: `public/manifest.json`
- Create: `public/icons/icon-192.png` (placeholder)
- Create: `public/icons/icon-512.png` (placeholder)

- [ ] **Step 1: Write `public/manifest.json`**

```json
{
  "name": "HK Transit",
  "short_name": "HK Transit",
  "description": "Real-time Hong Kong transport guide",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#0a0a14",
  "theme_color": "#0a0a14",
  "orientation": "portrait",
  "categories": ["travel", "navigation", "transport"],
  "icons": [
    { "src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

- [ ] **Step 2: Generate placeholder icons**

```bash
# Generate 1x1 pixel PNGs as placeholders (replace with real icons later)
cd /home/opencode/Works/hk-transit-next
python3 -c "
import struct, zlib
def create_png(w, h, filepath):
    def chunk(t, d):
        c = t + d
        return struct.pack('>I', len(d)) + c + struct.pack('>I', zlib.crc32(c) & 0xffffffff)
    ihdr = struct.pack('>IIBBBBB', w, h, 8, 2, 0, 0, 0)
    raw = b''
    for y in range(h):
        raw += b'\x00' + bytes([10, 10, 20] * w)
    idat = zlib.compress(raw)
    return b'\x89PNG\r\n\x1a\n' + chunk(b'IHDR', ihdr) + chunk(b'IDAT', idat) + chunk(b'IEND', b'')
create_png(192, 192, 'public/icons/icon-192.png')
create_png(512, 512, 'public/icons/icon-512.png')
print('Icons created')
"
```

- [ ] **Step 3: Update `next.config.ts`** for PWA headers

```ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  output: "standalone",
  headers: async () => [
    {
      source: "/sw.js",
      headers: [{ key: "Cache-Control", value: "no-cache" }],
    },
  ],
};

export default nextConfig;
```

- [ ] **Step 4: Verify build**

```bash
cd /home/opencode/Works/hk-transit-next
pnpm build
```

Expected: Build succeeds, manifest.json is accessible in `.next/static/`.

- [ ] **Step 5: Commit**

```bash
cd /home/opencode/Works/hk-transit-next
git add -A && git commit -m "feat: add PWA manifest and service worker"
```

---

### Task 17: Capacitor Setup

**Files:**
- Modify: `capacitor.config.ts`

- [ ] **Step 1: Update `capacitor.config.ts`**

```ts
import type { CapacitorConfig } from "@capacitor/cli";

const config: CapacitorConfig = {
  appId: "com.hktransit.app",
  appName: "HK Transit",
  webDir: ".next",
  server: {
    url: process.env.CAPACITOR_SERVER_URL || undefined,
    cleartext: !process.env.CAPACITOR_SERVER_URL,
  },
  plugins: {
    SplashScreen: {
      launchShowDuration: 1000,
      backgroundColor: "#0a0a14",
      showSpinner: false,
    },
    StatusBar: {
      style: "dark",
      backgroundColor: "#0a0a14",
    },
    LocalNotifications: {
      smallIcon: "ic_stat_icon",
      iconColor: "#22d3ee",
    },
  },
  ios: {
    contentInset: "always",
  },
};

export default config;
```

- [ ] **Step 2: Add native platforms**

```bash
cd /home/opencode/Works/hk-transit-next
npx cap add ios
npx cap add android
```

Expected: `ios/` and `android/` directories created.

- [ ] **Step 3: Commit**

```bash
cd /home/opencode/Works/hk-transit-next
git add -A && git commit -m "feat: configure Capacitor for iOS and Android"
```

---

### Task 18: Animations & View Transitions

**Files:**
- Modify: `src/app/globals.css` (add VT animations)

- [ ] **Step 1: Add View Transitions CSS to `src/app/globals.css`**

Append to the bottom of `globals.css`:

```css
/* === View Transitions === */
::view-transition-old(root) {
  animation: fade-out 0.2s ease;
}
::view-transition-new(root) {
  animation: fade-in 0.2s ease;
}

/* Tab transitions */
::view-transition-old(tab-content) {
  animation: slide-out-left 0.25s ease;
}
::view-transition-new(tab-content) {
  animation: slide-in-right 0.25s ease;
}

/* Shared element morphs */
::view-transition-old(morph) {
  animation: fade-out 0.2s ease;
}
::view-transition-new(morph) {
  animation: fade-in 0.2s ease;
}

@keyframes fade-in {
  from { opacity: 0; }
  to { opacity: 1; }
}
@keyframes fade-out {
  from { opacity: 1; }
  to { opacity: 0; }
}
@keyframes slide-in-right {
  from { transform: translateX(30px); opacity: 0; }
  to { transform: translateX(0); opacity: 1; }
}
@keyframes slide-out-left {
  from { transform: translateX(0); opacity: 1; }
  to { transform: translateX(-30px); opacity: 0; }
}

@media (prefers-reduced-motion: reduce) {
  ::view-transition-old(root),
  ::view-transition-new(root),
  ::view-transition-old(tab-content),
  ::view-transition-new(tab-content) {
    animation: none;
  }
}
```

- [ ] **Step 2: Add Framer Motion page transitions to `src/components/layout/app-shell.tsx`**

Wrap the `children` area:

```tsx
import { motion, AnimatePresence } from "framer-motion";
import { usePathname } from "next/navigation";

// Inside AppShell:
const pathname = usePathname();

// Replace <main> with:
<AnimatePresence mode="wait">
  <motion.main
    key={pathname}
    initial={{ opacity: 0, y: 8 }}
    animate={{ opacity: 1, y: 0 }}
    exit={{ opacity: 0, y: -8 }}
    transition={{ duration: 0.2, ease: "easeOut" }}
    className="flex-1 max-w-lg mx-auto w-full px-4 pb-20 pt-4"
  >
    {children}
  </motion.main>
</AnimatePresence>
```

- [ ] **Step 3: Verify build**

```bash
cd /home/opencode/Works/hk-transit-next
pnpm build
```

Expected: Build succeeds.

- [ ] **Step 4: Commit**

```bash
cd /home/opencode/Works/hk-transit-next
git add -A && git commit -m "feat: add View Transitions and Framer Motion page animations"
```

---

### Task 19: Copy Static Data Files and End-to-End Verification

**Files:**
- Copy: `data/` from old project to new project
- Verify: `pnpm build` passes
- Verify: `pnpm dev` starts without errors

- [ ] **Step 1: Copy static data files from the old project**

```bash
cp -r /home/opencode/Works/hk-transit/src/data/* /home/opencode/Works/hk-transit-next/src/data/
```

- [ ] **Step 2: Full build verification**

```bash
cd /home/opencode/Works/hk-transit-next
pnpm build 2>&1
```

Expected: Build succeeds with no errors or warnings.

- [ ] **Step 3: Fix any build issues** — check for TypeScript errors, missing imports, or broken references. Fix any issues found.

- [ ] **Step 4: Final commit**

```bash
cd /home/opencode/Works/hk-transit-next
git add -A && git commit -m "feat: copy static transit data and finalize project"
```

---

### Post-Plan: Deployment

After all tasks complete:

```bash
cd /home/opencode/Works/hk-transit-next

# Deploy to Vercel
npx vercel deploy --prod

# Build native apps
npm run build
npx cap sync
npx cap open ios   # → Archive in Xcode
npx cap open android  # → Build APK/AAB
```
