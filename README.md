# NIRA-INTEL

**Uganda Civil Registration Intelligence Dashboard**

A geospatial intelligence platform for the National Identification and Registration Authority (NIRA) of Uganda. Monitors NID coverage, birth, death, and marriage registration rates across all 57 districts — identifying underserved communities, forecasting mobile team deployments, and tracking CRVS performance in real time.

> **Prototype** — submitted to the Uganda Ministry of ICT & National Guidance Government Systems Prototype Showcase (Ref: MoICT&NG/DIEG/2025-26/001-AF, deadline 1 June 2026).

---

## The Problem

Uganda's civil registration reporting cycle is quarterly: district offices compile paper or Excel reports, submit to regional headquarters, which forward to NIRA HQ. By the time leadership sees the data, coverage trends have shifted and mobile team deployments are reactive rather than forecasted.

- **41.1%** national NID coverage — 27.7 million adults without a National ID
- **34.2%** birth registration nationally; Karamoja sub-region below 13%
- **18 districts** in critical status (NID < 40%) with no real-time monitoring tool

Every unregistered citizen is invisible to health, education, financial inclusion, and social protection systems.

---

## What NIRA-INTEL Does

| Feature | Description |
|---|---|
| **District choropleth map** | 57 districts colour-coded red→amber→green by registration coverage. Circles scale with population. |
| **Four coverage layers** | Switch between National ID (adults 18+), Birth, Death, and Marriage registration — one layer active at a time. |
| **District Intel Panel** | Click any district: all 4 coverage metrics, trend direction, last mobile drive, people-per-centre ratio, and an algorithmically generated recommended action. |
| **Priority ranking engine** | Scores districts by `(100 − NID%) × log(population) ÷ centres` to surface the highest-impact mobile team deployment targets. |
| **NIRA alerts feed** | Structured critical/warning/info alerts — coverage drops, disease outbreaks affecting centres, active mobile drives. |
| **Disease alerts layer** | WHO AFRO Uganda outbreak locations overlaid on the map. |
| **Refugee settlements layer** | All 9 major UNHCR settlements in Uganda with population figures (Bidi Bidi, Nakivale, Rhino Camp, and 6 others). |
| **Uganda-scoped search** | Instant district lookup + Nominatim geocoding bounded to Uganda's geographic extent. |
| **Mobile responsive** | Full bottom-nav drawer UI for field officers on smartphones. Tapping a district auto-opens the intel panel. |

---

## Screenshots

> _Map view showing all 57 Uganda districts colour-coded by NID coverage, with the district intel panel open for Kaabong (21% — Critical)._

---

## Tech Stack

| Layer | Technology | Version |
|---|---|---|
| Framework | Next.js App Router | 16.2.6 |
| UI | React | 19.2.4 |
| Language | TypeScript | 5.x |
| Styling | Tailwind CSS | 4.x |
| Map engine | MapLibre GL JS | 5.24.0 |
| Map React bindings | react-map-gl | 8.1.1 |
| Animations | Framer Motion | 12.38.0 |
| Icons | Lucide React | 1.14.0 |
| Hosting | Vercel Edge Network | — |

All dependencies are open-source (MIT / Apache 2.0). No proprietary APIs are used in the prototype.

---

## Project Structure

```
nira-intel/
├── src/
│   ├── app/
│   │   ├── page.tsx                  # Main dashboard — state, layout, mobile nav
│   │   ├── layout.tsx                # Root layout, metadata, JSON-LD
│   │   ├── globals.css               # CSS custom properties, light theme, HUD classes
│   │   └── api/
│   │       ├── registration/         # GET /api/registration?layer={nid|birth|death|marriage}
│   │       ├── alerts-nira/          # GET /api/alerts-nira
│   │       └── region-dossier/       # GET /api/region-dossier?lat=X&lng=Y
│   └── components/
│       ├── OsirisMap.tsx             # MapLibre GL map, all sources and layers
│       ├── DistrictIntelPanel.tsx    # District detail slide-in panel
│       ├── IntelFeed.tsx             # Alerts + priorities tabs
│       ├── LiveAlerts.tsx            # Filterable NIRA alerts feed
│       ├── LayerPanel.tsx            # Layer toggle controls + coverage legend
│       ├── SearchBar.tsx             # District + geocode search (Uganda-scoped)
│       ├── GlobalStatusBar.tsx       # Scrolling NIRA metrics ticker
│       ├── SharePanel.tsx            # Map state share/permalink
│       ├── ScaleBar.tsx              # Map scale indicator
│       ├── ViewPresets.tsx           # Uganda region quick-navigate presets
│       ├── KeyboardShortcuts.tsx     # Keyboard shortcut reference
│       └── ErrorBoundary.tsx         # React error boundary wrapper
├── public/
│   ├── data/
│   │   └── nira-coverage.json        # 57-district CRVS dataset
│   └── [favicon suite]               # logo.jpg + 16/32/48/192/512px PNGs + ICO
└── docs/
    └── system-description.md         # MoICT showcase system description (5 pages)
```

---

## Data

All data in the prototype is sourced from publicly available reports:

| Data | Source |
|---|---|
| Population by district | UBOS National Population and Housing Census 2024 |
| NID coverage | NIRA Annual Report 2022/23 (disaggregated from regional totals) |
| Birth / death / marriage rates | UBOS Vital Statistics Report 2022 |
| Refugee settlements | UNHCR Uganda — 9 major settlements (hardcoded) |
| Disease alerts | WHO AFRO Uganda bulletins (prototype snapshot) |

`public/data/nira-coverage.json` contains 57 district records. Each record:

