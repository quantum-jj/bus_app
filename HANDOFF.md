# Bus Checker App — Handoff Summary

**App:** Single-file PWA at `bus_app/index.html`
**Current version:** `v2.2.0`
**Hosted at:** `https://quantum-jj.github.io/bus_app`

---

## What the app is

A lightweight London bus arrival checker. Users save **favourite locations** (addresses, landmarks, postcodes via Photon/OSM search), then pick an **origin and destination** to look up live bus arrivals from nearby stops — no saved routes, everything is live.

Positioned as a quick daily commute companion, not a replacement for Citymapper or Google Maps.

---

## CLAUDE.md rules (must follow)
- Increment **minor version (Y)** for every non-bug-fix change. Small tweaks do not bump.
- Bug fixes do **not** bump the version.
- **Do not offer to push to git** — user handles all commits/pushes.
- After changes, run `node --check` on extracted JS.

---

## Architecture (v2.2.0)

### Data Model
```js
config = {
  locations: [
    { id: 'loc_home', name: 'Home', lat: null, lon: null, address: '', emoji: '' },
    { id: 'loc_work', name: 'Work', lat: null, lon: null, address: '', emoji: '' },
    // user can add unlimited custom locations, each with optional emoji
  ],
  weatherLocation: 'hoxton',   // fallback if Home not set
  tflKey: '',
  locationEnabled: false,
  onboardingSeen: false,
  dataVersion: 1,              // DATA_VERSION — for migration checks on updates
  lastFromId: null,            // remember last origin selection
  lastToId: null,              // remember last destination selection
}
```

Storage key: `bus_config_v2` (v1 data is auto-cleared on first load with notice).

### Screens
1. **Onboarding** — shown once for first-time users
2. **Home** — weather widget + route picker (from/to selectors) + route results
3. **Settings** — location management, weather, TfL key, update check, reset
4. **Location Editor** — edit name, search address via Photon, delete

### User Flow
1. Save favourite locations (at least set Home and Work via Photon address search)
2. On home screen, tap origin selector → pick from: saved locations / current GPS / ad-hoc search
3. Tap destination selector → same options
4. Tap search button → app finds bus stops near origin AND destination (configurable radius)
5. Builds set of line IDs from destination stops, then fetches arrivals at origin stops
6. Only shows lines that serve **both** origin and destination stops (with direction sanity check)
7. Results shown as stop-grouped cards with bus badges, destinations, arrival chips, walk times
8. Auto-refreshes every 30 seconds

---

## All features

### Core
- TfL Unified API: `/StopPoint?lat=X&lon=Y` for nearby stops, `/StopPoint/{id}/Arrivals` for live arrivals
- **Dual-endpoint line matching**: finds stops near both origin AND destination, then cross-references which bus lines serve both. Compass-based direction filtering (within 120°) used as secondary sanity check
- Configurable walking radius (300m–1.5km) in Settings, applied to both origin and destination
- Walk time: haversine × 1.3 detour factor ÷ 80 m/min
- Auto-refresh every 30s, manual refresh button
- Results grouped by stop, each showing multiple lines with up to 3 arrival chips each

### Search (Photon + postcodes.io)
- `searchPhoton(query)` calls `photon.komoot.io` with London bias and bbox filtering
- Results filtered to Greater London bounding box (lat 51.28–51.70, lon -0.52–0.34)
- UK postcodes auto-detected via regex → resolved via `postcodes.io` for reliable results
- Improved address formatting: street numbers, districts, postcodes shown clearly
- 300ms debounce on all search inputs
- Used in: location picker modal (ad-hoc search), location editor

### Location Picker Modal
- Bottom-sheet modal with search box, GPS option, and saved location list
- "Current location" triggers device geolocation
- Ad-hoc search via Photon (one-time, not saved)
- Disables the already-selected opposite-direction location

### Onboarding
- First-time welcome screen: icon, title, feature list, "Get Started" button
- `config.onboardingSeen` flag — shown once
- Introduces app as lightweight daily commute checker

### Weather Widget
- Open-Meteo API for current conditions + AQI
- Uses Home location coords if set, else falls back to `weatherLocation` setting
- Weather alerts for extreme cold/heat/wind/storms
- Tap → opens BBC Weather

### PWA Update Mechanism
- `checkForUpdates()` fetches own URL with cache-bust, parses `APP_VERSION` from HTML
- Checks on init (after 5s delay) then every 4 hours
- Shows unobtrusive toast notification with "Update" button
- `DATA_VERSION` constant — if it changes between versions, data is cleared with notice
- Settings has manual "Check for Updates" button

### Settings
- **Saved Locations**: list with tap-to-edit, add new, editor with Photon search
- **Weather Location**: auto from Home, or manual dropdown if Home not set
- **TfL API Key**: optional, raises rate limit
- **Check for Updates**: manual trigger
- **Reset All**: clears localStorage and all selections

---

## Key functions reference

| Function | Purpose |
|---|---|
| `searchPhoton(query)` | Photon geocoding + postcodes.io fallback (London-filtered) |
| `searchPostcodesIo(query)` | UK postcode geocoding via postcodes.io |
| `isPostcode(query)` | Regex check for UK postcode format |
| `isInLondon(lat, lon)` | Greater London bounding box check |
| `setupPhotonInput(input, results, onSelect)` | Wires up autocomplete on any input element |
| `openLocationPicker(mode)` | Opens modal for from/to selection |
| `pickLocation(sel)` | Confirms a location selection and updates state |
| `lookupRoutes()` | Main route lookup: nearby stops → arrivals → filter → render |
| `findNearbyStops(lat, lon, radius)` | TfL StopPoint proximity search |
| `renderRouteResults(stopResults)` | Renders stop-grouped result cards |
| `calculateBearing(lat1, lon1, lat2, lon2)` | Compass bearing in degrees |
| `haversineMetres(lat1, lon1, lat2, lon2)` | Straight-line distance |
| `calculateWalkTime(lat1, lon1, lat2, lon2)` | Walk time estimate in minutes |
| `fetchArrivals(stopId)` | TfL arrivals with 30s cache |
| `checkForUpdates(manual?)` | PWA version check |
| `loadWeather()` | Weather + AQI fetch |

---

## What was removed in v2.0.0
- Groups system (groups, favIds, fromPostcode/toPostcode)
- Saved routes/favourites (routes are now live-only)
- Wizard modal (replaced by location picker + live lookup)
- Night bus merging (all lines shown naturally in results)
- Drag-to-reorder (no persistent order to manage)
- Hub/cluster stop resolution (now using direct proximity search)
- Stop name enrichment, walk time enrichment (calculated live)
- Collapse/expand for cards and groups
- postcodes.io geocoding (replaced by Photon)

---

## Last task completed
v2.2.0: Rewrote route filtering — now finds stops near BOTH origin and destination, cross-references line IDs to only show buses that actually connect the two locations. Added configurable walking distance radius (300m–1.5km) in Settings. Compass-direction filtering demoted to secondary sanity check.
