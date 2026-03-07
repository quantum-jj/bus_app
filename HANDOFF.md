# Bus App — Project Handoff

**Current version:** `v2.6.0`
**File:** `bus_app/index.html` (~1855 lines, single-file PWA — no build step)
**Stack:** Vanilla HTML + inline CSS + vanilla JS, no dependencies, no npm

---

## What This App Is

A London-only PWA for checking live TfL bus arrivals between two locations. Works exclusively with the TfL public bus network. Designed as a mobile-first daily commute tool, not a full journey planner.

---

## Architecture

Single self-contained `index.html`. All CSS, HTML, and JS are inline. Deployed as a static file (GitHub Pages or similar). The PWA manifest is inlined as a base64 `data:` URI in `<link rel="manifest">`.

**localStorage key:** `bus_config_v2`
**DATA_VERSION:** `1` (checked on load; mismatch wipes config and alerts user)
**No service worker** currently.

---

## Config Schema (localStorage `bus_config_v2`)

```js
{
  locations: [
    { id: string, name: string, lat: number|null, lon: number|null, address: string, emoji: string }
  ],
  weatherLocation: string,   // one of WEATHER_LOCATIONS ids
  tflKey: string,            // optional TfL API key
  locationEnabled: bool,
  onboardingSeen: bool,
  dataVersion: 1,
  lastFromId: string|null,   // persists last-used from/to selections
  lastToId: string|null,
  walkingRadius: number      // metres, default 500
}
```

Default locations: `loc_home` (Home, 🏠) and `loc_work` (Work, 💼). User-created locations get IDs like `loc_1234567890`.

---

## Key APIs Used

| API | Purpose | Auth |
|-----|---------|------|
| `api.tfl.gov.uk` | Live bus arrivals, stop data, route sequences | Optional free key — 50 req/min without, 500/min with |
| `photon.komoot.io` | Address search (OSM-based), London bbox filtered | None |
| `api.postcodes.io` | UK postcode lookup fallback | None |
| `api.open-meteo.com` | Weather (current conditions) | None |
| `air-quality-api.open-meteo.com` | AQI (European index) | None |

TfL key stored in `config.tflKey`, appended as `?app_key=` or `&app_key=`.

**Important:** The TfL API (and WebFetch for TfL domains) is unreachable from the Claude VM (`getaddrinfo EAI_AGAIN`). All route logic must be tested via the user's browser console. Use `console.log` with `[BusChecker]` prefix for debug output.

---

## Route Lookup Logic (`lookupRoutes`, ~line 908)

Three-phase process — the most complex part of the app:

**Phase A — Gather arrivals**
- `findNearbyStops(lat, lon, radius)` → `GET /StopPoint?lat=&lon=&stopTypes=NaptanPublicBusCoachTram&radius=`
- Run for both origin AND destination in parallel
- Build `destLineIds` set from destination stops (3 sources: `stop.lines[]`, `stop.lineModeGroups[]`, arrivals fallback if <3 lines found)
- Fetch arrivals for up to 10 origin stops in parallel; group by lineId

**Phase B — Direction validation via Route Sequence** *(the key filter)*
- For each matched line (appears at both origin and destination), call `GET /Line/{id}/Route/Sequence/inbound` AND `outbound`
- For each direction's ordered stop sequence: check if any origin stop index < any destination stop index
- Record `validStops[lineId]` = Set of origin stopIds on the correct direction
- Fallback: if route sequence API fails, use compass bearing (reject if diff >150°)

**Phase C — Build and render results**
- Only include arrivals where stop is in `validStops[lineId]`
- Sort stops by walk time; lines within each stop by soonest arrival
- Auto-refresh every 30 seconds (clears arrivals cache)

**Walk time formula:** `haversineMetres × 1.3 ÷ 80 m/min`, minimum 1 minute

---

## Screen Structure

```
navStack: string[]  — push/pop navigation, no router

Screens (`.screen.active`):
  onboardingScreen      → first visit only
  homeScreen            → weather widget + route picker + results
  settingsScreen        → renderSettings()
  locationEditorScreen  → renderLocationEditor(locId)
```

The location picker is a **modal overlay** (`#locPickerModal`), not a screen.

---

## Location Picker Modal (mobile keyboard fix)

The search input (`#pickerSearchInput`) is in the **modal header**, NOT the scrollable body. This prevents iOS keyboard from pushing the search bar off-screen. When the keyboard opens, `visualViewport` resize events detect the height shrink (>25%); the sheet gains `.keyboard-open` (removes rounded corners, sets `maxHeight` to visual viewport height − 8px). A `MutationObserver` cleans up the listener when the modal closes.

---

## Address Search

- Debounced 300ms
- If query matches UK postcode regex → `postcodes.io` first
- Otherwise → Photon with London bbox (`-0.52,51.28,0.34,51.70`) and `lat=51.505&lon=-0.09` bias
- All results filtered to Greater London lat/lon bounds
- Max 5 results shown

---

## Emoji Handling (Location Editor)