```json
{
  "district_id": "UGA001",
  "name": "Kampala",
  "region": "Central",
  "population": 1680000,
  "nid_coverage_pct": 70.0,
  "birth_registration_pct": 68.2,
  "death_registration_pct": 51.0,
  "marriage_registration_pct": 22.0,
  "registration_centres": 14,
  "lat": 0.3476,
  "lon": 32.5825,
  "status": "on_track",
  "trend": "stable",
  "mobile_teams_needed": 0,
  "last_drive": "2024-10"
}
```

---

## API Reference

### `GET /api/registration`

Returns district coverage data for a given registration type.

**Query parameters:**

| Parameter | Values | Default |
|---|---|---|
| `layer` | `nid` \| `birth` \| `death` \| `marriage` | `nid` |
| `region` | Any Uganda region name or `all` | `all` |

**Response:**

```json
{
  "layer": "nid",
  "districts": [ /* 57 district objects */ ],
  "summary": {
    "total_districts": 57,
    "critical": 18,
    "needs_attention": 24,
    "on_track": 15,
    "national_average": 41.1,
    "total_population": 47123531,
    "total_registered": 19341368
  },
  "priorities": [ /* top 10 districts ranked by priority score */ ]
}
```

Cache: `public, s-maxage=3600, stale-while-revalidate=7200`

---

### `GET /api/alerts-nira`

Returns the structured NIRA alert feed.

**Response:**

```json
{
  "alerts": [
    {
      "id": "ALT001",
      "type": "critical",
      "category": "coverage_drop",
      "title": "Kaabong NID coverage fell below 20%",
      "district": "Kaabong",
      "region": "Karamoja",
      "lat": 3.52,
      "lon": 34.14,
      "timestamp": "2024-11-15T08:00:00Z",
      "source": "NIRA District Report",
      "summary": "...",
      "action": "Deploy mobile team — Priority Level 5"
    }
  ]
}
```

Alert types: `critical` | `warning` | `info`
Alert categories: `coverage_drop` | `disease_outbreak` | `mobile_drive` | `policy`

---

## Priority Ranking Formula

Districts are scored to identify the highest-impact targets for mobile team deployment:

```
Priority Score = (100 − NID_coverage%) × log(population) ÷ registration_centres
```

This surfaces large, underserved districts with few centres — where deploying a single mobile team reaches the most unregistered people. The top 15 non-on-track districts are shown in the INTEL → PRIORITIES tab.

---

## Coverage Colour Scale

| Colour | NID Coverage | Status |
|---|---|---|
| 🔴 Dark red `#7f1d1d` | < 20% | Crisis |
| 🔴 Red `#dc2626` | 20–30% | Critical |
| 🟠 Orange `#f97316` | 30–40% | Critical |
| 🟡 Amber `#eab308` | 40–55% | Needs Attention |
| 🟢 Light green `#22c55e` | 55–70% | Needs Attention |
| 🟢 Dark green `#15803d` | ≥ 70% | On Track |

---

## Getting Started

### Prerequisites

- Node.js 20+
- npm or bun

### Install and run

```bash
git clone git@github.com:khanblair/nira-intel.git
cd nira-intel
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

### Build for production

```bash
npm run build
npm start
```

### Lint

```bash
npm run lint
```

### Deploy to Vercel

The project is configured for zero-config Vercel deployment. Connect the repo at [vercel.com](https://vercel.com), select the `main` branch, and deploy. No environment variables are required for the prototype.

---

## Roadmap

| Phase | Timeline | Description |
|---|---|---|
| **Prototype** ✅ | May 2026 | Static dataset, full UI, public Vercel deployment |
| **Integration** | Q3 2026 | Live NIRA database connection via read-replica |
| **Authentication** | Q3 2026 | NITA-U SSO / OAuth2 — role-based access for HQ, regional, and district staff |
| **Alerts automation** | Q4 2026 | WHO AFRO RSS parser as scheduled Edge Function; NIRA webhook |
| **PWA / Offline** | Q1 2027 | Service worker + offline mode for field officers in low-connectivity areas |
| **On-premise** | Q1 2027 | Docker packaging for NITA-U National Data Centre deployment |

---

## Alignment with Government Priorities

| Priority | Alignment |
|---|---|
| NDP III — Digital Transformation | Converts quarterly paper reports into a real-time intelligence layer |
| Vision 2040 — Universal Registration | Directly targets the 59% without NID by identifying the highest-priority underserved districts |
| Uganda Data Protection Act 2019 | Prototype handles zero PII — all data is district-level aggregates only |
| NITA-U Interoperability Framework | RESTful API layer designed for integration with the National e-Government Infrastructure |
| Government Open Data Initiative | Built entirely on open-source tools and free public data |

---

## Intellectual Property

NIRA-INTEL is adapted from the open-source [Osiris](https://github.com/khanblair/osiris) geospatial intelligence platform. The NIRA application layer — all Uganda data models, API routes, district coverage logic, priority ranking engine, intelligence panels, and alerts feed — is original work.

All intellectual property remains vested in the applicant. Submission to the Government Systems Prototype Showcase does not transfer any rights to Government.

---

## Licence

This project is open source. The application code is released under the [MIT Licence](LICENSE). The NIRA Annual Report data and UBOS Census data remain the property of their respective owners and are used here solely for prototype demonstration purposes.

---

## Contact

**Project:** NIRA-INTEL — Uganda Civil Registration Intelligence Dashboard
**Repository:** [github.com/khanblair/nira-intel](https://github.com/khanblair/nira-intel)
**Submitted to:** Ministry of ICT & National Guidance, Uganda (Ref: MoICT&NG/DIEG/2025-26/001-AF)