- Emoji stored separately from `loc.name` — never combined in the data model
- Display: `(loc.emoji || defaultEmoji) + ' ' + loc.name` at render time only
- **Validation:** `Intl.Segmenter` grapheme cluster count must equal 1, and must match `\p{Emoji_Presentation}`
- **Dropdown:** 20 common emoji shown on tap of the emoji input (which is `readonly`); single `capture: true, once: true` listener closes it on outside tap
- **Name save:** emoji stripped before saving via `[\p{Emoji_Presentation}\p{Emoji}...]` regex; saves on blur OR Enter key
- Defaults: Home → 🏠, Work → 💼, new locations → ⭐

---

## Settings Structure

Three sections; sections 2 and 3 hidden behind "More settings ▾" accordion (`max-height` CSS transition, state in `window._settingsAdvancedOpen`):

1. **Saved Locations** — draggable list (mouse + touch), add button, search radius selector
2. **API Settings** *(advanced)* — status cards for all 4 APIs + TfL key input + weather location
3. **App** *(advanced)* — Check for updates, View on GitHub, Reset, version label

**Drag-and-drop:** HTML5 drag API for mouse, touchstart/move/end with ghost clone for touch. Reorder syncs to `config.locations` immediately on drop.

---

## Versioning Rules (CLAUDE.md)

- `const APP_VERSION = 'vX.Y.Z'` at ~line 393 of `index.html`
- **Bump minor (Y)** for features, UI changes, behaviour changes
- **Do NOT bump** for bug fixes
- **Do not offer to push to git** — user handles all commits

---

## Key Function Reference

| Function | ~Line | Purpose |
|----------|-------|---------|
| `loadConfig()` | 447 | Load + migrate config from localStorage |
| `saveConfig()` | 476 | Persist config |
| `loadWeather()` | 560 | Fetch Open-Meteo + AQI, render weather widget |
| `searchPhoton()` | 647 | Geocoding via Photon + postcodes.io |
| `setupPhotonInput()` | 700 | Wire debounced search to input + results div |
| `openLocationPicker()` | 741 | Open modal, focus search, start viewport listener |
| `closeLocPicker()` | 757 | Close modal, blur keyboard, clean up listener |
| `renderLocPickerBody()` | 764 | Render saved/unsaved locations in picker scrollable body |
| `updatePickerButtons()` | 884 | Sync from/to button labels + toggle no-locations notice |
| `lookupRoutes()` | 908 | Main 3-phase route search |
| `findNearbyStops()` | 1110 | TfL StopPoint API call |
| `renderRouteResults()` | 1155 | Render stop cards with arrival time chips |
| `startAutoRefresh()` | 1218 | 30s interval refresh |
| `renderSettings()` | 1237 | Build settings HTML (3 sections + accordion) |
| `toggleAdvancedSettings()` | 1359 | Expand/collapse API + App sections |
| `initLocDragDrop()` | 1369 | Wire mouse + touch drag reorder |
| `addNewLocation()` | 1453 | Create loc with ⭐, show flash, navigate to editor |
| `renderLocationEditor()` | 1472 | Editor UI: emoji dropdown, name, address search |
| `calculateBearing()` | ~1760 | Compass bearing between two lat/lon points |
| `haversineMetres()` | ~1770 | Distance in metres between two lat/lon points |

---

## Version History (this context window)

| Version | Summary |
|---------|---------|
| v2.0.0 | Full overhaul — new UI, location editor, weather widget, improved search |
| v2.1.0 | London-only Photon bbox search, postcodes.io fallback, edit buttons in picker, emoji per location, condensed route picker layout |
| v2.2.0 | Dual-endpoint line matching (origin + destination stop cross-reference); configurable walking radius; `[BusChecker]` console debug logging |
| bugfix | Route sequence direction filter replaced crude compass check — fetches `/Line/{id}/Route/Sequence/{dir}` to confirm origin appears before destination in ordered stop sequence |
| v2.3.0 | Search spinner on lookup button; temporal "Saved" flash for name/emoji in editor; emoji input validates single grapheme via `Intl.Segmenter`; emoji stripped from name field |
| v2.4.0 | Emoji dropdown with 20 common options; Home/Work emoji prefilled; name saves on Enter (not just blur); new locations get ⭐; "New location created!" flash; drag-and-drop location reordering (mouse + touch); London-only onboarding notice; iOS PWA 🚌 icon |
| v2.5.0 | Onboarding copy shortened, layout tightened; emoji dropdown tap-outside fix (capture-phase once listener); settings reorganised into 3 sections with "More settings ▾" accordion |
| v2.6.0 | Modal search bar moved to header (keyboard-proof); `visualViewport` listener adjusts sheet height on keyboard open; no-locations friendly notice below route picker |

---

## File Structure

```
bus_app/
  index.html      ← entire app (HTML + CSS + JS, ~1855 lines)
  CLAUDE.md       ← rules for Claude agents working on this project
  HANDOFF.md      ← this file
  README.md       ← user-facing documentation
  LICENSE
  old versions/   ← archived older builds
```
